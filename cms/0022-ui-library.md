# UI Component Library
Request for Contribution (RFC) 0022 : UI Component Library

## Code of conduct
Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience
The intended audience for this RFC is: technical users, developers and package authors

## Summary
We want to ensure a coherent experience for both Content Editors and Developers across products and extensions of these. By separating the essential user interface components from any specific project we ensure usage and maintenance of these will stay as straightforward as possible, making the life of maintainers, contributors, and package developers easier.

The UI Library is a central source of visual building blocks for Umbraco applications and extensions of such.

Sharing improvements and ensuring the functionality for a component fits in the bigger perspective and not just solving an immediate need.

The UI Library will serve as a component guide and come with a browser application (Storybook) in which developers can view, try out and learn about the available components.

The UI Component library will be a separate Open Source Project with source code on Github and distributed via NPM.


## Motivation
We want to ease the development process of the Umbraco backoffice and packages for it. A UI component library will lower the learning curve and make it easy for developers to get an overview of components and their purposes.

This overview will also help to design a consistent product experience. By easing the process of identifying components, their role, and their relationships.

The UI Library is a standalone project, which can be used in multiple projects, including third-party.
The components of the UI Library will be based on web standards and will work in any project.

The library provides a workspace for developing UI components, enabling us to work on UI without being bound to a specific technology. Initially, we will use this to create components for the future backoffice without relying on any framework decisions. 

We want to ensure that our core UI Components serve a general-purpose. Having them developed separately from any specific project will help ensure the scope of the components keep the right level of abstraction for reusability across projects, extensions, and packages.

The workspace will enable us to ensure a set of requirements of each component, for example making accessibility a first-class citizen.

## Out of Scope
- **Technology/framework** for Umbraco’s backoffice
- **How components are used in the backoffice**: The usage examples in this RFC are examples of how this could be done but not settled.
- **Umbraco Design System** It's our intention to provide the parts necessary for package developers to follow the Umbraco style, but this will not be part of this project.
- **No specific purpose** We want to ensure any product can use this Library, therefore we do not want this to be specific towards the backoffice or any other usages.


## Detailed Design

For the choice of technology for our UI components, we want to stay as close to native browser technologies as possible. Therefore Web Components is our choice. The components should work in any other tech stack to achieve this it must not have any external technology requirements.

The components should only contain presentational logic, not any application logic, this architectural choice ensures that the components will never mess up the logic of a project. The only purpose they serve is as visual building blocks. This ensures a clear boundary between the responsibilities of the UI Library and the project using this. Making project code much simpler as the interface code is sealed within the UI Library components.

We do not want to reinvent the wheel and therefore we have settled for using Google’s LitElement to handle the common needs for building Web Components. It is extremely lightweight and broadly used in design-systems at companies like IBM, Adobe, and others.

To ensure high-quality code we use TypeScript. This will help ensure consistent and solid code for developers of components. It's optional to use TypeScript as an implementer (i.e. when using components in a project). For those that do use TypeScript, there will be definitions to enable intellisense in IDEs,  making it easier to discover and implement the components correctly.

The project will be published on NPM, to enable ease of use and developers without .Net knowledge. The library will ship with the backoffice, meaning that backoffice packages will not require the use of NPM.

