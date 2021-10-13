# Define the Backoffice Extension API
Request for Contribution (RFC) 0023: Define the Backoffice Extension API.

## Code of conduct
Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience
The intended audience for this RFC is: 
* technical users
* developers
* package authors.

## Summary
- Extension API based on Web Standards and best practices without any ties to a specific framework.
- Web Components as the contract between the Backoffice and an extension.
- Registration through JSON manifests.
- Service-based architecture.
- Dependency injection through DOM events.
- Data management through events and the observable pattern.

## Motivation
At the very heart of the Umbraco CMS lives the desire to provide the most intuitive editing experience possible. It can only be done with a good toolkit for developers to create the best UI and functionality for their backoffice users. Being able to extend Umbraco is in the DNA of the CMS.

The challenge with the current extension API is it's too limited, which forces developers to rely on unofficial and undocumented hacks. That, in turn, means that the backoffice unofficially supports, or at the very least needs to consider these. For that reason it is difficult for newcomers to understand the current API, and makes it hard to know what can be safely used/changed.

Furthermore, the current API is tightly coupled to AngularJS, which means developers have to learn an additional (outdated) framework to extend/customize Umbraco, and it means that changes to the framework can break the API.

The purpose of this RFC is to validate the concepts of the new Extension API and make sure the new Backoffice will be flexible and extendable. The RFC presents the general concept of how the extension API will work and will be the foundation for all current official and future extension points.

## Out of Scope
- Choice of framework
- Implementation details

## Detailed Design
The Umbraco Backoffice is a complex application that is composed of features from independent packages, implementor code, and the core. Everything can be customized and extended: from Property Editors, to Content Apps and even full Sections with custom Trees and Node Editors. Each extension needs to be easily maintainable and independently upgradeable without the fear of breaking other parts of the Backoffice.

