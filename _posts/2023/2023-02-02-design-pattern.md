---
title: "Software Design Patterns: MVC, MVP, and MVVM"
tags:
  - study
  - architecture
toc: true
toc_sticky: true
toc_label: "Table of Contents"
---


## Software Design Pattern

`Software design pattern` is a **reusable** solution (**template**) to a commonly occurring problem in software design. For instance, modern applications need diverse features which results in a growth of code size and complexity. To solve this problem, programmers can use the architectural design pattern to make their applications more cleaner/clearer, readable, and maintainable/extensible.

There are many formalized best practices that programmers can use when designing an application or system. These are normally based on the concept of **modularization** that is characterized by the following features:

- `coupling` : the degree to which two components (classes or modules) interact with one another.
- `cohesion` : the degree to which all of the methods and data in a component are related to one another.


![image](https://github.com/jonghwanchung/jonghwanchung.github.io/assets/97339878/05a55936-a812-4088-b885-37c64ec56a92){: .align-center}
<span style="font-size:80%">Coupling and Cohesion.</span> [[Source]](https://www.coursera.org/lecture/object-oriented-design/1-3-1-coupling-and-cohesion-q8wGt)


To achieve this goal, each component has a high cohesion and the inter-components have lower coupling.



Among these design patterns, the three most popular design patterns are:

 - `MVC` : Model(data & business logic), View(UI visualization), **Controller**(input handling, model update, view selection)
 - `MVP` : Model(data & business logic), View(UI visualization, input handling), **Presenter**(model update, view update)
 - `MVVM` : Model(data & business logic), View(UI visualization, command), **ViewModel**(model update, data & data-binding)


![image](https://github.com/jonghwanchung/jonghwanchung.github.io/assets/97339878/e97c0b34-91a4-4232-8852-0ba0cebaf243){: .align-center}
<span style="font-size:80%">Architecture of MVC, MVP, and MVVM</span>



## Interaction flow of MVC Pattern

The typical flow of interactions in MVC pattern is as follows:
 1. The User (web browser) interacts with the Controller.
 2. The Controller handles user input, updates the Model accordingly, and selects the View.
    - The Model manages data, processes business logic, and maintains the application’s state.
 3. The View actively reflects the changes using the Model. (The View has a reference to the Model, but the Model has no information about the View.)
    - The View can be updated by a retrieve query using the Model,
    - The View can be updated by a change notification from the Model, or
    - The View can be updated by a periodic polling to the Model.


While the MVC pattern offers numerous benefits (simplicity) in terms of code organization and maintainability, it has some disadvantages that developers should consider when choosing an architectural design pattern. For instance, the View(UI) and Model(data access) are tightly coupled, which makes it difficult to respond to changes of application features.



## Interaction flow of MVP Pattern

The typical flow of interactions in MVP pattern is as follows:
 1. The User (web browser) interacts with the View.
 2. The View handles user input/event and forwards this action(s) to the Presenter.
 3. The Presenter updates the Model and the View, respectively.
 4. The change of the View is reflected by the Presenter. (not Model-aware)



## Interaction flow of MVVM Pattern

The typical flow of interactions in MVVM pattern is as follows:
 1. The User (web browser) interacts with the View.
 2. The View has a reference to ViewModel and commands the ViewModel.
 3. The ViewModel updates the Model and manages data. (The ViewModel has no information about the View.)
 4. The change of the View is automatically reflected by synchronization (data-binding) through the ViewModel. (not Model-aware)



The MVP and MVVM patterns remove the dependency between View and Model. In other words, the View is not Model-aware. This decoupling make it easier to respond to changes of application features. However, these patterns are difficult to design and implement.



## Reference

- [github]()
