# RFC Name

Request for Contribution (RFC) 0000 : _Future of the Back-Office Front-End_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is technical users and developers.

## Summary

With this RFC we are highlighting priorities for making new decisions for the future of the Umbraco CMS front-end project. This document outlines several areas that we believe should be part of the discussions which can then lead to decisions on technology, implementation and roll-out strategies.

We note that part of these discussions include a recommendation to also change the framework currently used (AngularJS), as it is being discontinued. However, the technology changes is intentionally not the reason for change itself, instead we focus on the value and opportunities of making changes in the first place.

## Motivation

Our motivation for changing the project and selecting a new approach for the front-end of Umbraco CMS going forward is listed below. These are examples of value that can be added/improved in the platform, without the choice of technology or solution it-self being the motivation.

- We want to __make it easy to contribute__ to the project, which today might be more difficult due to inconsistency and limitations. By cleaning up and lowering the learning curve we believe we can improve the contribution experience.
- We want to __make the project interesting to work on__ for new and existing contributors. We believe that new technology choices and changes can improve this.
- The backoffice user experience is a key part of the value of Umbraco. We want to __enhance the ability to create great UX__ in Umbraco, where the technology is assisting improvements rather than creating barriers.
- We want to __make the project easier to maintain__ to limit degeneration over time, even with multiple contributors with limited connection.
- Transparency and extensibility are important parts of the open source project, so we want to __make it easy to extend on the project__ by following best practice examples, templates and guidelines - which are also followed by the project in general to lead by example.

## Opportunities

We use the following list of questions to highlight the priorities that we think are most important. For each priority there is some reflection on why the current state of the project is unsatisfactory, an approach to how this could be improved and (where possible) an action we could take to address this.

### Quick navigation

