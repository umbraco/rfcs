# Element Data storage

Request for Contribution (RFC) 0024 : Storing _Element Data_ as separate nodes.

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is technical users, developers and package creators.

## Summary

The purpose of this RFC is to introduce a way to store data based on _Element Types_ as separate data in the database, instead of storing them as a single blog of JSON on an existing node.

The storage of data created from _Element Types_ will be defined as _Element Data_ in this RFC. For the sake of focus, the name of _Element Data_ is _Out of Scope_ for this RFC.

Once this RFC has been implemented, the Editor Experience and Developer Experience should appear the same (Element Data will look and behave the same in both the UI and in Razor Views).

The goal is for this work is to _enable_, but _not include_

1. Element Data reuse
2. Pickers (UI) for selecting elements
3. Tooling for updating/sync'ing variants using element data

## Motivation

The current design of how Element Data is stored has a number of less than optimal side-effects:

* Element Data is stored as "dumb" data, meaning Umbraco only knows about the data when it has been de-serialized
* This means that it's hard to ensure data integrity
* The consequence of this is that it's not possible to treat Element Data as semantic data
* Thus there's a great deal of workarounds in anything from UI to storage to views to "understand" the Element Data
* This makes it hard to enable things like comparing Element Data in variants of a node, re-using Element Data, ensuring relations between element data and much more

## Detailed Design

Currently Element Data is stored as a JSON blob in a property on a node, instead of being stored as individual nodes with no URL. 

Say I have a Document Type "Article" with a Property "Content" of Property Type "Nested Content". The "Content" property will use a number of Element Types to allow data to be created by the "Nested Content" Property Type. 

In this simplified example, Nested Content will be used as a simple block editor allowing the editor to create a number of rows with different layout:

1. Headline
2. Image + Text
3. Text + Quote
4. Full Image

When the editor then create a new _Node_ of type _Article_ and edit the _Content_ with a "Headline", a "Full Image", a "Text + Quote" and an "Image + Text", all that data is then serialized as one blob of JSON and stored on the _Article_ node in the _Content_ property. This means that to ensure any type of data integrity (understanding what that data is), the data needs to be de-serialized every single time:

* Every time the Article is loaded in the back office

* Every time the Article is used in a razor view

* Every time a deploy tool like "Umbraco Deploy" or "uSync" needs to find related nodes or images

  

#### An example of why missing data integrity is bad

Let's say the editor wants to create a variant of the Article and update the text to German, but leave the images used in "Full Image" and "Image + Text" unchanged. This works fine as creating new variant simply copies all the data from the English version to the German version.

Later, the images needs to be updated. So the editor simply updates them on the English version and publish both the English and German version. Because there's no relationship between the two versions, the images are only updated on the English version.

In other words, the variant features available on properties on a node is not available on Element Data.



### Possible solution   

All _Element Data_ should be stored as real nodes with no URL. This means that they should be _referenced_ by their UDI in a Property on a Node, instead of being _stored_ in a Property on a Node.

For the Editor and for the Developer, this change won't be noticed compared to what we have today as Element Data won't be visible in the tree in the back office and in the Razor views, the Element Data will appear as a collection of IPublishedElement on a property of an IPublishedElement as it does today.



## Drawbacks

There _might_ be initial performance implications from storing more data as nodes especially for larger installations. However, the worry of this shouldn't prevent us from moving forward. We should therefore do performance comparisons during our work and it should be a goal that this change will have a _positive performance_ impact.

## Alternatives

Accept what's already there and let it up to the individual developer to come up with workarounds

## Out of Scope

1. The name of _Element Data_.
2. This is purely about how the element data is stored, thus giving us a more solid foundation for working with element data in the future. The work of this RFC _could enable_, but _doesn't_ include
   1. Element Data reuse
   2. Pickers (UI) for selecting elements
   3. Tooling for updating/sync'ing variants using element data

## Unresolved Issues

The answers that we are hoping to get from the community HQ is:

* What would be the best way to store the data (as nodes, in new tables, in ...?)
* How can be best ensure that this is a change with full backwards compatibility

## Related RFCs (where are we in roadmap?)

This work builds upon [RFC 0011 - Project Proteus: Block-based data structure interoperability](https://github.com/umbraco/rfcs/pull/11)

## Contributors

This RFC was compiled by:

* Niels Hartvig
