# RFC Name

Request for Contribution (RFC) 0013 — Concept For New Rich text Editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is:

* Content creators, editors and other people who use the rich text editor on a daily basis
* Back Office developers who can help implement, improve and extend the rich text editor

## Summary

We would like to provide a Rich Text Editor experience that would enable content creators to work faster and simpler than our current implementation of TinyMCE 4.

The new implementation must be easy for developers to setup and configure on a data-type instance basis.

We would like the new editor to automatically import photos that are dragged or copied into the editor as Media items.

We want to focus on basic content creation, the editor should not be a page builder, grid or in other ways an editor for advanced landing pages, but we want to enable that the editor can be used as a part of a page builder.

The outcome of this project should be a new property-editor that's fast, supports elements that are considered basic part of modern content such as Photos, Headers, Quotes, Formatting, Embeds of both Video and Social Media Posts. For content more advanced than that, we believe this project can be used as a part of block based editors rather than trying to solve that on its own.

## Motivation

We believe that we should accommondate a better the editorial experience when working with rich text in Umbraco.

We want to make a simpler writting experience for Content Creators, the actions available in the UI should be minimized to a bare minimum of entry-actions, that can unfold several sub-actions. This should make the first impression simple, and provide a intuative selection of features that enable the editor to think less about the features presented.

Developers should be able to make configurations on a data-type instance level. Meaning that we should be able to have both an extensive configuration and a simple configuration in the same BackOffice.

Configuration should be simple, and configuring costum styles or embed options should be made posible in a simple way with the ability to go advanced if needed.


## Detailed Design

### Feature set

Our requirements for a Rich Text Editor implementation.

We have the current prioritization for features:

1. Basic text formatting (bold, italic, unordered and ordered lists).

2. Ability change format of selected text/paragraph in one action(not necessary one click).

3. Apply internal and external links.

4. Ability to create custom text formatting (supporting block and inline formatting, allowing formats to be combined by begin addiative to other formats).

5. Pasted or dropped images should be created as media items.

6. Ability to insert images from the media library.

7. Ability to insert links to files from the media library.

8. The stored data should contain relational information to ensure data can be transfered between enviroments (Media-element needs to have UDI of the media-item stored).

9. The copied data from the RTE should contain nesecary information to avoid uploading the same image again (if proven that it existis in the pasted enviroments).

10. Usage of a stylesheet to style text appearence.

11. The value of the property should be posible to output as presentable HTML, without the developer being aware that the data might was parsed to be able to present everything.

12. Copy/paste or drag styled text and media from another source, example Microsoft Word. Ensure an acceptable conversion to the format options available.

13. Style the editor UI to match Umbraco UI.

14. Documentation about configuration options of the RTE.

15. Ensure accessibility of the editor itself.

16. Ensure the output of the editor follows common accessibility practice.

17. Support for embedding videos from YouTube and Vimeo (and turn it into correct markup).

18. Support for embedding social media posts from Facebook and Twitter (and turn it into correct markup).

19. Support for extending embedding options.

20. Support for custom html snippets (such as inserting pre-styled Call To Action buttons).

21. Support for inserting Umbraco Macros.

22. Easy to use across the solution, since it could be used by different types of packages.

23. Documentation on implementation.

24. Documentation on how to write extentions.

25. Support for tables (if the choosen editor supports it)


### Use cases

We have defined a few use-cases that should be used when comparing different choices/implementations.

##### Technical article
Writting a techincal article would require the format options to underline the needs of presenting data in proper ways. Displaying diagrams with descriptions adjacent to it and present textual information in list formats is some of the key features for this use-case. Its also nesecary to make quotes with a clear visual representation of the source. The need for costum format options would be needed to display equations, recipies or code. Format options that can be additive to other format options can be nesecary, example the ability to superscript parts of a word keeping the format of the sentence (example a headline with a superscript character).

##### Blog post
Light content thats easy to read is offen done by gathering other medias than text. The ability to spicy up your content with Images, Video and Social Media Posts is a must. And it should be posible within a shift move, without having to work with code.

##### Reworking the content
Seeing your content in the right context offen gives new perspectives to the structure of your message. Editors would have to change and move the content around to adjust it to the layout of the website. This means that everything should have the ability to be moved and copied/pasted. A paragraph should be able to be split up and have a headline inserted or made from a line of the paragraph.


### Measurable improvements

To clarify what an improved implementation means for us, we have identified a list of things to be improved:

* Time it takes to get to the fully initialized state.
* Minimize both visual and interactional interuptions while initiliazing the editor.
* Simplify the configuration of custom format options, without compromissing the options to define wether a format should be a block or inline, addative or solo.
* Simplifyied experience when importing a new image.


## Drawbacks

* TinyMCE cant be removed immidiatly, meaning that there would be two different types of Rich Text Editors available. This could cause confusion.

## Alternatives

* Keep using TinyMCE, but investigate wether the implementation can be improved.
* Investigate wether TinyMCE can improve the experience if upgraded to version 5.
* Limit our feature set to be able to implement a even simpler RTE. To then use our energy on a Block Based Layout property-editor that enables compositions of Text, Media, Video, Social Media Posts and other block types.
* Is there another alternative that you think we should look into?

## Out of Scope

On idea level everything regarding this idea is in scope.

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

* Which Rich Text Editor should we use?
* Are there other use-cases we should accommodate?
* Any other improvements we should be meassuring?
* Should the editor store its data as HTML, JSON or another format?
* Which extension points should be available?
* How do we ensure good data, what can we do to avoid malformed HTML?
* Any important features we have missed?
* How can we minimize the confusion of having two Rich text Editors available?
* How should the Rich Text Editor be named?
* How should images dragged into the RTE be handled?

## Related RFCs

* project L'Éditeur (https://github.com/umbraco/rfcs/pull/5)

## Contributors

This RFC was compiled by:

* Niels Lyngsø (HQ)
