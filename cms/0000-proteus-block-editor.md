# RFC Name

Request for Contribution (RFC) 0000 : Proteus Block Editor

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: technical users, developers & creators of packages.

## Summary

We would like to find a global, structured and standard solution for storing structured data in complex Umbraco editors. That includes how the Grid-editor / DocType Grid Editor (DTGE) / LeBlender Editor, Nested / Stacked / Inner Content editors etc, must manipulate and store data.

This project has the codename [Proteus](https://en.wikipedia.org/wiki/Proteus), named after the Greek god whom was "capable to assume many forms"; and that is exactly what we want to achieve with this RFC.

**The short term goals are:**
- Standardizes the way we store Umbraco elements.
- Having a core framework for element manipulation to eventually implement packages like: Grid, Nested Content or other Element oriented packages (out-of-scope for this RFC).
- Make sure that all the content is structured, easy to manipulate, predictable and indexed properly.

**The long term goals are:**
- Make the element shareable, stored as elements in database.
- Use variants and segmentation at element level

## Motivation

At this moment there a several complex editors that store this complex data in it’s own way. This makes it less reusable. Furthermore, design or implementation specific configuration is often mixed with the data, which restricts the flexibility and reusability of some editors (e.g. Grid)

Grid, DTGE, Leblender, Nested / Stacked / Inner Content have no consistent way of storing structured data, which makes it challenging or impossible for data manipulation, including: 
- Indexing the information
- No way to reuse the data
- No consistency between the data model of these packages

## Detailed Design

We propose creating a uniform way to store a structured, single dimension, list of blocks.

Each block would be comprised of...

**Settings:** Configuration options to store use-case specific settings alongside the content (`key-value pair`)
  - Settings could be edited either through a UI or directly by concrete implementations of block-based editors in AngularJS code

**Content:** Element-type based content item (`IPublishedElement`)
  - Element Types are Document Types without any routable settings (e.g. templates, URL)
  - In the short term (phase 1), the element data will be stored as JSON. In the long term (phase 2) this JSON could be replaced by and UDI which could be a link to an IPublishedElement.

**Example of a list of blocks:**

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

By storing this data in a standardized way it would make it possible in the future to create several data types that share the same concept of storing data. In that way you could swap a property from “Nested Content” to “Grid” and vice versa. 

This will also make it possible to come up with your own data type, but use the same concept of storage.

In phase 2 the "content" element will simply be a reference to an `UDI`, which is a `PublishedElement` => `"content": "umb://document-type/..."` This unlocks the possibility for variant blocks, segmentation and scheduled publishing blocks in the future. It also means multiple blocks could share the same underlying content item, and therefore make reusability of content far simpler across the CMS.

## Drawbacks

Compatibility between existing complex editors (Grid, Nested / Stacked / Inner Content, DTGE, etc) and the proposed model will unfortunately not be possible.

Due to this, block based implementations of existing complex editors might need to be maintained alongside the existing editors for a while. This could cause confusion for users about which editors to be using.

An upgrade or migration path between specific editors could be defined at a later stage.

## Out of Scope

- Creating new implementations of existing complex editors as block-based editors - separate RFCs will be proposed to determine the exact approaches per editor
- Extending behaviour of `IPublishedElement`, including permissions and scheduled publishing

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

- Is this the best data structure for storing blocks?
- Once the element is sharable, how do we manage the data manipulation (Edit, remove) between the different implemented block-based editors (e.g. Nested Content to Grid)
- How can we best handle validation while editing blocks content, given the many forms block editors can take (e.g. infinite editors, inline, etc)?
  - How do we handle required fields?
  - How do we handle custom field validation (e.g. via RegEx)?

Additionally, with the proposed change of storing PublishedElements as documents in phase 2...
- When do we create / edit / delete element documents?
 - Should a document be created the first time a PublishedElement is added into a content item?
 - Should a element document be deleted when the last reference to it is deleted from a content item?
- What is the flow for an editor picking an existing element to reuse it elsewhere?
- How do we handle rollbacks & versioning when a reference to to a UDI is stored in a content item? What if that UDI no longer exists?

## Contributors

This RFC was compiled by:

- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/nathanwoulfe) (aaantoinee)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)
