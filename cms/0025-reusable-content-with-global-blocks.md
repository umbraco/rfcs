# Reusable Content with Global Blocks

### Request for Contribution (RFC) 0025 : Reusable Content with Global Blocks

**Please comment on this RFC in this [RFC Dicsussion](https://github.com/umbraco/rfcs/discussions/35)**

## Code of Conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md).

## Intended Audience

The intended audience for this RFC is

- Technical users
- Developers
- Package authors
- Editors


## Summary

We want to create Global Blocks that can be referenced from multiple Block Lists or Block Grids, and we want these to be editable from the Content Section as well as a Global Block tree in a dedicated section. This new section should include "global content that the editor can/should maintain". We propose naming this new section "Library" and include global Blocks, Tags, and Content Templates. Within the Library section, the Global Blocks should be maintained using the same (or similar) conventions and functionality as in the Content Section - e.g. functionality to publish, unpublish, schedule a block, and handle language variations.

## Motivation

Since the very beginning of Umbraco, we have thrived to create a flexible CMS that gives both developers and content editors the freedom to be creative. The introduction of Blocks, via first the Block List editor and later the Block Grid, is one of the concepts that support this journey. As we now have multiple Property Editors using Blocks, we see an opportunity for introducing even more flexibility and making it easier to reuse and maintain Block content specifically, and content in general. 

We believe Global Blocks will help increase flexibility for developers by adding more options for constructing a good editing experience, and for content editors as they can now publish, unpublish and schedule the publishing of Block content. In addition, maintaining a single Global Block will be easier, as you can do it in one place while having the Blocks that are referenced in multiple places, introducing a more streamlined content model which helps the editor be more effective and consistent.

## General Design

Blocks can now be made local or global. Local Blocks work in the same way as current Blocks. Global Blocks can be created and edited in the same way as Local Blocks, or via a new “Library” section.
To facilitate a good experience all Block content, whether it is local or global, will be handled as standard content. This means storing it in the same way in the database, providing similar actions for managing publish state, variants, and so forth.

## Detailed Design

### Library Section

We want to offer a section where Global Blocks can be maintained, however, doing this, opens up the question if other types of content are placed the right way today. Global Blocks are by nature something that is not related to a specific page. It's used on multiple pages but that's also the case for tags, and content templates. We, therefore, suggest that the new Library section also includes these types. An example of such a section could look like this:

![Library Section](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Library%20-%20clean.jpg?raw=true)

As seen, we want to introduce a folder structure for the global blocks so that the editor has a way to control the structure.
Also be aware, that while Blocks have both Content and Settings, we only consider the Content part to be Global as we believe the Settings part could benefit from being specific for each usage of a Global Block. We, therefore, don’t plan to show any Settings for blocks in the Library section.  

#### Preview

As a block will always be shown and styled depending on the content wherein it exists, it will never make sense to try to preview a Block without also choosing a Content Node. The editor should therefore be asked to select the Content Node before an actual preview can be shown:

![Library Preview](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Library%20-%20Preview.jpg?raw=true)

In case a Block has not yet been referenced from any Content Node, a message should be shown to the user, that a Global Block can't be previewed when it's not yet in use on any Content Items.

#### Info tab

Like with Content Nodes, we want to show the general status and history, and we also want to show a list of the Content Items where the Block is referenced. That list should include information about the status of the Content Nodes concerning published, unpublished, scheduled, created and latest changes so that the editor can be guided in updating the right content.

![Library Info tab](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Library%20-%20info.jpg?raw=true)

#### Language

In short, there should be the same opportunities for languages as with Content Nodes - e.g. with split-view and with publishing messages:

![Library Info tab](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Library%20-%20language.jpg?raw=true)

### Content Section

We believe that Local Blocks still have relevance on sites, where you want to limit complexity. We, therefore, want to offer a setting on the Block List and the Block Grid to indicate if it accepts Local and/or Global Blocks. However, as we also see use cases for Block Lists/Grids with both Local and Global Blocks, we want to present this in a friendly way in the UI. This could either be through labels, icons, or both - and should of course work with both the Block List and the Block Grid:

![Content Section - block list](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20clean.jpg?raw=true)
![Content Section - block grid](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20block%20grid.jpg?raw=true)

##### Make Global

We want to ensure a smooth transition between Local and Global Blocks, so that if you have a Local Block that you want to use in more than one Block List or Block Grid, - it should simply be possible to make it global. In this process, you need to give the Block a name and also place it in the correct folder - and the block will afterward be available in the Library folder:

![Content Section - make global](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20make%20global.jpg?raw=true)

#### Make Local

While using Global Blocks there might be cases, where the Global Block is almost as you want it - but somehow not 100% the right fit. In these cases, we want to offer an opportunity to make a Block local. By doing this we'll create a new local instance while still keeping the Global Block:

![Content Section - make local](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20Make%20local.jpg?raw=true)

#### Publish content

With Global Blocks that can have various states and e.g. be unpublished or scheduled for publishing, we want to make it visible for the editor during the publishing of a content node if there are Global Blocks that are either unpublished or scheduled for publishing. As the Global Blocks are published, unpublished, and scheduled individually they will not affect the publishing of a Content Node, but we see the information about these states as important for the editor:

![Content Section - publish](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20publish.jpg?raw=true)

#### Edit block

The experience of editing a Block in the context of a Content Node should be the same as with Local Blocks today, however, instead of a submit button we want to provide the editor with the option to save, unpublish, schedule, and publish.

![Content Section - edit block](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20Edit%20block.jpg?raw=true)

#### Add block

We want the editor to be able to decide up-front if she would like to create a new Local/Global Block or add an existing Global Block to the currently selected Block List or Block Grid. If creating a new Global Block, we want that to be done within the Block tree:

![Content Section - add block](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20Add%20block.jpg?raw=true)

#### Language

As mentioned above, we want to provide the same language opportunities for Global Blocks as for Content Nodes. A Global Block should therefore always be visible in all languages, although its content might fall back to the default language. Contrary, a Local Block - just as it is today - is considered unique for a specific language.

![Content Section - add block](https://github.com/umbraco/rfcs/blob/0025-reusable-content-with-global-blocks/cms/assets/global-blocks/Content%20-%20splitview.jpg?raw=true)


### Permissions

In short, we want to offer the same permissions as exist today on Content Nodes, - meaning that there should be permissions for browse, delete, create, notifications, publish, permission changes, send to publish, unpublish, update, etc. - and that these permissions should be available to set both as default permissions and as specific granular permissions for a node. Also, like with content and media, we want to offer opportunities for limiting the Block Library Tree to a specific start node.

## Technical considerations

We propose creating a similar data structure for Global Blocks as we have it for regular Content Nodes and thereby adding all the same options as with Content Nodes, except for routing. That also means that it should e.g. be possible to find deleted Global Blocks in the recycle bin. 
As we think it makes sense to have the Block Settings as something that relates to the specific usage of a Global Block Content, we only consider the Content part to be stored in this “new” structure, whereas we suggest storing the Settings part within the Content Node - as we do it today. 

## Out of Scope

Although we mention Tags as part of the new Library section, it's not in the scope of this RFC to discuss moving and updating Tags.

### Unrosolved issues

Some of the answers that we hope to get out of this are:
- Related to the technical considerations, if we are right, that it makes most sense to have Block Settings stored within the Content Node or if there are more/better use cases for storing Block Settings globally too. 
- In case you work with segments, if you can see any implications in this proposal. 
- More perspectives on the proposed new section called “Library”. If it should be there at all and if so what it should include.
- The naming. Should we call it Global Blocks, resusable content or something else.

## Contributors

This RFC was compiled by:

- Bjarke Berg (Umbrac HQ)
- Jacob Overgaard (Umbraco HQ)
- Filip Bech-Larsen (Umbraco HQ)
- Rune Strand (Umbraco HQ)
- Lasse Fredslund (Umbraco HQ)

- The CMS Group
- The CMS Community Team

