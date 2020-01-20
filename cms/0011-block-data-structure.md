# RFC Name

Request for Contribution (RFC) 0011 : Block-based data structure interoperability (Project Proteus)

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
- Make the element shareable, stored as elements in database. (out-of-scope for this RFC)
- Use variants and segmentation at element level (out-of-scope for this RFC)

## Motivation

Currently there are many complex editors, both in Core and as community packages, that store complex data structures in their own way. This makes the data more difficult to consume and re-use. Furthermore, design or implementation specific configuration is often mixed with the data, which restricts the flexibility and reusability of some editors (e.g. Grid).

The varying implementations result in equally varying developer experiences when utilising these editors. Having a common foundation for data storage means we can share common functionality across many parts of the CMS, and secure a more uniform developer experience.

The lack of consistency makes it challenging or impossible for data manipulation, including: 
- Indexing the information
- No way to reuse the data

## Detailed Design

We propose creating a uniform way to store the data of a complex editor. The data will contain a single dimension list of blocks alongside the `layout` of these blocks. 

Each block would be comprised of...

**Content:** Element-type based content item (`IPublishedElement`)
  - Element Types are Document Types without any routable settings (e.g. templates, URL)
  - In the short term the element data will be stored as JSON and each element will have it's own UDI assigned

**Settings:** Configuration options to store use-case specific settings alongside the content (`IPublishedElement`)
  - Settings for each block is based on an Element type like the `Content` above which provides a consistent experience for both developers and editors to manage editing either `Content` or `Settings`



### Example of a list of blocks

```json
[
  {
    "settings": {
      "setting1": true,
      "setting2": "blue"
    }, 
    "content": {
      "udi": "umb://element/fffba547615b4e9ab4ab2a7674845bc9",
      "title": "Hello"
    }
  },
  {
    "settings": {
      "setting1": false,
      "setting2": "red"
    }, 
    "content": {
      "udi": "umb://element/e7dba547615b4e9ab4ab2a7674845bc9",
      "title": "World"
    }
  }
]
```

### Layout

Alongside the list of blocks stored, the editor will also store a Layout. The Layout will reference each content item in the list of blocks by it's UDI. The Layout object in the json will be key/value pairs where the `key` is the Property Editor alias. This is required because some editors will not store a one dimensional array layout structure such as Nested Content, other more complicated editors like the grid will store references to the blocks with it's own layout structure. By defining each layout with the alias of the Property Editor it means developers can in theory swap the underlying property editor without losing data and while keeping the layout preserved by the previous editor.

#### Example of full JSON

```json
{
  "layout": {
    "Umbraco.NestedContent": ["umb://element/fffba547615b4e9ab4ab2a7674845bc9", "umb://element/e7dba547615b4e9ab4ab2a7674845bc9"]
  },
  "blocks": [
    {
      "settings": {
        "setting1": true,
        "setting2": "blue"
      }, 
      "content": {
        "udi": "umb://element/fffba547615b4e9ab4ab2a7674845bc9",
        "title": "Hello"
      }
    },
    {
      "settings": {
        "setting1": false,
        "setting2": "red"
      }, 
      "content": {
        "udi": "umb://element/e7dba547615b4e9ab4ab2a7674845bc9",
        "title": "World"
      }
    }
  ]
}

```

### Strongly typed

Both content and settings are strongly typed as `IPublishedElement`. The layout can be strongly typed too but will be up to the editor to define. This means we can have a generic c# representation of this data like:

```cs

public interface IBlockData<TContent, TSettings, TLayout>
  where TContent: IPublishedElement
  where TSettings: IPublishedElement
{
    TContent Content { get; }
    TSettings Settings { get; }
}
```

Which also means these can be Models Builder strongly typed models too. 

### Future

In the future blocks could be stored in the database as a document rather than embedded JSON objects. This unlocks the possibility for variant blocks, segmentation and scheduled publishing blocks by leveraging similar behaviour already in the Core CMS. It also means multiple blocks could share the same underlying content item, and therefore make reusability of content far simpler across the CMS. (This part is out of the scope of this RFC).

## Drawbacks

Compatibility between existing complex editors (Grid, Nested / Stacked / Inner Content, DTGE, etc) and the proposed model will unfortunately not be possible.

Due to this, block based implementations of existing complex editors might need to be maintained alongside the existing editors for a while. This could cause confusion for developers about which editors to be using, however the old ones can be marked as deprecated so they no longer show up in the backoffice for new projects.

An upgrade or migration path between specific editors could be defined at a later stage.

## Out of Scope

- Creating new implementations of existing complex editors as block-based editors - separate RFCs will be proposed to determine the exact approaches per editor
- Creating a mechanism to create and maintain PublishedElements as distinct documents in the database (Ability to share block data between editors)
- Extending behaviour of `IPublishedElement`, including permissions and scheduled publishing
- Use variants and segmentation at element level (out-of-scope for this RFC)
- Validation of complex editors - We have the capability for this already but more prototypes and docs need to be written

## Contributors

This RFC was compiled by:

- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/aaantoinee) (community)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)
- [Shannon Deminick](https://twitter.com/shazwazza) (HQ)
