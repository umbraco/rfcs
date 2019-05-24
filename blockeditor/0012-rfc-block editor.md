# RFC Name

Request for Contribution (RFC): 0012-block-editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is mainly for editors and segundly for developers.

## Summary

Block Editor is a new data type to build pages on the fly with a collection of block of content. This is not a new version of the current Umbraco data type grid, but an alternative to it, and to others packages like stack content, doc type grid editor, leblender, etc. 
The main concept of the editor is to manage list of blocks that represents the web page structure. It is mainly focus on editing page with landing page layout style, as for instance the Umbraco.com home page.
The editor experience with the editor is very simple, fast and intuitive: He adds block under and between others, edits its content properties (block data values), and its setting properties (define the presentation and styling of the block)

![\[IMG\]](assets/GridStyleexamples1.jpg)

The main goals are:

 - Short term:
	 - Having a new editor that cover the most common page structure edition in a very simple and intuitive way, potentially replacing bad grid implementation and makes the editors' lives easier.
 - Long term
	 - Add a new feature to create content variance at block level to make content personalization.
	 - Make possible to share block’s content between different blocks in different places.  
	 - Give the editor a very good inline editing experience directly into the page context.

## Motivation

 - The current Grid sucks (just a bit)... its implementations is
   completely random, not reusable, and doesn’t reach its main purpose:
   make editors lives easier. 
- It very challenging for the editor with the current grid to know what
   he is currently editing.
- Creating a new page from scratch with the grid is too much clicks and
   time consuming, and it's hard to maintain.
- Grid data are not structured, hard to index and impossible to reuse.
- The grid is usually too flexible and become too complex and hard to
   work with.
- The current grid add dependencies between the CMS and how the design
   or UI have to be done. 
- We need a new solution for V8 in margin of the
   current grid.

## Detailed Design

The block editor is a specific implementation of the Proteus blocks (refer to RFC 11).

As mentioned before, this editor is about managing list of block to build entire pages. In this example you see the backoffice on the left side, and the visual representation on the right side:

![\[IMG\]](assets/GridStyleexamples2.jpg)

- The editor have the option to add specific types of block in any place between blocks.
- Blocks can be edited, ordered or removed.
- Each type of block has his setting, mainly used for visual styling, but does not contain content, as for instance:
	- Layout: right, left, wide, box, etc.
	- Background color
	- etc. 
	
- Block content are the properties data for the block:
	- Title
	- Description
	- List of element
	- etc.

Also, there are two different ways to work with the block editor (configurable by setting):

1. **Standard display:** Blocks can be edited inline on form format (the way NestedContent works)
![\[IMG\]](assets/GridStyleexamples3.jpg)
2. **Contextual display:** Blocks can be edited into the Umbraco right pane (infinite editing, the way LeBlender and DTGE used to do in v7) with a preview of the block on the main panel. The editor click on the block, and the right panel open with its properties.
![\[IMG\]](assets/GridStyleexamples4.jpg)

**How to create and set a new block editor?**
The way to set a new block editor is quick similar on how nested content work:
4. Create Document Types as Element that composed the block’s properties
5. Create a new data type “block editor” and set  its settings:
	a. Type of visualisation: standard or contextual
	b. Choose the previously created Elements and define their settings
6. Use this new data type as property type into your document types 

**How to handle column and more complex layout?**
The standard way to use the block editor must cover most of the page structure cases and doesn’t cover the column handling. However, that can be done easily including a more level of blocks, block 
into block.

**Example:** if for instance, a separation of blocks into columns is needed, a first level of block can be created as “splitters” and then “Content” block can be placed into.
![\[IMG\]](assets/GridStyleexamples5.jpg)

## Drawbacks

Because this project is not about rebuilding the grid (yet), we don’t have any solution in very short term to cover Grid uses with Leblender or Doctype grid editor on V8. Our recommendation is:
Use the Umbraco Grid as it is out of the box.
Use Nested content and DTGE (not yet compatible with V8)) with the grid to build your component.

## Alternatives

- Create a new Grid version V2, simpler and more usable, but with structured data.
- Just keep the Grid as it is, but adding Doctype grid editor into the core with a visual editor into the back office to configure it (like Leblender).

## Out of Scope

- The new implementation of the grid. This RFC is not about creating a new grid v2. 

## Unresolved Issues

- When the data is in “Editing in inline mode” (The way Nested Content works). How should be data validation be done?
- Where should we offer the preview mode and inline editing of the page:
	- Data edition with forms into the back office (main panel), preview in front as it is now.
	- Data edition in forms into the back office (right panel), preview into the back office (main panel) and also in front as it is. This the solution described before.
	- Preview and inline edition into the back office (main panel) and also in front as it is.
	- Data edition in forms into the back office, preview and inline edition also in front.

## Related RFCs 

RFC 11: Proteus
- [RFC 11: Proteus](https://github.com/umbraco/rfcs/blob/731e1872323a2e3a8c4cd8c91524c795f3c5efa3/cms/0000-proteus-block-editor.md)

## Contributors

This RFC was compiled by:

- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/aaantoinee) (community)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)

