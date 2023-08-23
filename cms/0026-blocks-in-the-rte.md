# Blocks in the RTE

### Request for Contribution (RFC) 0026 : Blocks in the Rich Text Editor (RTE)

**Please comment on this RFC in this [RFC Discussion](https://github.com/umbraco/rfcs/discussions/36)**

## Code of Conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md).

## Intended Audience

The intended audience for this RFC is

- Technical users
- Developers
- Extension developers
- Package authors

## Summary
With the new backoffice (Bellissima) currently targeted for Umbraco 14, we want to remove macros and, consequently, with the release of Umbraco 13m deprecate macros and introduce an alternative.
 
We have good alternatives to macros in most use cases, but we are still missing the use case where the editor can insert complex content in the Rich Text Editor (RTE) that is styled and controlled by the developer - e.g. inserting a call to action button within the RTE. 

In this RFC we propose a solution for inserting complex content into the RTE based on Blocks.

## Motivation
We have for some time wanted to avoid implementing Macros for the New Backoffice, as the approach is a bit misaligned with the current state of Umbraco. We know, that for some developers it has worked fine and used frequently, but for many developers, it has been an approach “a bit out of context” and not as intuitive as we would like it to be. 

We have investigated the usage of macros and identified a few use cases where there currently lacks an alternative to the functionality provided by Macros. 

We believe these remaining use cases are solved by introducing blocks in the RTE.
As we see more and more usage of blocks and e.g. are looking into [Reusable Content with Global Blocks], it makes sense to build this functionality based on blocks so that we can benefit from future improvements and functionality related to blocks.

## Detailed Design
We suggest introducing Blocks in the RTE by offering functionality similar to the Block List and the Block Grid editors. 

#### Editor Settings
Like inserting a macro today, we propose an additional button in the RTE toolbar. As with other buttons in the RTE, this should be enabled in the toolbar configuration together with a block configuration where you can choose the block types that should be allowed for this specific instance of the RTE. It could then look something like this:

![Editor Settings](assets/blocks-in-the-rte/Settings.png?raw=true)

#### Configuration of the block
For each block configured to work in the RTE, there should be additional configuration on the block level to choose block appearance, data models (both content- and settings models), and catalog appearance - just like we know it from the block list and the block grid.

For the first implementation of this feature, we propose not to include custom backoffice views, as it - compared to the expected usage - will be time-consuming to implement. This could be something to consider for later.

![Configuration of the block](assets/blocks-in-the-rte/Settings-block-config.png?raw=true)

#### Button in the RTE toolbar
When the Blocks button is enabled in the RTE toolbar configuration, we suggest a button like the one to the right in the mockup below will be available. 

![Button in the RTE toolbar](assets/blocks-in-the-rte/RTE.png?raw=true)

#### Insert block
When clicking on this button, the configured blocks for this instance of the RTE should be selectable in a way similar to the one known from the block list and the block grid.

![Insert block](assets/blocks-in-the-rte/Insert.png?raw=true)

#### Edit block
When inserting the block (or clicking on it later) we should show an edit panel to the right where both content and settings can be edited. 

![Edit block](assets/blocks-in-the-rte/Edit-content.png?raw=true)

#### Show the block in the RTE
When inserted, we propose to show the block as a bar/box similar to how we show blocks in the block list and the block grid when they have no custom view. Unlike the macros where you can choose to insert it as inline or not, the block will always be shown inline but can of course be controlled simply by adding paragraphs like shown below.

![Show the block in the RTE](assets/blocks-in-the-rte/Content-in-the-rte.png?raw=true)

## Technical considerations
We could extend the RTE in other ways than with blocks but believe that by building this on top of the block technology, we open up for benefiting from future block improvements while at the same time ensuring a more aligned developer- and editor experience. 
For the Content Delivery API, we propose that blocks should be listed in an array in the Json output and then referenced from within the RTE HTML element. While not used in a headless context, a partial view is needed, just like with blocks for the block list and the block grid. 

## Out of Scope
We hope to introduce global blocks (as described in the RFC for [Reusable Content with Global Blocks]) but that is not part of this RFC, although we believe that blocks in the RTE should also work with global blocks when these are implemented.

### Unresolved issues
The primary purpose of this RFC is to ensure that we with this - and other already existing functionality - have covered all use cases that are today handled by macros. Please let us know if you see use cases for macros that are still not solved.
Also, as described above, we propose blocks without custom views as a starting point but let us know if you see that as a problem.

## Contributors
This RFC was compiled by:

- Bjarke Berg (Umbraco HQ)
- Jacob Overgaard (Umbraco HQ)
- Filip Bech-Larsen (Umbraco HQ)
- Rune Strand (Umbraco HQ)
- Lasse Fredslund (Umbraco HQ)
- The CMS Group
- The CMS Community Team
