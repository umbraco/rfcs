# Future-proofing the Umbraco backoffice

Request for Contribution (RFC) 0025 : Future-proofing the Umbraco backoffice

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: technical users, developers and package authors

## Summary

To future-proof Umbraco’s backoffice, we intend to run a three-part process - each with their own RFC.
1. Standalone UI library (RFC soon - January 2021)
2. Defining backoffice API (RFC spring 2021)
3. Implement backoffice (RFC summer 2021)
## Motivation

Umbraco’s backoffice is a big reason editors AND developers choose Umbraco. It is easy to use and flexible to customize to specific needs for a project/client

The current (second) generation back-office is built using AngularJS. It was a great choice in 2013, and has served us well, but has long been considered outdated. From December 2021 it will no longer receive updates/patches (RIP).

Since 2012 we have learnt a lot about building a complex Single Page Application (SPA). The current situation doesn’t allow us (or makes it really hard) to fix things we have done inherently wrong. The current state is holding development - and excitement - back. 

There are 4 challenges of the current backoffice, that motivates us for this project:
1. Product-specific UI makes the Umbraco journey (CMS, 1st- and 3rd party packages, Cloud, etc.) inconsistent and development inefficient.
2. Unclear API surface makes developers reach for undocumented hacks, resulting in an unofficial API layer that we still try to maintain.
3. The current architecture and framework is holding back and/or slowing down development.
4. AngularJS enters EOL in ~12 months.

With the next generation of Umbraco’s backoffice, we want to bring a number of benefits to Umbraco, namely:
- Speed up maintenance and future development of BackOffice for Umbraco HQ and community
- Make integrations to 3rd parties more straightforward
- Make Umbraco more attractive for newcomers, by using modern technology and methodology - thereby also making it easier to contribute 
- Simplify and speed up work for developers and package-authors, by enhancing the ability to create great UI/UX
- Streamline the experience throughout the entire journey incl. 1st- and 3rd party packages, Cloud etc.
- Reuse work across our products for less development but also for improvements to be distributed.
- Maintain market leadership in customizable editing experience (tailored to clients)
- Improve the upgrade experience

## Detailed Design

To tackle the issues and achieve the benefits described above, we intend to split the project into 3 parts/major milestones.

We do this in order to get real user feedback through an exploratory and agile process, and to make a difference for developers and editors in increments.

The three milestones will have individual RFCs and are as follows:
1. Build a UI Library
2. Define backoffice API
3. Implement backoffice

The parts are not entirely sequential and can run in parallel to a certain extent.

### Timing
RFC for part 1 will be published in January and we expect work to start shortly thereafter. We feel this is important and that we risk halting further development if we don't act now and with sufficient team-size.
We expect part 2 to start in the spring. Part 1 does not need to be done before this can start
Part 3 will start when part 2 is ready.

Read on to learn more about the different parts, and, as mentioned, there will be separate RFCs for each part (we will make sure to update this with links as they are published). We also intend to create a new community team around these efforts. 
### 1. Build a UI library
We want to create a library of new UI components that are thoughtfully built with reuse and accessibility in mind.
The UI library will be its own separate Open Source project with source code on GitHub and distributed via NPM.

The goal of part 1 is to create a consistent set of components covering our needs for the CMS. Additionally these should make it easy for developers and package-authors to make something that "feels like Umbraco".

We want documentation to be built in, so it won't become outdated or left out.

We want the components to work with any framework so it is not tied to Umbraco’s backoffice only.

We intend to start releasing public builds when we have sufficient components to get an understanding of how the UI Library will work and in order to get feedback and collaboration as soon as possible.

**A UI Library will make the Umbraco experience more beautiful, accessible and consistent, while also making it easier and less work for developers and maintainers.**
### 2. Define BackOffice API
In the second part, the goal is to define a public API for the backoffice. The official/documented API for the current backoffice implementation is too limited, which has led developers to rely on undocumented hacks, that the backoffice now unofficially supports.

The current API is not easy for newcomers to understand, and it makes it hard to know what can be safely used/changed.

Furthermore, the current API is tightly coupled to AngularJS, which means developers have to learn an additional (outdated) framework to extend/customize Umbraco and also means that changes to the framework can break the API.

The goal of part 2 is to define an API that is based on Web Standards and best practices, without any ties to a specific framework.