For the presentation of the components, we will use [Storybook](https://storybook.js.org/) as it’s used widely and supports Web Components. Storybook enables use to document and demonstrate each component, with the ability to interact with the component and explore the configuration options.

Storybook also enables us to write a story for a component before it has been developed, in this way it becomes a communication tool that enables us to share the ideas and scopes of components before development has started.
We intend to have the parts specific for Storybook as automated as possible to keep maintenance of Storybook as little as possible. Configuration options for components should propagate from Type definitions, code comments etc.


### Lit-element
We want to use Lit-Element to provide helpers for managing the basics of Web Components.

[![Screenshot 2021-03-05 115426](https://user-images.githubusercontent.com/3663619/110106176-a3717980-7da9-11eb-8ead-f0ef36ac33b0.jpg)](https://www.youtube.com/watch?v=ADgo_JVK02A)
*To suit you with the base knowledge of what Web Components and Lit-Element does, we had Filip make an [introduction video](https://www.youtube.com/watch?v=ADgo_JVK02A). We recommend you to watch this if you are interested in the code behind the components.*

We have highlighted the two most impactful parts of Lit-Element here:

#### HTML and CSS Template Literals
Lit-Element provides HTML and CSS Template Literal Tags which smoothens the experience when writing web component templates. This enables us to set component properties, add event listeners, and much more.

```
html`
<my-component attribute=”attribute value always as a string” .property=${myVaraibleToPassToProperty} @click=${clickEventHandler}></my-component>
`
```

Notice how properties are bound by inserting a dot in front of the property name. Similarly, we can listen for events by using the @ symbol in front of any event name.

#### Property-Decorators
Property-Decorators simplifies the initialization of web-component properties.
Most commonly used is @property(), which will make a property reactive, meaning that the element will rerender when the value is changed.

@property() also enables us to turn a property into an element attribute, by turning on reflection the value of the attribute will be synchronized. Which is a great way to make styling based on the state of a property.

[Read more about Lit-Element property decorators here](https://lit-element.polymer-project.org/guide/properties#declare-with-decorators)

### Styling

As the web components will be using Shadow DOM, the styling of these can stay fairly simple as they are fully isolated from each other. We don't intend to use any preprocessor framework for this at the current state.
The UI Library will host a set of CSS custom properties, which will be used for styling its components. These properties will also be available for any project implementing the UI Library.

Additionally, we will use custom properties for the style-values of a component that we want to enable developers to change. Custom Properties inherits through the DOM and ShadowDOMs which enables a property to be overridden for a certain part of the application without affecting other parts.

This means it would be possible to overwrite the background-color of `<uui-button>` by defining the custom property “--uui-button-background”, like this “--uui-button-background: red;”

Common styling cases should be available through attributes of components. To exemplify this, let's have a look at the UUI-Button component:

`<uui-button>Click me</uui-button>`

This component will in many cases need to be styled to stand out visually or communicate its effect.
For a delete button we would apply the danger-style through the `look` attribute:

`<uui-button look=”danger”>Delete</uui-button>`


#### Theming

We will enable a limited set of color and sizing customization options, based on CSS Custom Properties. This is mainly to enable a high-contrast mode for accessibility purposes.


#### Parts

CSS-parts is also one of the features of Web Components that can be used to overwrite certain styling of a Web Component. Currently, we do not have any case where this provides any reasonable value. But we will consider enabling this when it makes sense.

### A few notes on architecture

#### Tag-Names
All elements of this library should be prefixed with “UUI-”. This prefix will ensure that Elements from the UI Component Library will differ from specific project Elements and be different from the umb-directives from current backoffice. See the following example of an element from the CMS and two from the library:

```
<umbraco-cms-content-tree>
	<uui-tree>
		<uui-tree-item></uui-tree-item>
	</uui-tree>
</umbraco-cms-content-tree>
```

To enable component versioning at a later stage: No code should depend on TagNames of library elements, neither their own nor any of its children. This means when querying or writing CSS selectors we must use `:host`,  `this` or use classes/IDs. The reason for this is to enable component versioning at one point.

#### Use attributes for styling

When styles are derived from the state of a property, they should be turned into an attribute with reflection.
In this way, we can use CSS Selectors for the attribute rather than mapping values to CSS classes. Example:

Component CSS:

```
:host([disabled]) {
  /* styling specific to the disabled state */
}
```

Component usage via attribute:
```<uui-button disabled></uui-button>```

Component usage via property:
```<uui-button ?disabled=${...JS-expression}></uui-button>```


#### Slots

Web Components allows the use of Slots, which enables implementers to inject markup into one or more selected slots. This enables us to make compositions of components, simplifying each component as they can stay true to their own responsibility.

Enabling us to make very simple base components and extend those for specific needs.

This can be exemplified by the dialog component. It will just serve as a visual frame for any dialog-content. The dialog will provide the base for specific usages of the dialog. This could be the confirm-dialog component which will accept a few properties: headline, description, confirm-label. When a different case is needed this can be built upon the dialog-component, helping us avoid bloated components.

#### Events

To make Events propagate outside the scope of a Web Component we have created a base Event class, called UUIEvent. For any custom events, we recommend extending this class to make a proper typed Event for the given Component.

Events should only hold data if there is specific data for the event. If developers need data of the state of the component they should retrieve this from the element, eventually through `event.target`.

### Workflow

#### Contribution process

When contributing to the library it should be submitted as a PR to the library. 
PR specific for adding a new Element  will first be accepted(merged) when it lives up to these criteria:

- Element name must be prefixed with “UUI-”
- Elements must have tests and pass those
- Elements must pass the basic accessibility test
- Elements must be represented through a story in Storybook
- Elements must follow the Umbraco look and feel
- Source-code must follow the ES-lint rules

#### Publishing and versioning
We intend for the UI Library to have independent semantic versioning and rapid releases through CI/CD. Releases will automatically be available on NPM.

### Documentation
We intend to make documentation auto-generated from component source code. Via TypeScript and JS-docs.

### Usage
Any project can implement and use components of the UI Library. A project can combine this with their own components. 

###Demo

To get some context to this concept we have already implemented a prototype, which you can try out today.

[Find the repository here](https://github.com/umbraco/Umbraco.UI.RFCDemo)

Read `readme.md` for a guide on how to get the project up and running.

If you like to view the solution without building it by yourself, [we hosted the demo here.](https://umbraco.github.io/Umbraco.UI.RFCDemoStatic)

[![Screenshot 2021-03-05 114326](https://user-images.githubusercontent.com/3663619/110105188-5f31a980-7da8-11eb-9063-c358d4a381ac.jpg)](https://www.youtube.com/watch?v=yb41HCdvjFE)
*If you like a brief introduction, please watch this video where Niels Lyngsø shows the solution and presents some of the thoughts behind the code. [Available on YouTube here](https://www.youtube.com/watch?v=yb41HCdvjFE)*

### Contributions

After the RFC is accepted we would love code contributions to the repository. For now, we would love to focus on the RFC.


## Drawbacks
There is an added complexity as a result of separating the UI Components from the CMS Project.

## Alternatives
***We could build our UI Library in VueJS or any other framework.***

Developing the UI components without a framework enables us to incorporate them in any other project, and it enables projects to evolve without changing the UI Library, as the UI Library does not rely on any specifics regarding project implementations.

***VueJS***

We know VueJS is popular among many of our community members, but we do not see any benefits in using a specific framework for the scope of the Component Library. Staying with native technology will enable the library to be used in multiple contexts.

***Use Tailwind for styling UI Components in the UI Library***

We do not see any gains by enforcing contributors to learn Tailwind, the benefits of Tailwind are strongest when styles are applied for one DOM with no style encapsulations, in this project we do not have any shared classes across components. If so they will be purposely imported for the given cases.

***Use an existing UI Library***

There are many libraries out there that can solve our needs for basic UI components. There are none that deal with Umbraco-specific use cases. In order for us to have, as simple as possible, native Umbraco UI that deals with Nodes, Media, Trees, etc. We need to own the code ourselves. We expect to use parts from existing libraries in cases where it makes sense. (i.e. incorporating a third party DatePicker)

***Element name prefix***

In regards to choosing the prefix for element names ("<uui-...>")
We have considered other and shorter prefixes (u-, ui-, umb-) but uui- still looks like the best option in order to avoid potential collisions. We see many other UI Libraries using a single letter or a very short prefix and we want enable usage of such libraries. The use of "umb-" would collide with existing backOffice elements.


## Unresolved Issues
- Is this approach understandable?
- Are there aspects of this approach missing?
- Where is the line drawn between specific project components and UI Library components?


## Related RFCs 
- Overall process RFC about [Future-proofing the Umbraco backoffice](https://github.com/umbraco/rfcs/blob/main/cms/0021-future-proofing-the-umbraco-backoffice.md).

## Contributors
This RFC was compiled by:

* Niels Lyngsø (Umbraco HQ)
* Filip Bech-Larsen (Umbraco HQ)
* Mads Rasmussen (Umbraco HQ)
* Julia Gruszczynska (Umbraco HQ)
* Rune Strand (Umbraco HQ)


