# RFC Name

Request for Contribution (RFC) 0000 — Concept For New Rich text Editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is:

* Content creators, editors and other people who use the rich text editor on a daily basis
* Back Office developers who can help implement, improve and extend the rich text editor

## Summary

We would like to replace the current TinyMCE based editor with something simpler and faster, that focus on basic content creation.

We would like for the new editor to be able to support a faster workflow for content creators while knowing that they likely create their drafts outside the CMS and then copy their text into the editor.

We would like the new editor to automatically import photos that are dragged or copied into the editor as Media items without the editor needing to do anything.

We don't want the editor to be a page builder, grid or in other ways an editor for advanced landing pages, but we want to enable that the editor can be used as a part of a page builder.

## Motivation

Umbraco's mission is about _helping you deliver delightful digital experiences by making Umbraco friendly, simpler and social_.

We believe that the current solutions provided in Umbraco are either too basic or too advanced and there's room for a modern and focused Rich Text Editor that is faster and less bloated than the current TinyMCE based one and where the focus for the features should be centered around improving the workflow and speed of content creators.

The outcome of this project will be a new editor that's fast, supports elements that are considered basic part of modern content such as Photos, Quotes, Formatting, Embeds of both Video and Social. For content more advanced than that, we believe this project can be used as a part of block based editors rather than trying to solve that on its own.

## Detailed Design

Our requirements for a Rich Text Editor implementation.

We have the current prioritization for features:

1. Basic text formatting (bold, italic, unordered and ordered lists).

1. Ability change format of selected text/paragraph in one action.

1. Apply internal and external links.

1. Ability to create custom text formatting (supporting block and inline formatting).

1. Usage of a stylesheet to style text appearence.

1. Copy/paste or dragging images should create them as media items.

1. Ability to insert images from the media library.

1. Ability to insert links to files from the media library.

1. The stored data should contain nesecary information to transer (Media-element needs to have UDI of the media-item stored).

1. The data should be posible to output as presentable HTML, without begin aware that the data might was parsed to be able to present everything.

1. Copy/paste or drag styled text and media from another source, example Microsoft Word. Ensure an acceptable conversion to the format options available.

1. Style the editor UI to match Umbraco UI.

1. Documentation about configuration options of the RTE.

1. Ensure accessibility of the editor itself.

1. Ensure the output of the editor follows common accessibility practice.

1. Support for embedding videos from YouTube and Vimeo (and turn it into correct markup).

1. Support for embedding social media posts from Facebook and Twitter (and turn it into correct markup).

1. Support for extending embedding options.

1. Support for custom html snippets (such as inserting pre-styled Call To Action buttons).

1. Support for inserting Umbraco Macros.

1. Easy to use across the solution, since it could be used by different types of packages.

1. Documentation on implementation.

1. Documentation on how to write extentions.

1. Support for tables.


## Drawbacks

* TinyMCE cant be removed immidiatly, meaning that there would be two different types of Rich Text Editors available. This would cause confusion.

## Alternatives

Everything is open at the current state.

## Out of Scope

On idea level everything regarding this idea is in scope.

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

* Which Rich Text Editor should we use?
* Should the editor store its data as HTML or JSON?
* Which extension points should be available?
* How do we ensure good data, what can we do to avoid malformed HTML?
* Any important features we have missed?
* How can we minimize the problems of having two Rich text Editors at the same time?
* How should the Rich Text Editor be named?
* How should images dragged into the RTE be handled?

## Related RFCs 

* project L'Éditeur (https://github.com/umbraco/rfcs/pull/5)

## Contributors

This RFC was compiled by:

* Niels Lyngsø (HQ)