[How might we make the code more developer-friendly?](#1-how-might-we-make-the-code-more-developer-friendly)

[How might we make the code easier to understand?](#2-how-might-we-make-the-code-easier-to-understand)

[How might we make developers feel more confident committing code?](#3-how-might-we-make-developers-feel-more-confident-committing-code)

[How might we make it easier for people to get started?](#4-how-might-we-make-it-easier-for-people-to-get-started)

[How might we make it easier to contribute?](#5-how-might-we-make-it-easier-to-contribute)

[How do we enable user experience improvements?](#6-how-do-we-enable-user-experience-improvements)


### 1. How might we make the code more developer-friendly?

__1.1 Current barrier:__ the current codebase is complex, meaning that the developer has to be aware about many features and concerns when developing on the system. The unclear responsibilities makes it hard to grasp what effects a change in the code would cause. Many components rely on a complex implementation where part of the functionality is in the implementation rather than the component itself.

__Proposed approach:__ We want to improve the architecture, by creating a separation of concerns that would improve the understanding of a feature and making it easier to grasp its purpose.

__Proposed action:__ Rewrite or refactor existing code and components to only deal with a certain set of responsibilities that are clearly defined.

### 2. How might we make the code easier to understand?

__2.1 Current barrier:__ One of the barriers to being able to easily read and interpret the code is that there are varying and inconsistent implementations of patterns.

__Proposed approach:__ Making consistent implementations would reduce the amount of time for code comprehension - for both seasoned and new developers.

__Proposed actions:__ 
- Introduce a library of available components that could be used to demonstrate building blocks that developers can use.
- Introduce strong typing to the JavaScript to surface available types/properties through IDE/intellisense.

---

__2.2 Current barrier:__ Currently, the legacy back-office code is built on outdated software patterns which means that a developer swapping from more current JavaScript frameworks to working in the back-office is forced to make a fairly substantial context switch. This requires valuable effort and increases the overhead for developers to jump in and out of back-office code. 

__Proposed approach:__ Making the code more in-keeping with current front-end paradigms would tap into patterns that developers who do other front-end work will be familiar with. E.g reusable component based. This would reduce the mental load associated with working on the back-office by minimising the number of patterns a developer needs to follow.

__Proposed action:__ Update existing legacy framework so that patterns are consistent with other widely-used front-end frameworks

### 3. How might we make developers feel more confident committing code?

__3.1 Current barrier:__ The backoffice holds many complex dependencies that means that touching one area can impact other, seemingly unconnected, areas.

__Proposed approach:__ Making use of a test framework that would allow for developers to easily run integration and unit tests when contributions are made would both reassure that other areas remain functional and increase the overall robustness of the code. It is worth noting that there is a test framework included in the current framework, but test coverage is patchy and not maintained.

__Proposed actions:__
- Introduce a properly maintained test framework (e.g. implementing a minimum code coverage of tests)
- Encourage contributors to write tests by providing best-practise structure and implementation templates
  
### 4. How might we make it easier for people to get started?

__4.1 Current barrier:__ At the moment, there is scattered documentation on the official Umbraco site as well as in various blog posts.

__Proposed approach:__ Pooling these resources into a single source of truth would be beneficial for developers to have a more informed starting point instead of piecing code together from multiple sources.

__Proposed actions:__ 
- Create a dedicated Front-End space on “our” to collect code examples and tutorials in a single location
- Provide a repository of best practise templates for custom property editors, sections, packages, tests etc

---

__4.2 Current barrier:__ If you want to contribute to the current back-office then it is necessary to have the entire core project running on your environment. Getting the project set-up is well documented but can experience issues and require troubleshooting.

__Proposed approach:__ By allowing the front-end of the backoffice to run independently from the core, we would remove this as a starting barrier and would reduce the ramp-up time for new and existing developers to being able to contribute.

__Proposed action:__ Decouple front-end code from server-side build to create a clear definition between the two
  
### 5. How might we make it easier to contribute?

__5.1 Current barrier:__ Current project contains a lot of reusable features as components and services etc. but these are hard to surface and not always well-utilised.

__Proposed approach:__ It would be good to provide an overview of features and a guide on how to use them.

__Proposed actions:__ 
- Displaying components in a UI-Component-Library will clarify the available components, document their usage and implementation, plus ensuring a clear responsibility of components as they have to be able to live in the component library.
- Limiting the public available interface/API of features, we will provide a certain set of functionality and ensure a more consistent usage of features.

### 6. How do we enable user experience improvements?

__6.1 Current barrier:__ It is currently hard to make quick and responsive updates to the UI based on accessibility issues due to the complex nature of global style re-use

__Proposed approach:__ We believe that by improving the architecture using approaches outlined in [How we might make the code more developer-friendly](#1-how-might-we-make-the-code-more-developer-friendly) and [How might we make it easier for people to get started](#4-how-might-we-make-it-easier-for-people-to-get-started), this will allow for quicker and easier re-modelling of the UI

__6.2 Current barrier:__ At the moment it is possible for editors to overwrite each other’s changes and lose their own changes when editing content in the back-office.

__Proposed approach:__ It would be beneficial to support asynchronous editing as this would give clients the flexibility and confidence to include multiple editors on one task.

__Proposed action:__ Research current best practices for asynchronous editing on existing large software and produce an RFC that outlines the best possible approach to resolving this issue

## Proposed Approach

Based on the key priorities and highlighted actions, our proposed approach contains the following key features:
- Introduce a new JavaScript framework to replace the current AngularJs code
- Create a UI component library in the new framework that would provide discrete building blocks that can be used to compose the updated backoffice
- Start introducing UI views on a section-by-section basis
- When a section is introduced in the new framework allow back-office users to switch between the new and legacy views
- New features would be added in the new framework allowing users to access them through the new views and encouraging them to progress to using the new framework

## Drawbacks

- Given the complexity and breadth of the back-office, implementing the functionality in a new framework will not be a trivial amount of work.
- Future package development would need to be re-worked quite substantially and package developers would ideally be given a high degree of support for porting their packages to the new framework.
- There will be a period of two platforms that will both need to be maintained in tandem with potentially quite different technology stacks.
- Extensive UAT testing would need to take place on the newer framework whilst the legacy framework is still active 

## Alternatives

- Keep the existing framework as-is
  - This would be deferring the current issues with maintainability until they become urgent and untenable 
- Keep the existing framework and refactor current code to improve consistency and structure where possible
  - This is potentially a large amount of work to reach a place where the framework will still be outdated in a few years time and this situation will arise again
- Try to update the existing framework to a newer version and migrate/rewrite the code to meet the updated patterns
  - This would be akin to re-writing the current codebase given that the next version of Angular has remarkably different patterns and paradigms to the existing AngularJs version
- Start creating new components and features in a chosen new framework, and exist in a “hybrid” structure where a new framework and the legacy framework work side-by-side
  - This would introduce a staged approach but would also add more complexity to the current code (and technology stack).
  - This would also mean that when the changeover was “complete” there would be a like-for-like structure meaning that the current issues with complexity would remain, but in a different framework language.
- Complete rewrite of the current back-office without a staged handover
  - The project itself is too large and complex to complete in a short amount of time. During the rewrite, the current codebase would need to be frozen with hotfixes being completed on both the legacy and new code.


## Out of Scope

List any items out of scope for this RFC. Use this to try to steer the discussion to be as specific as possible to the RFC details.

## Unresolved Issues

These are the areas that we are hoping to receive input from the community on:

Reasons for change
- Are there other key priorities that should be taken into account when considering the future of the back-office?
- Have we covered the main key areas that will improve the front-end going forwards?
- Are there any other questions or concerns that should be covered?

Action points
- Do the highlighted action points contribute to this improvement process?
- Are there other action points that you believe is more important to focus on at this stage?
- Are there action points that need to be clarified or expanded on?

Proposed approach
- Does the proposed approach make sense within the context of the project scope?
- Are there other approaches that should be explored before a final decision is made?


## Related RFCs (where are we in roadmap?)

This is the beginning and will hopefully be used as a base for future RFCs and progress

## Contributors

This RFC was compiled by:

* Laura Weatherhead (Umbraco community)
* Niels Lyngsø (Umbraco HQ)
* Jacob Midtgaard-Olesen (Umbraco HQ)
* Rasmus John Pederson (Umbraco HQ)
* Umbraco 2019 Retreat members
