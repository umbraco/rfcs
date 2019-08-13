# RFC Name

Request for Contribution (RFC): 0012-block-editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is mainly for editors and segundly for developers.

## Summary

A new back-office editor that handles common page structure editing in a simple and intuitive way. The main concept of the editor is to manage list of blocks that represents a web page's structure, where each block is both a collection of content and a way to configure that collection to be rendered. Each content block and configuration is defined by an Element Type to make administrating and editing with the new Block editor consistent.

This editor aims to be an alternative to the popular editors in Umbraco version 7 such as the Grid, Stacked Content, LeBlender and Doc Type Grid Editor.

![\[IMG\]](assets/GridStyleexamples1.jpg)

## Motivation

- The current Grid implementation has issues and although it seems like a 'page builder' type of editor it is not. 
- It very challenging for the editor with the current grid to know what
   they are currently editing.
- Creating a new page from scratch with the grid requires too many clicks, is 
   time consuming and the content is hard to maintain.
- The Grid data is not structured, hard to index and impossible to reuse.
- The grid is usually too flexible and become too complex and hard to work with.
- The current grid adds dependencies between the CMS and how the design or UI has to be done. 
- We need a new solution for V8 especially since the popular alternatives such as Doc Type Grid Editor and Stacked Content are not yet v8 ready and we are hoping to make support for such editors more robust with the new block editor.

## Detailed Design

This editor is about managing list of block to build entire pages. In this example you see the backoffice on the left side, and the visual representation on the right side:

![\[IMG\]](assets/GridStyleexamples2.jpg)

- The editor has the option to append or insert specific types of blocks anywhere in the editor.
- Blocks can be edited, ordered or removed.
- Each block consists of both content and config. The "Content" portion of a block is the normal content being edited such as: Title, Description, etc... The "Config" portion of a block is mainly used for visual styling, for example: Layout (right, left, wide, box), Background color, etc...


### Working with content

There are two different ways to work with the block editor (configurable by setting):

1. **Standard display:** Blocks can be edited inline on form format (the way NestedContent works)
![\[IMG\]](assets/GridStyleexamples3.jpg)
1. **Contextual display:** Blocks can be edited into the Umbraco right pane (infinite editing, the way LeBlender and DTGE used to do in v7) with a preview of the block on the main panel. The editor clicks on the block, and the right panel opens with its properties. 
![\[IMG\]](assets/GridStyleexamples4.jpg)

In order for the contextual display to work, a custom stylesheet will need to be applied to the datatype which will be used when rendering the contextual editor.

### Setting up the block editor

The way to set a new block editor is quick similar on how nested content work:

1. Create new Element Types, one for each type of "Content" block and one for each type of "Config"
1. Create a new data type using the "block editor" property editor configure it:
	* Type of visualisation: standard or contextual
	* Choose the previously created Elements Types for your "Content" blocks and then choose their associated "Config" types
1. Use this new data type as property type for your document types

### Complex layouts?

The block editor simply stores an array (linear list) of data. This will work for most pages structures since developers can apply any custom styling they want to render this data. However in somce cases a page's structure may be more complicated and in those cases it is certainly possible to put another block editor inside of a block (nested block editors).

**Example:** if a separation of blocks into columns is needed, a first level of blocks can be created as "splitters" and then "Content" block can be placed into them. 

![\[IMG\]](assets/GridStyleexamples5.jpg)

### Data model

The data model underpinning all of this is based on another RFC: https://github.com/umbraco/rfcs/blob/master/cms/0011-block-data-structure.md

This editor will store a simple array of elements within it's `layout` property.

## Drawbacks

This editor may be competing with community built editors such as Stacked Content and Doc Type Grid Editor, however these editors are not yet v8 ready and it seems like this is a much needed feature in the CMS.

## Alternatives

- Create a new Grid version V2, simpler, more usable and with structured data. However, we think the block editor would be a good start to getting everything in the core prepared for a new v2 of the grid since that will require everything (and more) than what this editor requires and we think that the block editor and a v2 grid could complement each other. 
- Just keep the Grid as it is, but adding Doctype grid editor into the core (v8)with a visual editor into the back office to configure it (like Leblender). We don't feel like this is a great solution since we want to make the editor experience and this feature really shine, whereas trying to move DTGE into the core now isn't going to get us closer to this goal.

## Out of Scope

* The new implementation of the grid. This RFC is not about creating a new grid v2. 
* Dealing with culture variance and multi-lingual implementations. The initial build of the block editor with be an MVP (Minimum viable product). In the future we will look into how this block editor can support content variance at the block level.
* Sharing a block's content between different content items or editors. In the future this may be possible but is not part of the initial implementation.
* Inline editing of content blocks on the front-end when previewing content.

## Unresolved Issues

* Validation: We will need to prototype validation for block editing. We have the capability to do this in the CMS now but we will need to enhance/simplify the implementation and make sure that it's consistent. This goes for both client side and server side validation. There will be some challenges with this too since editing of blocks can be done a couple of different ways: inline vs contextual
* Contextual display: Need to investigate the best way to inject custom styles to render the block editor in the main panel so that it renders the way a developer wants it to be displayed so that it can look like how it will be rendered on the front-end. Currently the idea is to configure a custom angular view to display this 'preview' but it seems like this needs to be much easier to do that creating an entirely brand new view with all of the required functionality for previewing (and any interactions)
* The upload control is notorious for not working with these types of editors, it will probably remain that way but we'll need to deal with that somehow
* Need to determine how a block is named, with Nested Content this is done with an angular template which is not very intuitive but it could also be an option
* How will we deal with Groups (tabs)? In the contextual format this is fine because the groups are accordian based and they just render as a normal content editor, however in the Standard mode how will this work?


## Related RFCs 

RFC 11: Block data structure
- [RFC 11: Proteus](https://github.com/umbraco/rfcs/blob/master/cms/0011-block-data-structure.md)

## Contributors

This RFC was compiled by:

- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/aaantoinee) (community)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)
- [Shannon Deminick](https://twitter.com/shazwazza) (HQ)

