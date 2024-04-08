---
title: How To: Kotlin Architecture - Which One to Use?
layout: note
date: 23-08-11
tags:
  - android
  - architecture
  - kotlin
  - mvi
  - mvvm
  - mvc
---

Choosing between MVC (Model-View-Controller), MVI (Model-View-Intent), and MVVM (Model-View-ViewModel) in Android application design involves considering the complexity of our project, the scalability requirements, the team's familiarity with the pattern, and the specific needs of our application. These architectural patterns are not strictly interchangeable; each offers  advantages and is better suited to certain types of projects.

### MVC (Model-View-Controller)

- **When to Use**: MVC is great for simple applications with a small team, where we want clear separation between the database/logic (Model), UI (View), and the intermediary between the two (Controller). It's a well-understood pattern, making it easy to onboard new developers.
- **Preferred Projects**: Smaller applications or projects where we want a straightforward, no-frills architecture that gets the job done without too much complexity.

### MVVM (Model-View-ViewModel)

- **When to Use**: MVVM shines in applications that require rich data binding and have a complex UI that needs to be kept in sync with underlying data models. The pattern is highly scalable and facilitates a more decoupled and testable codebase. It's particularly beneficial when using frameworks that support data binding, as it reduces boilerplate code for updating the view.
- **Preferred Projects**: Larger, more complex applications especially those that benefit from strong separation of concerns and where UI state is heavily dependent on underlying data models. It's also well-suited for projects that anticipate frequent UI updates based on data changes.

### MVI (Model-View-Intent)

- **When to Use**: MVI is newer compared to MVC and MVVM and follows a more cyclical pattern where the intent (user actions) triggers updates to the model, which then reflects back onto the view. It's great for reactive applications and where a unidirectional data flow is preferred, making the application easier to reason about and debug.
- **Preferred Projects**: Applications that benefit from reactive programming principles, particularly those where we want to manage states in a very controlled and predictable manner. It's also suitable for apps with complex interaction patterns that need a more structured approach to state management.

### Decision Factors:

- **Complexity and Size**: MVVM and MVI are generally better suited for more complex and larger applications due to their scalability and support for complex data operations. MVC might be more appropriate for simpler projects.
- **Team Expertise**: Familiarity with the pattern among team members can be a critical deciding factor. Adopting a pattern that our team is already comfortable with can speed up development.
- **UI Complexity and State Management**: If our project involves complex UI interactions and state management, MVVM or MVI might be more suitable. MVVM offers strong support for data binding, while MVI offers a predictable state management and unidirectional data flow.
- **Testing and Maintenance**: Consider how easy it is to test and maintain applications built with each pattern. MVVM and MVI typically offer better separation of concerns, making it easier to write unit tests and maintain the codebase.

Ultimately, the choice between MVC, MVVM, and MVI depends on our specific project requirements, the complexity of the UI and data model, and the preferences and expertise of our development team. Each pattern has its strengths and is designed to solve different kinds of problems, so consider the needs of our project carefully before making a decision.