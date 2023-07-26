---
title: "Implementing a New Custom Netlink Family Protocol"
tags:
  - research
  - linux
toc: true
toc_sticky: true
toc_label: "Table of Contents"
classes: wide
---



## What is Netlink?

> "`Netlink` is a **socket family** used for **inter-process communication** (IPC) between both the **kernel** and **userspace processes**, and between different userspace processes, in a way similar to the Unix domain sockets available on certain Unix-like operating systems, including its original incarnation as a Linux kernel interface, as well as in the form of a later implementation on FreeBSD. Similarly to the Unix domain sockets, Netlink communication cannot traverse host boundaries. However, while the Unix domain sockets use the file system namespace, Netlink sockets are usually addressed by process identifiers (PIDs)."
> <br>
> [https://en.wikipedia.org/wiki/Netlink](https://en.wikipedia.org/wiki/Netlink)


From the definition, Netlink is normally used as a communication protocol between kernel and userspace. Unlike `ioctl()`, it is based on socket, which enables **notification** from the kernel to userspace. With `ioctl()`, the kernel can only send a response regarding to a user request. With netlink socket, however, user processes can be blocked via blocking functions such as `send()` and `recv()` to send/receive any messages to/from the kernel.

```
#include <asm/types.h>
#include <sys/socket.h>
#include <linux/netlink.h>

netlink_socket = socket (AF_NETLINK, socket_type, netlink_family);
```

There are some pre-defined famous netlink protocol family: for instance, `NETLINK_ROUTE` for routing and link updates, `NETLINK_KOBJECT_UEVENT` for device events, and so on. These pre-defined family protocols merged into Linux mainline can be found in [netlink.h](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netlink.h#L9).

Without the use of generic netlink protocol, the maximum number of unique protocol families is 32: `#define MAX_LINKS 32`. This is one of the main reasons that the generic netlink family was created.

In this post, we use the existing netlink protocol (not generic netlink protocol). As there are 23 protocol families exist in the latest kernel, we can implement up to **9 custom protocol families** (23~31).



## Basic Implementation with Custom Netlink

### Kernel Module

THe following is a **basic kernel module** that creates a custom netlink protocol family. The value of which is **30**, using the kernel function `netlink_kernel_create()`. Note that the signature of the function was changed since kernel version 2.6. [[link]](https://elixir.bootlin.com/linux/latest/source/include/linux/netlink.h#L58)

```c
static inline struct sock *
netlink_kernel_create(struct net *net, int unit, struct netlink_kernel_cfg *cfg)
{
  return __netlink_kernel_create(net, unit, THIS_MODULE, cfg);
}
```

```c
#include <linux/kernel.h>
#include <linux/netlink.h>
#include <linux/module.h>
#include <net/netlink.h>
#include <net/net_namespace.h>

#define NETLINK_CUSTOMFAMILY 30

struct sock *socket;

static void custom_nl_receive_message(struct sk_buff *skb) {
  printk(KERN_INFO "Entering: %s\n", __FUNCTION__);

  struct nlmsghdr *nlh = (struct nlmsghdr *) skb->data;
  printk(KERN_INFO "Received message: %s\n", (char*) nlmsg_data(nlh));
}

static int __init custom_init(void) {
  struct netlink_kernel_cfg config = {
    .input = custom_nl_receive_message,
  };

  socket = netlink_kernel_create(&init_net, NETLINK_CUSTOMFAMILY, &config);
  if (socket == NULL) {
    return -1;
  }

  return 0;
}

static void __exit custom_exit(void) {
  if (socket) {
    netlink_kernel_release(socket);
  }
}

module_init(custom_init);
module_exit(custom_exit);
```

This creates a kernel module that listens netlink protocol family with the number 30. When any process creates a netlink socket and send a message, `custom_nl_receive_message()` will be called. Note that netlink is aware of **network namespace**, so the first argument for the function `netlink_kernel_create()` is the pointer of a network namespace (type: `struct net *`). Linux kernel, by default, has at least one network namespace: `init_net`. You can use the variable for creation or your own network namespace variable. Also, since netlink is aware of network namespace, netlink multicast (default) can only be received from within the specified network namespace. For example, if two network namespaces are available (netns and newns), and the above kernel module sends a multicast message to the initial network namespace, every processes in the other network namespace newns will not receive the message.



### Userspace Program

The below code is a **userspace program** that can send and receive messages from the kernel throught `NETLINK_CUSTOMFAMILY` family protocol (30).

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <linux/netlink.h>

#define MAX_PAYLOAD 1024
#define NETLINK_CUSTOMEFAMILY 30

int main(int argc, char *argv[]) {
  int fd = socket(AF_NETLINK, SOCK_RAW, NETLINK_CUSTOMEFAMILY);
  if (fd < 0){
    ... error handling ...
  }

  struct sockaddr_nl addr;
  memset(&addr, 0, sizeof(addr));
  addr.nl_family = AF_NETLINK;
  addr.nl_pid = 0;  // For Linux kernel
  addr.nl_groups = 0;

  struct nlmsghdr *nlh = (struct nlmsghdr *) malloc(NLMSG_SPACE(MAX_PAYLOAD));
  memset(nlh, 0, NLMSG_SPACE(MAX_PAYLOAD));
  nlh->nlmsg_len = NLMSG_SPACE(MAX_PAYLOAD);
  nlh->nlmsg_pid = getpid();
  nlh->nlmsg_flags = 0;
  strcpy((char *) NLMSG_DATA(nlh), "HELLO KERNEL FROM USER");

  struct iovec iov;
  memset(&iov, 0, sizeof(iov));
  iov.iov_base = (void *) nlh;
  iov.iov_len = nlh->nlmsg_len;

  struct msghdr msg;
  memset(&msg, 0, sizeof(msg));
  msg.msg_name = (void *) &addr;
  msg.msg_namelen = sizeof(addr);
  msg.msg_iov = &iov;
  msg.msg_iovlen = 1;

  printf("Sending message to kernel...\n");
  sendmsg(fd, &msg, 0);

  return 0;
}
```


When the program is executed, the following message should be received from the kernel via `dmesg`:

```
Entering: custom_nl_receive_message
Received message: HELLO KERNEL FROM USER
```



### Unicast from Kernel Module

Let's make the **kernel module** send a response for a request.

```c
static void custom_nl_receive_message(struct sk_buff *skb) {
  struct nlmsghdr *nlh = (struct nlmsghdr *) skb->data;
  pid_t pid = nlh->nlmsg_pid; // pid of the sending process

  char *message = "HELLO USER UNICAST FROM KERNEL";
  size_t message_size = strlen(message) + 1;
  struct sk_buff *skb_out = nlmsg_new(message_size, GFP_KERNEL);
  if (!skb_out) {
    printk(KERN_ERR "Failed to allocate a new skb..\n");
    return;
  }

  nlh = nlmsg_put(skb_out, 0, 0, NLMSG_DONE, message_size, 0);
  NETLINK_CB(skb_out).dst_group = 0;
  strncpy(nlmsg_data(nlh), message, message_size);

  int result = nlmsg_unicast(socket, skb_out, pid);
}
```

The userspace process will receive a netlink message with `NLMSG_DONE` type. It seems that the newly created `struct sk_buffer` variable is deleted when it is sent through `nlmsg_unicast()`, hence calling `nlmsg_unicast()` contains the memory management [[link]](https://stackoverflow.com/questions/10138848/kernel-crash-when-trying-to-free-the-skb-with-nlmsg-freeskb-out/10138935#10138935). Therefore, calling `nlmsg_free()` is forbidden, otherwise a **kernel panic** would be occured.



### Multicast from Kernel Module

The kernel module can also send a **multicast netlink message**, a broadcast for a specific group in netlink family.

```c
#define NETLINK_MYGROUP 2

static void custom_nl_send_user (void) {
  char *message = "HELLO USER GROUP MULTICAST FROM KERNEL";
  size_t message_size = strlen(message) + 1;

  struct sk_buffer *skb = nlmsg_new(NLMSG_ALIGN(message_size), GFP_KERNEL);
  if (!ksb) {
    printk(KERN_ERR "Failed to allocate a new skb..\n");
    return;
  }

  struct nlmsghdr *nlh = nlmsg_put(skb, 0, 1, NLMSG_DONE, message_size, 0);
  strncpy(nlmsg_data(nlh), message, message_size);

  int result = nlmsg_multicast(socket, skb, 0, NETLINK_MYGROUP, GFP_KERNEL);
}
```


## Reference

- [Latest Linux Source Code](https://elixir.bootlin.com/linux/latest/source)
- [Kernel Core API Documentation](https://www.kernel.org/doc/html/next/core-api/)