This second part will define what extending Umbraco will look like. We want to keep our options open and not decide anything just yet. This will be discussed and decided in the RFC for part 2 and there will be examples to start the discussion. Further prototyping will also be done in part 2.
_An example of how this could work_, as opposed to today where you define an angular-template and a .js-resource for a dashboard or property editor, you would instead define a tagName and a .js-resource. This will give developers the flexibility to decide on their own implementation of choice. 

We want to support all the current documented/official use cases, and a subset of the undocumented use cases. This is crucial to help lighten the burden on package developers in terms of migrating to the new API.

It should be easy and intuitive to extend and customize Umbraco’s backoffice - familiar for experienced Umbraco developers and recognizable for frontend developers.

We want the API-implementation to have TypeScript definitions, enabling IDEs and tooling to help developers as much as possible, but TypeScript will not be forced on developers. 

**A clearly defined API will make it easier to develop and  extend Umbraco’s backoffice, provide better tooling and make it safer to upgrade Umbraco.**
### 3. Rebuild BackOffice 

We need to rebuild the existing Umbraco backoffice, with the new API defined in part 2. This will be a complete rewrite, not a migration, and can only be released after it is completely done!

The goal of part 3 is to have a modern and maintainable platform, where the technology doesn't complicate or slow down further development. Umbraco’s backoffice should be flexible enough to allow for new ideas and should be testable. The rebuild will use TypeScript (mandatory for Core developers). 

When rebuilding backoffice a choice of a framework can become relevant. No matter the choice of framework, this won’t make any difference for third party extensions/packages as backoffice APIs have no framework coupling. 

In order to mitigate the impact of releasing part 3 for developers and package authors, we will ensure that documentation and training are updated in due time, and that a Release Candidate will be available for an extended period of time before the final release.
This will be done in collaboration with the various community teams, such as the documentation curators and the package team. 

**A new implementation of Umbraco’s backoffice will give us a better platform to build on and allow us to remove AngularJS. Furthermore, allowing us to evolve the backoffice without breaking the API.**
## Drawbacks

Part 3 will be a big-bang release (everything released at once)  and will effectively break all existing packages. 

We have considered alternative approaches  such as a more incremental approach using adapter/API-bridge/compatibilty layer. We chose to move on with the proposed solution, to allow the flexibility we want in a new architecture, to "cut the strings" and not have legacy-code to support.

It will require a major version release of Umbraco CMS.
## Alternatives

**We could just pick a framework and build everything in that**
While this might initially make it easier for everyone, it would likely put us in the same place in the future. Also, this would mean we have to do all the work up front, with no clear benefit for other products.

**We could do a more incremental upgrade and have adapters/bridges providing some backwards compatibility**
This would result in a larger project scope and we would eventually either be stuck maintaining the bridge/adapter or having to break everything later. This won’t allow us to get the cleanup or architectural freedom we want.

**We could stick with AngularJS - it works and won’t stop working. Or just wait some more...**
It’s not really a solution. We get further and further away from the “hype-train” and new developers will have to learn something “only” for Umbraco. It is already an argument against Umbraco. This compares to staying on the ASP .NET framework. The longer we wait, the longer we have to maintain potential issues in angularJS ourselves (after December, 2021).

## Out of Scope

This RFC describes the project structure/process and that’s what we would like to discuss. There will be detailed RFCs for each part of the process.

Any discussion about specific technologies is out of scope for this RFC. That will be appropriate in the individual RFCs, as they are specific to each part. Use of TypeScript can be debated here, as it spans all three parts!

No new features will be added when rebuilding backoffice, as we aim to keep the same feature set as latest v8. This can be discussed but not any specific new features.

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

Validating the overall approach for future-proofing Umbraco’s backoffice

Any obstacles/issues that need to be taken into account either for the overall process or for RFCs for the individual parts.

How to approach migration of packages. What can be done to ease the process while maintaining the ability to define a new API and rebuild the backoffice. 

## Related RFCs 

- Previous RFC about “Future of the Back-Office Front-End”
*Many ideas from this RFC surfaced in this previous RFC, especially the description of motivation and benefits/opportunities have been influential. 
- The main differences are around the proposed process and approach.*
## Contributors

This RFC was compiled by:

* Filip Bech-Larsen, Program Manager for Umbraco CMS (Umbraco HQ)
* Niels Lyngsø, UI/UX Engineer (Umbraco HQ)
* Rune Strand, Product Communications Producer (Umbraco HQ)



