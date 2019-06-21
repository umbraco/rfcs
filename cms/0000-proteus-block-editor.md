# RFC Name

Request for Contribution (RFC) 0000 : Block-based data structure interoperability (Project Proteus)

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: technical users, developers & creators of packages.

## Summary

We would like to define a standard for storing structured data as a foundation for complex Umbraco editors, such as Grid-editor / DocType Grid Editor (DTGE) / LeBlender Editor, Nested / Stacked / Inner Content editors, etc.

This project has the codename [Proteus](https://en.wikipedia.org/wiki/Proteus), named after the Greek god whom was "capable to assume many forms"; and that is exactly what we want to achieve with this RFC.

**The short term goals are:**
- Standardise the way we utilise Umbraco elements
- Provide a standard for separating content from presentation data within complex editors
- Having a core framework for element manipulation to eventually implement packages like: Grid, Nested Content or other `PublishedElement` oriented packages (out-of-scope for this RFC)
- Make sure that all the content is structured, easy to manipulate, predictable and indexed properly.

**The long term goals are:**
- Make the element shareable, stored as elements in database.
- Use variants and segmentation at element level

## Motivation

Currently there are many complex editors, both in Core and as community packages, that store complex data structures in their own way. This makes the data more difficult to consume and re-use. Furthermore, design or implementation specific configuration is often mixed with the data, which restricts the flexibility and reusability of some editors (e.g. Grid).

The varying implementations result in equally varying developer experiences when utilising these editors. Having a common foundation for data storage means we can share common functionality across many parts of the CMS, and secure a more uniform developer experience.

The lack of consistency makes it challenging or impossible for data manipulation, including: 
- Indexing the information
- No way to reuse the data

## Detailed Design

We propose creating a uniform way to store a structured, single dimension, list of blocks.

Each block would be comprised of...

**Settings:** Configuration options to store use-case specific settings alongside the content (`key-value pair`)
  - Settings could be edited either through a UI or directly by concrete implementations of block-based editors in AngularJS code

**Content:** Element-type based content item (`IPublishedElement`)
  - Element Types are Document Types without any routable settings (e.g. templates, URL)
  - In the short term the element data will be stored as JSON. In the long term this JSON could be replaced by and UDI which could be a link to an IPublishedElement.

### Example of a list of blocks:

```
[
  {
    "settings": {
      "setting1": true,
      "setting2": "blue"
    }, 
    "content": {
      ...
    }
  },
  ...
]
```

By storing this data in a standardised way it would be possible to create several data types that share the same concept of storing data. As an added benefit, it becomes easy to swap a property from "Nested Content" to "Grid" (for example) and vice versa, as the underlying data is still the same.

The foundation should supply a means of customising the editor experience on a per-implementation basis. This could be in the form of inline previews, an inline editing experience, interactive map, or anything you can imagine!

In the future blocks could store a reference to a published document (`IPublishedElement`) as a `UDI`, rather than embedded JSON objects. This unlocks the possibility for variant blocks, segmentation and scheduled publishing blocks by leveraging similar behaviour already in the Core CMS. It also means multiple blocks could share the same underlying content item, and therefore make reusability of content far simpler across the CMS.

## Drawbacks

Compatibility between existing complex editors (Grid, Nested / Stacked / Inner Content, DTGE, etc) and the proposed model will unfortunately not be possible.

Due to this, block based implementations of existing complex editors might need to be maintained alongside the existing editors for a while. This could cause confusion for developers about which editors to be using.

An upgrade or migration path between specific editors could be defined at a later stage.

## Out of Scope

- Creating new implementations of existing complex editors as block-based editors - separate RFCs will be proposed to determine the exact approaches per editor
- Creating a mechanism to create and maintain PublishedElements as distinct documents in the database
- Extending behaviour of `IPublishedElement`, including permissions and scheduled publishing

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

- Is this the best data structure for storing blocks?
- Once the element is sharable, how do we manage the data manipulation (Edit, remove) between the different implemented block-based editors (e.g. Nested Content to Grid)
- How can we best handle validation while editing blocks content, given the many forms block editors can take (e.g. infinite editors, inline, etc)?
  - How do we handle required fields?
  - How do we handle custom field validation (e.g. via RegEx)?

## Contributors

This RFC was compiled by:

- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/aaantoinee) (community)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)
