# RFC Name

Request for Contribution (RFC) 0016 : TinyMCE RTE Improvements

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is:
* Developers, implementors & editors

## Summary

This RFC is to give specific details on how we plan to improve the existing implementation of the TinyMCE version 4 rich text editor.

## Motivation

This has been a follow up RFC from #5 & #15. It has been discusssed that the rich text editor could be improved and enhanced to help give content editors a better experience when authoring or ammending content in the CMS.

The motivation from #15 isn't really much different from this one: We would like to provide a Rich Text Editor experience that would enable content creators to work faster and simpler than our current implementation. We want the rich text editor to be fully configurable at the data type level instead of relying on a global configuration file like it currently does. The rich text editor should handle drag and drop and copy and paste of images nicely along with copy/pasting of Word documents. Common elements should be supported out of the box such as headers, quotes without having to add custom styles to a stylesheet. We also want to improve the current OEmbed functionality to include things like social media posts.

## Detailed Design

The Rich Text Editor improvements will be based on the current rich text editor library that the Umbraco CMS uses which will be the latest version of TinyMCE 4.x.

The following improvements to the TinyMCE editor are planned and in another round we can do further iterative improvements.

**Configuration per data type**

Currently there is a global tinymceconfig.config file which affects every instance of TinyMCE in the backoffice which isn't ideal. We will allow for total configuration of the rich text editor via the data type configuration. There will be some prototyping taking place on how to best present all of these configuration options to the developer since there's quite an overwhelming amount of configuration that can be done. There is an option to keep the current tinymceconfig.config file to be used as the default configuration values which get copied to new data types when they're created.

**Implement pasting along with drag n drop of images**

We have verified with a CodePen sample to ensure we could get TinyMCE to support this. We will allow a prevalue configuration of the editor in Umbraco to set a folder location of where to store pasted and dragged images in the media library. A future iteration could improve this further and prompt the user on where they would like to store the image/s in the media library.

**Improve the HTML for images**

Currently when an image is inserted by the media library a `style` attribute is applied to the image element for sizing but the native `width` and `height` element attributes should be used instead. The width/height value is also a static value determined by the data type which is probably not ideal in many circumstances. We will investigate the options on how to improve this experience. This could include anything from allowing the editor to enter a width/height, trying to auto-detect these values, looking into using the html img element's `srcset` and `sizes` attributes or even look at the experimental [intrinsicsize](https://googlechrome.github.io/samples/intrinsic-size/index.html) attribute. We also need to be aware that the images inserted into the RTE use query strings on the image source to ensure a resized image is returned. To prevent a huge hi-res image from being used, however some editors do want the hi-res image source to be used. This could be a case where we have more options for the editor to configure the image when inserting.

**Reduce inital load time and file size of TinyMCE into the browser**

We will prototype various ways to improve the perceived initial load time of TinyMCE. We will be looking at minification and compression techniques along with loading some of it's assets in behind the scenes before the editor is rendered.

**Improve the loading experience** 

Use Tiny's `init_inistance_callback` property to remove Umbraco's own loader as opposed to the current implementation of checking it the TinyMCE library is loaded.

**Improve the experience with pasting from Word documents**

Use Tiny's event `before_paste` to notify us when content is being pasted into the editor. This event already notifies us if the content was copied from Word. We can use the pasted content to some JavaScript open source library or C# library to help clean up the HTML markup & formatting and thus remove any flicker associated with the editor loading.

We would like to allow developers to enhance this themselves by replacing this functionality if they choose too with already existing plugins such as the paid plugin by [TinyMCE PowerPaste](https://apps.tiny.cloud/products/powerpaste/)

**Improve moving un-editable elements**

For non-editable elements inserted into the rich text editor (such as macros, or OEmbed content) we want to allow for moving these elements in a nicer way by allowing copy/pasting or moving of these elements.

**Improve common and custom styles/formatting**

By default the toolbar will have common styles and formatting options such as headers, quotes and code references (blocks and inline). Creating these custom buttons/styles in TinyMCE is trivial and we will investigate how these things could be extended and configured in the datatype editor if developers wanted to add their own buttons. A great example of how this is possible today can be found [on this blog post](https://www.justnik.me/blog/umbraco-rte-styles-another-way). For the improved editor, we want to allow configuration of these things at the datatype level with a bit of UI jazz to make customization nicer than editing JSON text. 

## Drawbacks

There should be no drawbacks to this, as this project can be iterative and add improvements and enhancements to the existing editor that we all currently use.

## Alternatives

We did consider Trix from the previous user testing results from content editors that we recieved, however after comparing speed, extensibility, documentation of the editors. We found that it would be more beneficial to improve and enhance what we already have as opposed to re-implementing everything again in Trix and having to maintain two editors side by side. In part because removing TinyMCE would be considered a breaking change. Below are notes on some of the findings & comparissons.

**Cleaner HTML output** <br/>

Trix was proposed it had a cleaner HTML output, however from testing the editor side by side with our TinyMCE implementation when using the `Enter` key, Trix would generate markup as a `<div>` as opposed to TinyMCE's more semantically correct `<p>` paragraph.

**Loading and Rendering**<br/>
Trix is a more leightweight library and is able to load very quickly, however when considering the functionality and additions that the TinyMCE editor can currently do. It is an unfair comparisson.

We are aware that TinyMCE takes longer to do its initial load into the backoffice for the first request due to the size of the library. Subsequent requests - regardless if TinyMCE uses an IFrame or not - are fractionaly slower when rendering the editor and its contents than Trix. As mentioned above we aim to reduce the initial file size to help with the first render of a TinyMCE editor.

One thing from our testing we noticed that the current implementation of TinyMCE when testing with a slow/throttled network connection will load several times. This is due to an implemntation flaw where we currently check that the TinyMCE library is loaded. Due to the way TinyMCE has been developed checking the presence of the main library is loaded is not the best way to know if an editor is ready. We plan to fix this as part of our improvements as TinyMCE gives us a callback function to notify us when it has fully loaded and initilased the editor to help the jarring experience of several loading phases.


## Out of Scope

**Upgrading to TinyMCE 5** <br/>

After some initital research & findings, we found that it would cause breaking changes for any Umbraco sites who have written their own TinyMCE plugins/buttons. We also found that upgrading to TinyMCE version 5 would give us little benefit, apart from the underlying APIs that it uses has been made neater.


## Unresolved Issues

* Finding an OpenSource library to help cleanup pasted Word HTML, either JavaScript or C# that does the best job


## Related RFCs

<!-- vale valeStyle.ListStart = NO -->

* https://github.com/umbraco/rfcs/pull/15#issuecomment-514550647
* https://github.com/umbraco/rfcs/pull/5

<!-- vale valeStyle.ListStart = YES -->

## Contributors

This RFC was compiled by:

* Warren Buckley (Umbraco HQ)
