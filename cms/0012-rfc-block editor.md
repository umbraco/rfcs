# RFC Name

Request for Contribution (RFC): 0012-block-editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience:
 
* Umbraco developers
* Umbraco editors

## Summary

A new backoffice editor that handles common page structure editing in a simpler and intuitive way. The main concept of the editor is to manage list of blocks that represents a web page's structure, where each block is both a collection of content and a way to configure that collection to be rendered. Each content block and configuration is defined by an Element Type to make administrating and editing with the new Block editor consistent.

This editor aims to be an alternative to the popular editors in Umbraco version 7 such as Stacked Content and Nested Content. This editor also aims to be a foundation for other complex editors in the future. It's components will be able to be re-used to develop alternatives to other popular Umbraco 7 editors such as The Grid, LeBlender, and Doc Type Grid Editor.

![\[IMG\]](assets/GridStyleexamples1.jpg)

## Motivation

* We need a good foundation for building complex editors with a strongly typed data structure. This foundation will eventually power a new grid based editor.
* We need a new solution for V8 especially since the popular alternatives such as Stacked Content are not yet v8 ready and the new block editor should be a good replacement.
* Umbraco currently doesn't have a 'page builder' type of editor which it needs. The current Grid implementation seems like a 'page builder' but it is not designed this way and trying to make it work as a 'page builder' is cumbersome.

## Detailed Design

This editor is about managing list of blocks to build entire pages. In this example you see the backoffice on the left side, and the visual representation on the right side:

![\[IMG\]](assets/GridStyleexamples2.jpg)

- The editor has the option to append or insert specific types of blocks anywhere in the editor.
- Blocks can be edited, ordered or removed.
- Each block consists of both content and config. The "Content" portion of a block is the normal content being edited such as: Title, Description, etc... The "Config" portion of a block is mainly used for visual styling, for example: Layout (right, left, wide, box), Background color, etc...


### Working with content

Working with content for each row/cell will be done by clicking on an edit button to launch the content editor panel (using infinite editing, similar to the way LeBlender and Doc Type Grid Editor used to do in v7). Each row/cell can have it's own configuration applied to it and editing the configuration is achieved in the same way as editing content but instead by clicking on a configuration button. 

By editing each row/cell in the side panel it means we can render all property types for an Element Type and all Property Groups. This is unlike the editing experience with Nested Content where we can only render properties on a single group.

![\[IMG\]](assets/GridStyleexamples4.jpg)

#### Default rendering

There wil be a default rendering of blocks in the backoffice. Each block will be represented by a row showing the name of the block, it's icon along with the edit/config/delete buttons. The default rendering will also allow for nested blocks if the developer chooses to allow nested block element types.

#### Customized rendering

The block editor data type configuration will allow the developer to define a custom view for each block type. This will be based on the developer choosing a base folder location that will contain `.html` files for each block type. The name of the files are by convention and must correspond to the Element type alias. Umbraco's default rendering will render each block and if a custom view is found for the block type it will render that view for a given block. 

A more advanced functionality will also exists where a develoer can completely customize how an editor visualizes and works with the content blocks by specifying a custom `.html` view file for rendering the entire editor. If a custom view is specified the developer can customize how the editor visualizes and works with the content and in most cases it will look like and represent what will be rendered on the front-end. As an example, the image above is using a custom view to render out the full editor.

The aim is to make implementing a custom view either per block or for the entire editor easy. The developer can re-use as many components as possible to get a customized view working quickly. This includes standardized components that allow for consistent validation practices along with consistent re-ordering (drag/drop) of blocks within the view. This will of course require documentation.

#### Unsupported editors

There will be some unsupported editors in the Block Editor, like there is today with Nested Content. Typically these are file based editors: Upload, Image Cropper. These will not be supported in the block editor and warnings will be shown if attempted to be used (like Nested Content).

### Setting up the block editor

The way to set a new block editor is quick similar on how nested content work:

1. Create new Element Types, one for each type of "Content" block and one for each type of "Config"
1. Create a new data type using the "block editor" property editor and configure it to use the previously created Elements Types for your "Content" blocks and then choose their associated "Config" types
1. Use this new data type as property type for your document types

#### Naming blocks

Nested Content allows naming it's rows by using an AngularJs expression within the Data Type Configuration. The Block Editor will also support this since it provides the most flexibility however another simpler optino will be available. Block names can be generated based on the developer selecting which property type to use as the block name OR enter an AngularJs expression. Since a block stores both "content" and "configuration", it will be possible to name a block based on property data in either of these collections.   

By default the block name will be it's Index.

### Complex layouts?

The block editor stores an array (linear list) of data. This will work for most pages structures since developers can apply any custom styling they want to render this data. However in some cases a page's structure may be more complicated and in those cases it is certainly possible to put another block editor inside of a block (nested block editors). Since each row/cell contains it's own config, this config can be used to define any number of layout options.

**Example:** if a separation of blocks into columns is needed, a first level of blocks can be created as "splitters" and then "Content" block can be placed into them. 

![\[IMG\]](assets/GridStyleexamples5.jpg)

### Data model

The data model underpinning all of this is based on another RFC: https://github.com/umbraco/rfcs/blob/master/cms/0011-block-data-structure.md

This editor will store an array of elements within it's `layout` property.

Each block has it's own UDI (see [previous RFC for more info](https://github.com/umbraco/rfcs/blob/master/cms/0011-block-data-structure.md))

## Drawbacks

This editor may be competing with community built editors such as Stacked Content and Doc Type Grid Editor, however these editors are not yet v8 ready and it seems like this is a much needed feature in the CMS.

## Alternatives

- Create a new Grid version V2, simpler, more usable and with structured data. However, we think the block editor would be a good start to getting everything in the core prepared for a new v2 of the grid since that will require everything (and more) than what this editor requires and we think that the block editor and a v2 grid could complement each other. 
- Keep the Grid as it is, but adding Doctype grid editor into the core (v8)with a visual editor into the backoffice to configure it (like Leblender). We don't feel like this is a great solution since we want to make the editor experience and this feature really shine, whereas trying to move DTGE into the core now isn't going to get us closer to this goal.

## Out of Scope

* The new implementation of the grid. This RFC is not about creating a new grid v2. 
* Dealing with culture variance and multi-lingual implementations. The initial build of the block editor with be an MVP (Minimum viable product). In the future we will look into how this block editor can support content variance at the block level.
* Sharing a block's content between different content items or editors. In the future this may be possible but is not part of the initial implementation.
* Inline editing of content blocks, either on the front-end when previewing content or in the backoffice. Content blocks will only be edited by the slide out panel.
* Block "security" per user group - this can be done with an event by modifying the data type configuration

## Unresolved Issues

* Validation: We will need to prototype validation for block editing. We have the capability to do this in the CMS now but we will need to enhance/simplify the implementation and make sure that it's consistent. This goes for both client side and server side validation. There will be some challenges with this too since editing of blocks can be done a couple of different ways: inline vs contextual
* The upload control is notorious for not working with these types of editors, it will probably remain that way but we'll need to deal with that somehow
* Need to determine how a block is named, with Nested Content this is done with an angular template which is not very intuitive but it could also be an option
* Need to determine the nicest way to add block data to the Examine indexes and how fields should be named.


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