The new extension API is inspired by [Micro Frontends](https://micro-frontends.org/) - a design approach conceived to solve this problem. It is  rooted in the idea that a web application is a compilation of independent features that can be developed by different teams and then fused to create a homogenous interface. The general concept fits well with the functionality of Umbraco and has answers to some of the challenges we see today.

**The RFC is split into the following parts:**

1. Composition of the Backoffice
2. Communication between an extension and the Backoffice
3. Examples of extensions

All examples in the RFC are written in Vanilla Javascript. We have chosen to do it this way to make it clear that the API is technology agnostic and can be used with any framework or library. It will make them more verbose but we want to leave out the discussion of what framework the Backoffice will be built in until Part 3 of the project starts.

### Composition with Web Components
To prevent us from ending up with the same upgrade challenges as we face today, and to not force a specific framework choice on you, the Extension API can't rely on the component system of one specific framework. This would tie the extension API to a framework release cycle. A framework upgrade with breaking changes could result in a parallel rewrite of all extensions and packages.

Web Components introduce a neutral and standardized component model that we will use as the contract, or public API, between the Backoffice and an extension. Web Components are a widely implemented Web Standard, and many libraries exist to make development easier. The lifecycle methods introduced by Custom Elements make it possible to wrap the code in a standard way instead of custom lifecycle implementations. The features the API relies on are already there for the browsers that the backoffice supports.

Custom Elements and Shadow DOM both provide extra isolation features that make the extension more robust. That makes it possible for extensions to be self-contained and independently upgradeable.

It will be possible for extensions to coexist in the Backoffice, even if their technology stack is different. Each developer can build their extension using their web technology of choice and wrap it inside a Custom Element.

The following vanilla JS example shows how to scaffold a basic Backoffice extension:

```javascript
class MyUmbracoExtension extends HTMLElement {

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
 }

 render () {
   this.shadowRoot.innerHTML = `<div>My Umbraco Extension</div>`;
 }
}

customElements.define('my-umbraco-extension', MyUmbracoExtension);
```

### Registering extensions
Every extension comes with a manifest that provides important information to Umbraco about the extension, like its type, alias, resources, and more. Manifests are provided as JSON.

Manifests are already a part of the current way we register extensions for the Backoffice (package.manifest), and the new manifest will use a familiar format.

There will be two ways to register an extension for the Backoffice. Each method comes with its advantages.

* Manifest as JSON file as part of the filesystem.
* Registering runtime through Javascript.

This means it will no longer be possible to register extensions via C#.

Manifests as files are a low entry barrier for extensions. When using the filesystem, all manifest files will be picked up by the server and served to the frontend, where all registration will happen automatically. When registering through manifest files, the server can do high-level permissions and filter extensions that shouldn't be served to the frontend.

When registering runtime through javascript, you get full control over when an extension will be registered or removed. Backoffice will provide lifecycle events you can use for registration.

#### Common properties
Depending on the registration method used, all extensions are registered via either a JSON or JS object with a number of properties defining the relevant configuration for the extension.

These are the most common properties you will find for an extension. Other properties can vary depending on the extension type. We don't cover all possible properties in this RFC.

```jsonc
{
 "type": "", // type of extension
 "name": "", // unique name for the extension
 "alias": "", // unique alias for the extension
 "elementName": "", // unique name of the custom element
 "js": "", // path to the javascript resource
 "access": [] // the extension will only be served to the frontend if the currently logged-in user is part of a given user group.
}
```

The biggest difference from today is that you normally define an AngularJS template and a .js-resource. Instead you will define an elementName and a .js-resource. This will give you the flexibility to decide on your implementation of choice. Javascript resources will be lazy loaded when they are needed for an extension.

#### Registration with manifest files
The following example shows how to register a property editor.

**Manifest file**

/extensions/manifest.json
```jsonc
{
 "type": "propertyEditor",
 "name": "My Property Editor",
 "alias": "My.PropertyEditor",
 "elementName": "my-property-editor",
 "js": "/extensions/my-property-editor.js",
 "icon": "autofill", // specific for property editors
 "group": "Common", // specific for property editors
 ...
}
```

#### Registration through Javascript
The following example shows how to register and unregister a Property Editor through Javascript. We still need to provide Umbraco with an initial manifest file with our startup resources. After that, we can run any logic before we register the Property Editor.


/extensions/manifest.json
```jsonc
{
 "type": "startUp",
 "name": "My Start Up",
 "alias": "My.StartUp",
 "js": "/extensions/my-start-up.js"
}
```

**Register**
```javascript
const { extensions } = window.Umbraco;

const run = () => {
  const manifest = {
   type: "propertyEditor",
   name: "My Property Editor",
   alias: "My.PropertyEditor",
   elementName: "my-property-editor",
   js: "/app_plugins/my-property-editor.js",
   ...
 };

 // run any custom logic before you register
 extensions.register(manifest);
};

window.addEventListener('umbraco:authenticated, run);
```

**Unregister**
```javascript
const { extensions } = window.Umbraco;

// run any logic before you unregister 
extensions.unregister("My.PropertyEditor");

```

### Communication
Extensions and the Backoffice need to communicate with each other. When a value is updated in a Property Editor, when a dialog is opened, or when a tree node is expanded. In general, we see two types of communication. An extension communicating with the elements next to it (parent-child relationship) and cross UI communication.

#### Direct communication
For direct communication between an extension and the Backoffice, we want to follow the one-way data flow approach. Data flows downwards through props, and data should be updated by the parent itself by reacting to events of its children. Extensions will implement properties to get data and emit events to notify about any changes. Each extension will come with an interface they need to fulfill.

The native browser input field is a good example of how direct communication with an extension will be like. Listen for change events on the element and get the value from event.target.value.

The following example shows how an extension will implement similar functionality to communicate with the Backoffice.

```javascript
class MyUmbracoExtension extends HTMLElement {
 _value = '';
 
 // provide getter to read the internal property
 get value () {
   return this._value;
 }


 // provide setter to update the internal property
 set value (newVal) {
   this._value = newVal;
   this.render();
 }
 
 // We imagine having some internal logic to call the update method, such as a button click  within our components UI.
 _updateValue () {
   this.value = 'New Value';

   // when the value is updated we emit an event to notify that there are changes
   this.dispatchEvent(new CustomEvent('change', { "bubbles": true, "composed": true }));
 }

 render () {
   this.shadowRoot.innerHTML = `${this._value}`;
 }
```

In the examples section, you will find examples of how this will be used as part of a Property Editor and Property Action.

#### Cross UI communication

##### Services
The current Backoffice is built around a service architecture. We still believe this is the right design pattern for the new Extension API. Services give us the right amount of abstraction, they are well known to the current developer who extends the Backoffice, and we will follow the same pattern on both frontend and backend. That will make extending the Backoffice familiar to both frontend and backend developers who work with Umbraco.

The following example shows how to use the dialogService to open a dialog.

```javascript
this.dialogHandler = this.dialogService.open(dialogDefinedElsewhere);

this.dialogHandler.onSubmit().then(data => {
  // handle submit
  this.dialogHandler = null;
});
```

The following example shows how to use the notification service to show a success and an error notification.

```javascript
const message = 'My first notification';

this.notificationService.success(message);

this.notificationService.error(message);
```

##### Context and Dependency Injection
Using props and events works well when we want to communicate directly with an extension. Components can specify properties and attributes to receive data imperatively or declaratively. Things start to get more complicated when, for example, a tree structure is heavily nested, and a child component needs data from its parent, grandparent or an even more distant ancestor.

One approach is to pass data from one component to another down the DOM tree, also known as “prop drilling”. That is generally considered bad practice as it requires all child components to know of the data. For such cases, we want a mechanism where we can share context between extensions without having to explicitly pass a prop through every level of the tree. Parent components can serve as a dependency provider for all its children, regardless of how deep the component hierarchy is.

We have based our "Context API" / "Dependency injection" on a currently open [Context API proposal for Web Components](https://github.com/webcomponents-cg/community-protocols/blob/main/proposals/context.md). At a high level, the proposal is an event-based protocol that components can use to retrieve data from any location in the DOM:

* A component requiring some data fires a context-request event.
* vThe event carries a context value that denotes the data requested and a callback which will receive the data.
* Providers can attach event listeners for context-request events to handle them and provide the requested data.
* Once a provider satisfies a request it calls stopPropagation() on the event.

With this approach, we can provide dependencies on a global level and allow each dependency to be overwritten by any subtree. It makes it very useful in scenarios like Infinite Editing where every open editor lives in its own context.

##### Provide context
The following example shows how you can provide context for other components to consume. This is how you can provide a global service that will be available for the entire DOM. You can also restrict  a service so it can only be accessed by a specific subtree.

```javascript
const { provideContext } = window.Umbraco.context;

// provide context globally
provideContext(document, 'dialogService', new DialogService());

// provide context for a subtree
const myElement = document.querySelector("#myElement");
provideContext(myElement, 'dialogService', new DialogService());
```

This is how you will be able to provide context from a custom element.

```javascript
const { provideContext } = window.Umbraco.context;

class MySection extends HTMLElement {
  connectedCallback () {
    provideContext(this, 'myService', new MyService());
  }
}
```
##### Request context
An element that wishes to receive some context can use the requestContext helper function. The function takes a callback that will be resolved synchronously. A callback will also make it possible for the provider to do multiple resolutions if the provided value is updated. The element can then do whatever it wishes with this value. You can find more details in the [Context API proposal](https://github.com/webcomponents-cg/community-protocols/blob/main/proposals/context.md#could-we-use-promises-instead-of-callbacks).
The following example shows how you can request a dependency.


```javascript
const { requestContext } = window.Umbraco.context;

requestContext('dialogService', (dialogService) => {
  const dialogService = dialogService;
});
```

This is how you can request context from a Custom Element.

```javascript
const { requestContext } = window.Umbraco.context;

class MyEditor extends HTMLElement {
  this._myService = null;

  connectedCallback () {
    requestContext('myService', (myService) => {
      this._myService_ = myService;
    });
  }
}
```

#### Typescript Decorators
For the new extension API we want to provide Typescript decorators for those who want to use it. A decorator is a special kind of declaration that can be attached to a class declaration, method, accessor, property, or parameter. Decorators are currently a stage 2 proposal for JavaScript and are available as an experimental feature of TypeScript. They are one way we can encapsulate logic to expose an easier to use API. 

The following examples show how we can use the provide and context decorators in a custom element.

**Provide context**
```javascript
const { provideContext } = window.Umbraco.decorators;

class MyEditor extends HTMLElement {
  @provideContext('myService')
  myService: MyService = new MyService();
}
```
**Request context**
```javascript
const { requestContext } = window.Umbraco.decorators;
```

#### Data management and reactivity
Reactivity, or the phenomenon in which changes in the application data are automatically reflected in the DOM, is expected from a modern web application.

Today AngularJS takes care of all of this for us. Through its digest cycle, it automatically updates views and models whenever any of them changes. It is one of the main architectural and performance issues we face in the Backoffice today because any data model can be changed from anywhere.

For the new extension API, we want to leverage the service architecture and events. A service can be used to provide data to multiple parts of the application and through the Context API it can be requested from any place where the data is needed. We want to build the data management around the observer pattern where a service can offer a subscription model in which anyone can subscribe to an event and get notified when the event occurs.

When an extension wants data updates, it subscribes to the data service and will automatically get notified of any changes to the data. When it no longer wants to be notified of changes, it will unsubscribe from the service. The data in the service can only be modified by calling action methods on it, so any manipulation on the data is controlled and scoped much more tightly.

The following example shows how to use the content service to get a content item by its id. The custom element will get updates every time a new change is made to the content item as long as it is subscribed.


```javascript
const { requestContext } = window.Umbraco.context;

class MyEditor extends HTMLElement {
 _contentService = null;
 _subscription = null;
 _contentItem = null;

 connectedCallback () {
   requestContext('contentService', contentService => {
     this._contentService = contentService;
    
     this._subscription = this._contentService.getById(1).subscribe(contentItem => {
       this._contentItem = contentItem;
       this._render();
     });
   });
 }

 disconnectedCallback () {
   this._subscription.unsubscribe();
 }
}
```
It will also be possible to subscribe directly to the data source and do any custom logic before subscribing. The following example only gets updates if the same content item is published.

```javascript
...

this._subscription = this._contentService
 .getById(1)
 .filter(item => item.published === true)
 .subscribe(publishedItem => {
  
   this._contentItem = publishedItem;
});

...
```

### Extension examples
Below you will find examples of different extension points that are a part of the current Backoffice, and how they will be used with the new extension API. The examples do not cover all future extension points or functionalities, but they show different patterns we will expand to the rest of the Backoffice.

#### Dashboards
A dashboard is shown in the “main area” of every section of the Backoffice when you are on the tree root level. A section can contain multiple dashboards arranged over tabs. A dashboard can contain very complex functionality but the dashboard scaffold is the simplest extension we know in the Backoffice today.

The following example shows a dashboard registered for the “Content” section. It will be registered through a manifest file and served to the frontend only if the logged in user is part of the “admin” user group.

extensions/manifest.json
```jsonc
{
 "type": "dashboard",
 "name": "My Dashboard",
 "alias": "My.Dashboard",
 "elementName": "my-dashboard",
 "js": "/extensions/my-dashboard.js",
 "access": ["admin"], // only registered for user group "admin"
 "sections":  ["Umbraco.Section.Content"], // only added to the content section
}
```

extensions/my-dashboard.js
```javascript
class MyDashboard extends HTMLElement {

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
 }

 render () {
   this.shadowRoot.innerHTML = `<div>My Dashboard</div>`;
 }
}

customElements.define('my-dashboard', MyDashboard);
```
#### Property Editors
A Property Editor is the editor that a Data Type references. A Property Editor determines how a piece of content will be entered by a content editor in the Backoffice.

The following example shows how to build a text input property editor.

extensions/manifest.json
```jsonc
{
 "type": "propertyEditor",
 "alias": "My.PropertyEditor.Text",
 "elementName": "my-text-property-editor",
 "js": "/extensions/my-text-property-editor.js",
 "name": "My Text Property Editor",
 "icon": "autofill",
 "group": "Common",
}
```

##### Basic Scaffold
The scaffold sets up a Property Editor that renders a native input field.

extensions/my-text-property-editor.js
```javascript
class MyTextPropertyEditor extends HTMLElement {

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
 }

 render () {
   this.shadowRoot.innerHTML = `<input type="text" value="" />`;
 }
}

customElements.define('my-text-property-editor', MyTextPropertyEditor);
```

##### Value updates
In the following example, we add a value getter and setter. We also listen for events emitted when the native input field is changed and use them to update the local value property. When the value is updated we emit a custom event to notify the Backoffice about the new value.

extensions/my-text-property-editor.js
```javascript
class MyTextPropertyEditor extends HTMLElement {
 _value = '';

 get value () {
   return this._value;
 }

 set value (newVal) {
   this._value = newVal;
   this.render();
 }

 connectedCallback () {
   // add event listener for input changes
   const input = this.shadowRoot.querySelector('input');
   input.addEventListener('change', this._inputHandler);
 }

 _inputHandler = (event) => {
   this._value = event.target.value;
   // emit custom change event when the internal value is updated
   this.dispatchEvent(new CustomEvent('change', { "bubbles": true, "composed": true }));
 }

 render () {
   this.shadowRoot.innerHTML = `
     <input type="text" value="${this._value}" />
   `;
 }
}
```

##### Validation
A Property Editor implements a validation method that will be called by the Backoffice to validate the property value before it is sent to the server. The following example shows how the method uses the property configuration to validate the value.

extensions/my-text-property-editor.js
```javascript
class MyTextPropertyEditor extends HTMLElement {

 async checkValidity () {
   return this._config.validation.mandatory
   ? !!this._value
   : true
 }
}
```

The validation method can be asynchronous. The following examples show how you will be able to call any service to validate the property value.

extensions/my-text-property-editor.js
```javascript
class MyTextPropertyEditor extends HTMLElement {

 async checkValidity() {
   return this._myService.validate(this._value);
 }
}
```

##### Full Example
extensions/my-text-property-editor.js
```javascript
class MyTextPropertyEditor extends HTMLElement {
 _config = {};
 _value = '';

 get value () {
   return this._value;
 }

 set value (newVal) {
   this._value = newVal;
   this.render();
 }

 get config () {
   return this._config;
 }

 set config (newVal) {
   this._config = newVal;
 }

 async checkValidity () {
   return this._config.validation.mandatory
   ? !!this._value
   : true
 }

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
   const input = this.shadowRoot.querySelector('input');
   input.addEventListener('input', this._inputHandler);
 }

 _inputHandler = (event) => {
   this._value = event.target.value;
   this.dispatchEvent(new CustomEvent('change', { "bubbles": true, "composed": true }));
 }

 render () {
   this.shadowRoot.innerHTML = `
     <input type="text" value="${this._value}" />
   `;
 }
}
```

#### Property Actions
Property Actions are secondary functionalities of a property Editor. They appear as a small button next to the label of the property. The Backoffice comes with a couple of built-in Property Actions. The following examples show how similar actions could be made with the new extension API.

##### Property Action: Clear the property value
The following example shows how to build a Property Action that clears the property value.

extensions/manifest.json
```jsonc
{
  "type": "propertyAction",
  "alias": "My.PropertyAction.Clear",
  "elementName": "my-property-action-clear",
  "js": "/extensions/my-property-action-clear.js",
  "name": "Remove all items",
  "propertyEditors": ["My.PropertyEditor.Text"], // register for My.PropertyEditor.Text
  "weight": 10
}
```

extensions/my-property-action-clear.js
```javascript
class MyPropertyActionClear extends HTMLElement {
 _value = [];

 get value () {
   return this._value;
 }

 set value (newVal) {
   this._value = newVal;
   this.render();
 }

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
   // add event listener for button clicks
   const button = this.shadowRoot.querySelector('button');
   button.addEventListener('click', this._clickHandler);
 }

 _clickHandler = () => {
   // when the button is clicked we clear the internal value and emits a change event
   this._value = [];
   this.dispatchEvent(new CustomEvent('change', { "bubbles": true, "composed": true }));
 }

 render () {
   this.shadowRoot.innerHTML = `<button>Remove all items</button>`;
 }
}
```

##### Property Action: Copy to clipboard
The following example shows how to build a Property Action that stores a copy of the property value in the internal clipboard service. It uses the Context API to access the clipboard service, the current property name, and the current node name.

```javascript
const { requestContext } = window.Umbraco.context

class MyPropertyActionCopy extends HTMLElement {
 _currentNode = null;
 _currentProperty = null;
 _clipboardService = null;
 _value = [];

 get value () {
   return this._value;
 }

 set value (newVal) {
   this._value = newVal;
   this.render();
 }

 constructor () {
   super();
   this.attachShadow({mode: 'open'});
 }

 connectedCallback () {
   this.render();
   const button = this.shadowRoot.querySelector('button');
   button.addEventListener('click', this._clickHandler);

   // Get property context
   requestContext('property', (property) => {
     this._currentProperty = property;
   });

   // Get node context
   requestContext('node', (node) => {
     this._currentNode = node;
   });

   // get clipboardService
   requestContext('clipboardService', (clipboardService) => {
     this._clipboardService = clipboardService;
   });
 }

 _clickHandler = () => {
   // use context to get the name of the property and the node
   const title = `${this._currentProperty.name} from ${this._currentNode.name}`;
   this._clipboardService.copy(title, this._value);
 }

 render () {
   this.shadowRoot.innerHTML = `<button>Copy</button>`;
 }
}

window.customElements.define('my-property-action-copy', MyPropertyActionCopy);
```

#### Trees
The main navigation in Umbraco consists of sections. Each section has a sub-navigation which can consist of one or more trees. Today it is possible to register new trees only through C#. With the new extension API, setting up trees for the Backoffice will be handled client-side. A tree can fetch data from anywhere, inside or outside of Umbraco.

The following example shows how to register a Tree, and how to use a Custom Element to set up the tree.

extensions/manifest.json
```jsonc
{
 "type": "tree",
 "alias": "My.Tree",
 "elementName": "my-tree",
 "js": "/app_plugins/my-tree.js",
 "name": "My Tree",
 "section": "My.Section",
 "group": "My Tree Group",
 "weight": 10
}
```

Trees have complex markup and logic to handle recursive nodes, context navigation, etc. We will provide a Tree base class that custom trees can extend. We don’t want every extension to implement that logic.

In the example below the Tree implementation will override methods on the base class to replace the logic responsible of fetching the tree data. We will provide helper functions to easily create tree nodes.

extensions/my-tree.js
```javascript
const { UmbTreeBase, createTreeRootNode, createTreeNode } = window.Umbraco.tree

// extends the Tree base class which provides markup and basic functionality
class MyTree extends UmbTreeBase {
 // creates a root node named "My Tree" with children
 createRootNode = async () => createTreeRootNode('My Tree', true);

 // returns a collection of tree Nodes
 getTreeNodes = async (id: number) => {
   // you can get your data from anywhere, and they can represent anything...
   const items = [
     {
       id: 1,
       name: "My Item 1"
     },
     {
       id: 2,
       name: "My Item 2"
     }
   ];

   // loop through the items and create a tree node for each one
   const treeNodes = items.map((item: any) => createTreeNode(item.id, item.name, item.hasChildren));
   return treeNodes;
 }

}
```

#### Future extensions points
The current extension API is too limited and this RFC only covers a selected number of extensions. The final Backoffice implementation will cover all current official extension points and a subset of new ones. The API concept described in this RFC is the foundation of how future extension points added to the Backoffice will work.

These are a few examples of extension points that are not currently available in the Backoffice, but could be added in the future.

Extend the publish actions on a node with a custom action. Here we want to add a review process before publishing a node.

```jsonc
{
 "type": "publishAction",
 "alias": "My.PublishAction.Review",
 "trees": ["Umbraco.Tree.Content"], // registered for the content tree
 ...
}
```

Add the same actions as a bulk action on the default content list view.

```jsonc
{
 "type": "listViewAction",
 "alias": "My.ListViewAction.Review",
 "listViews": ["Umbraco.ListView.Content"], // registered for the content list view
 ...
}
```

Allow “Content”-apps to be extendable in all trees.

```jsonc
{
 "type": "nodeApps",
 "alias": "My.UserNodeApp.ApiKeys",
 "trees": ["Umbraco.Tree.Users"], // registered for the users tree
 ...
}
```

## Unresolved Issues
### Extension points to be included
The aforementioned examples show how the extension API will cover all current official extension points and any future extension points we will include in the Backoffice. The initial research for the new extension API showed that AngularJS interceptors and decorators are two very popular ways of modifying the current Backoffice. They are often used in lack of better options and are usually hard to maintain. With the new extension API, this won't be possible anymore. First and foremost we want to make sure we include a broad range of extension points in the Backoffice. [We would still love to hear](https://github.com/umbraco/DefineBackofficeExtensionAPI) what extension points are currently missing, so we can consider these.

### Extension point overrides
A similar concept of swapping out a view is possible with the new extension API, where you change an element name and the resource in a currently registered extension manifest. This way you will be able to override the implementation of another part of the Backoffice. Since it is not settled how the extension API will be implemented, and this approach comes with some of the same downsides as today (extensions can override each other), we haven't included some of the “hacky” extension points in this RFC. If you have inputs to how we can do this in a "safe" way we are all ears. We will open up for discussion when we have a proposal for how to do this.

## Contributors
This RFC was compiled by:

- Mads Rasmussen (Umbraco HQ)
- Niels Lyngsø (Umbraco HQ)
- Julia Gruszczynska (Umbraco HQ)
- Filip Bech-Larsen (Umbraco HQ)
- Rasmus John Pedersen (Umbraco HQ)
- Rune Strand (Umbraco HQ)
- Kenn Jacobsen (Backoffice Community Team)
- Matt Brailsford (Backoffice Community Team)
- Ryan Helmn (Backoffice Community Team)
- Sian Simms (Backoffice Community Team)
- Søren Kottal (Backoffice Community Team)
