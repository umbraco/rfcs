# The Future of Search

### Request for Contribution (RFC) 0027 : The Future of Search

**Please comment on this RFC in this [RFC Discussion](https://github.com/umbraco/rfcs/discussions/TODO)** // TODO

## Code of Conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md).

## Intended Audience

The intended audience for this RFC is

- Technical users
- Developers
- Extension developers
- Package authors

## Summary

TODO: polish

In this RFC we propose a replacement of the current search functionality in Umbraco.

The RFC outlines details for both indexing and searching as we envision it moving forward, and provides concrete examples of both.

## Terminology

- **Content**: Common denominator for Documents, Media and Members.
- **Search implementation**: An implementation of the search and indexing abstraction for a specific search provider.
- **Search provider**: The underlying search technology for a search implementation (e.g. Elasticsearch).
- **Field**: A container of data in the search index.

## Motivation

For many years, search functionality in Umbraco has relied on Examine, utilizing a standard implementation of Lucene.NET. However, Lucene.NET is known for its issues with hosting on network drives, making deployments on platforms like Azure Web Services particularly challenging. This has required extensive configuration adjustments and has been a frequent source of support cases for Umbraco.

Additionally, we recognize that many of our partners already use more advanced search providers. As a result, we aim to simplify integrations with these platforms, regardless of the specific technology used.

Finally, we see an opportunity to optimize the index rebuilding process, enabling it to run more efficiently and faster.

## Detailed design

We suggest to introduce a complete new search and indexing abstraction, which will allow for different search providers to be used for searching Umbraco content.

During our investigation, we have looked into Elasticsearch, Algolia, Meilisearch and Examine (Lucene.NET) as potential search providers.

We suggest the following design, based on the these criteria:

1. The abstraction should be easy to use.
2. It should be possible to implement with any search provider.

### Indexing

We suggest handling indexing for the search index via cache refreshers.

This approach allows different search implementations to determine which server roles should perform a task. For all out-of-process search implementations, indexing would only be performed on servers with the `Publisher` role.

The indexing will support both protected content (public access restrictions) and content variance (culture and segment).

The indexing will explicitly exclude sensitive data - e.g. property types marked as sensitive.

Umbraco will be responsible for gathering index data when content changes. This includes the gathering of index data for all relevant related content items - for example:

- Documents affected by the publishing of an ancestor document.
- Changes made to document protection (public access).

Subsequently, the index data will be handed off to the search implementation, which is then responsible for updating the search index accordingly.

#### Required indexes

We suggest creating four core indexes:

- **Documents** for end user document search.
    - Contains only published document data.
    - Replaces the current indexes "external index" and "Delivery API index".
- **Draft Documents** to power the backoffice document search.
    - Contains only draft document data.
    - Replaces the current "internal index".
- **Media** to power all media search.
- **Members** to power all member search.

Note that as of today, media are included in both the "external index" and the "internal index". We want to change this moving forward, as we perceive documents and media as two different things - particularly from a search perspective.

#### Custom indexes

It should be possible to define and maintain custom indexes from any Umbraco extension (for example Umbraco Forms).

#### Storing master data in indexes

We do _not_ propose storing master data (commonly known as "stored fields" or "raw fields") within the indexes.

An index is only meant to be an instrument for querying and resolving IDs for master data records. The actual master data records can subsequently be retrieved from their concrete storage.

#### Index data per property value

We will create new property index value factories to generate the property level index data for property editors. This replaces the `IPropertyIndexValueFactory` implementations in the current codebase. 

There will be different property index value factory implementations for different property editors, and they will return a common data format for indexing property data.

The index data format will include the following:

- `Texts` (`string[]`)
- `TextsH1` (`string[]`)
- `TextsH2` (`string[]`)
- `TextsH3` (`string[]`)
- `TextsH4` (`string[]`)
- `TextsH5` (`string[]`)
- `TextsH6` (`string[]`)
- `Keywords` (`string[]`)
- `Decimals` (`Decimal[]`)
- `Integers` (`int[]`)
- `DateTimeOffsets` (`DateTimeOffset[]`)

In this index data format:

- All `Texts` will be used for full text search. The granular division of text types are to be used for relevance boosting.
- `Keywords`, `Decimals`, `Integers` and `DateTimeOffsets` are meant for filtering and/or faceting.

We will _not_ provide specifics on how the index data should be indexed (e.g. how to analyze for full text search). We consider this an implementation detail for the individual search implementations, as it varies greatly between providers.

#### Index data per content item

All relevant properties of a content item will have their values indexed as per the previous section.

The property level index data will be passed to the index as individual fields by property type alias, thus allowing for filtering and faceting by property at query time.

Additionally, the following content level data will be included, using the same index data format:

- Name (as `TextsH1`)
- Creation date (as `DateTimeOffsets`)
- Update date (as `DateTimeOffsets`)
- Content type alias (as `Keywords`)
- ID (GUID) (as `Keywords`)
- Tags (as `Keywords`)
    - All tags accumulated across all property values.
- Ancestor IDs (GUIDs) (as `Keywords`)
    - Included to facilitate structural search queries.

#### Persisting index data in the database

We propose storing index data in the database, as calculating it can be highly CPU-intensive — particularly for content with deeply nested types, such as block lists within block lists.

By persisting these data, they only need to be calculated once per change, even for search implementations that require a search index per machine, such as Examine (Lucene.NET).

The primary benefit of persisting index data is significantly faster indexing during cold-boot scenarios where the entire index needs to be rebuilt.

For regular indexing of individual content items, we anticipate no noticeable difference.

The only drawback we foresee is an increase in database size by approximately 5-10%, due to this new data.

##### Persisted index data format

The persisted index data will be stored in a new table, similarly to how we store published content today.

The table will be named `umbracoIndexData` and contain the following columns:

- `Key`: GUID (primary key)
- `Published`: Bit
- `DataRaw`: Binary (same binary serialization as the published content)

The `DataRaw` column will contain the serialized index data for all variants of an entire content item.

#### Indexing protected documents

_This section applies only to indexing of published documents._

Protected documents are configured in Umbraco by specifying the concrete members and/or member groups that should have access to the documents.

When Umbraco hands off protected documents to be indexed by the search implementation, the configured member and member group IDs (GUIDs) will be passed to the search implementation. The search implementation must either:

- Index the documents accordingly, so they can be made available for the relevant members at query time, or
- Discard the documents, thus not indexing it at all (efficiently not supporting protected documents).

#### Indexing variants of content

For variant content, Umbraco will append the relevant variance qualifiers (culture and/or segment) to the index data at property level. The search implementation must either:

- Index the variance qualifiers, so variant content can be found at query time (given a concrete variant context), or
- Discard (parts of) the variance qualifiers and the corresponding property data (e.g. discard segmented property data if not supporting segmented querying).

#### Full re-indexing versus partial index updates

Some search providers support zero downtime for full re-indexing - for example by means of "index swapping".

Umbraco will be handing off either individual content items or collections of content for indexing, but for large sites it will eventually not be possible to hand off _all_ content in a single operation. However, we still want to support zero downtime re-indexing if at all possible.

This requires us to introduce an instruction protocol for communicating re-indexing start and end. The specifics are yet to be defined, but it will require some queueing mechanism to ensure atomic operations ("transactions") across multiple batches of content indexing.

#### Extensibility

We envision an extension model for indexing, which will allow for changing the default behavior (altering, adding and removing index data before it is sent to the search implementation).

Part of this extension model will be the property index value factories themselves. They will be interchangeable, so custom indexing can be performed for _all_ properties of a given property editor.

However, this will not cover all cases, so additional extension points will be called for. These are yet to be defined, but _could_ include:

- An "also index these items" option to include additional content items for re-indexing when changes are made to specific content.
- A "last minute" option to manipulate all index data for a single content item before it is passed to the search implementation for indexing.

#### Examples of index data

See Appendix A for a detailed example of index data being gathered for indexing.

### Searching

We suggest defining a search abstraction that covers:

- Full text search
- Filtering
- Faceting (including range facets)
- Autocomplete/suggestions
- Variance (as defined by Umbraco, by culture and/or segment)
- Protected content (as defined by Umbraco, by concrete principal or group membership)
- Sorting and pagination

Full text search, filtering and faceting can be used in conjunction with one another.

The goal is to cover the most common use cases, not to define an exhaustive search abstraction for all search providers. For concrete search provider features outside the scope of the search abstraction, implementors are expected to utilize the search provider directly instead of going through this abstraction.

#### Example of the search abstraction

See Appendix B for a code example of what the search abstraction signature _could_ look like.

#### Full text search

Full text search will be defined as:

- A query (`string`) containing one or more words to search for.
- An operator (`enum`) specifying if the words in the query should be matched as `OR` or `AND`. 

The search should be performed across the `Texts` from the property level index data.

Relevant boosting should be applied for the individual `Texts` data, e.g. ensuring higher search result relevance for data from `TextsH1` than `TextsH4` (where `Texts` is considered of the least relevance value).

We do _not_ plan to include detailed boosting levels as part of the search abstraction. We consider this an implementation detail for the individual search implementations. 

#### Filtering

Filtering allows for narrowing down the search results by exact matching against the following from the property level index data:

- `Keywords`: For exact matches on text.
- `Decimals`: For exact matches on decimal values.
- `Integers`: For exact matches on integer values.
- `DateTimeOffsets`: For exact matches on date and time.

The abstraction will support:
- Filtering against specific fields, thus narrowing down by content property value.
  - For example: The "color" property should have the value "Red".
- Chaining filters in a single query. Filters will be combined with an AND.
  - For example: ("color" must be "Red") AND ("price" must be less than 100).
- Providing multiple values per filter. Filter values will be combined with an OR.
    - For example: ("color" must be "Red" OR "Blue") AND ("size" must be "M" OR "L").

We propose splitting filtering into two parts:

- **Exact filtering** for concrete filter values.
  - For example: "color" is "Red" OR "Blue".
- **Range filtering** for range bound filter values.
  - For example: "price" is between 100 AND 200.

#### Faceting

From a search abstraction point of view, faceting works in much the same way as filtering; faceting can be performed against the same fields as filtering, and follows the same combination rules.

Faceting yields the same content results as filtering, but adds facet values to the search result.

From a search implementation perspective, faceting likely works very differently from filtering, and might incur a performance penalty over filtering.

We propose splitting faceting into two parts:

- **Exact faceting** for concrete facet values.
  - For example: "color" is "Red" OR "Blue".
- **Range faceting** for range bound facet values.
  - For example: "price" is between 100 AND 200.

#### Autocomplete/suggestions

We suggest including an optional "suggested next query" in the search results. This can be used by the search implementation to supply options for autocompletion or query suggestions for a given query.

The "suggested next query" will be a collection of strings, meant to replace or complement the active full text search query (if any).

#### Variance

Given a concrete variation context (culture and/or segment), the search implementation should be able to yield content results relevant to this context.

See the indexing section for more details.

#### Protected content

Given the context of a "current principal" (by means of a concrete principal identifier and/or group memberships), the search implementation should be able to include relevant protected content in search results.

See the indexing section for more details.

#### Search result

The search implementation should produce search results that contain:

- The total number of results.
- The IDs (GUIDs) of the content items within the current result page.
- Matching facets, if any have been requested.
- A collection of "suggested next queries", if any have been requested.

The search abstraction will resolve the actual content items for results generated by the search implementation - for example, retrieve matching published documents from the cache.

### Configuration

We have not identified any candidates for configuration of the search and indexing abstraction.

The individual search implementations may allow for search provider specific configuration, but this is considered an implementation detail for this RFC.

## Technical considerations

TODO: Any?

## Out of Scope

TODO: Polish?

We do only initially plan to ship a single implementation of the search abstraction, which will be based on Examine to be backward compatible.

We will not support other search providers in the initial release.

The following index values have been identified as potential candidates for future extension, but will not be included in the initial release: 

- `GeoPoints` (`GeoPoint[]`)
- `Dates` (`DateOnly[]`)
- `Times` (`TimeOnly[]`)
- `Booleans` (`bool[]`? `string[]`?)

## Contributors

This RFC was compiled by:

- Bjarke Berg (Umbraco HQ)
- Kenn Jacobsen (Umbraco HQ)

## Appendix A: Example of index data being gathered for indexing.

As mentioned, a property editor defines its own property index value factory for generating values for indexing. The following exemplifies how this will work.

In this example, we are indexing a document with:

- Name: Hitchhiker's Guide
- Document type alias: bookPage
- Created at: 2024-11-27 12:00:00
- Last updated at: 2024-11-27 14:30:00
- ID: 0e88ec21-dc56-4f8c-8247-40f72b2fd676
- Ancestor IDs: [33d41c8b-f541-4237-96ed-c5c85d4c3edd, 5ce303c9-8e0f-41cd-80d8-3f7906604e6a]

...and the following properties:

<table>
<tr>
<th>Alias</th>
<th>Type</th>
<th>Value</th>
</tr>
<tr>
<td>title</td>
<td>Umbraco.TextBox</td>
<td>The Hitchhiker's Guide to the Galaxy</td>
</tr>
<tr>
<td>featured</td>
<td>Umbraco.Toggle</td>
<td>true</td>
</tr>
<tr>
<td>pages</td>
<td>Umbraco.Integer</td>
<td>255</td>
</tr>
<tr>
<td>published</td>
<td>Umbraco.DateTime</td>
<td>1979-10-12</td>
</tr>
<tr>
<td>isbn13</td>
<td>Umbraco.TextBox</td>
<td>9780330258647</td>
</tr>
<tr>
<td>genre</td>
<td>Umbraco.Tags</td>
<td>[Science Fiction, Comedy]</td>
</tr>
<tr>
<td>subjects</td>
<td>Umbraco.MultipleTextstring</td>
<td>[Travel, Philosophy, Towels]</td>
</tr>
<tr>
<td>price</td>
<td>Umbraco.Decimal</td>
<td>119.95</td>
</tr>
<tr>
<td>score</td>
<td>Umbraco.Slider</td>
<td>[7.8, 9.3]</td>
</tr>
<tr>
<td>abstract</td>
<td>Umbraco.RichText</td>
<td>
<pre>
&lt;h1&gt;A brilliant read&lt;/h1&gt;
&lt;p&gt;This is the first book.&lt;/p&gt;
&lt;umb-rte-block (RTE block 1)/&gt;
&lt;h2&gt;Science Fiction&lt;/h2&gt;
&lt;p&gt;A trilogy of five books.&lt;/p&gt;
</pre>
<br/>
<b>RTE block 1 (element type alias: relatedBook)</b>
<table>
<tr>
<th>Alias</th>
<th>Type</th>
<th>Value</th>
</tr>
<tr>
<td>heading</td>
<td>Umbraco.TextBox</td>
<td>Fancy a bite?</td>
</tr>
<tr>
<td>text</td>
<td>Umbraco.TextArea</td>
<td>Check The Restaurant at the End of the Universe</td>
</tr>
<tr>
<td>tags</td>
<td>Umbraco.Tags</td>
<td>[Related, Sequel]</td>
</tr>
<tr>
<td>subjects</td>
<td>Umbraco.MultipleTextstring</td>
<td>[Food, Restaurant, Universe]</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>blocks</td>
<td>Umbraco.BlockList</td>
<td>
<b>Block 1 (element type alias: characterBlock)</b>
<table>
<tr>
<th>Alias</th>
<th>Type</th>
<th>Value</th>
</tr>
<tr>
<td>title</td>
<td>Umbraco.TextBox</td>
<td>Arthur Dent</td>
</tr>
<tr>
<td>text</td>
<td>Umbraco.RichText</td>
<td>
<pre>
&lt;h1&gt;Earthman and Englishman&lt;/h1&gt;
&lt;p&gt;Arthur is mostly harmless.&lt;/p&gt;
</pre>
</td>
</tr>
<tr>
<td>image</td>
<td>Umbraco.MediaPicker3</td>
<td>4414b7e4-7d41-45b2-9e41-d2085ed995bf</td>
</tr>
<tr>
<td>character</td>
<td>Umbraco.DropDown.Flexible</td>
<td>Main</td>
</tr>
</table>
<b>Block 2 (element type alias: teaserBlock)</b>
<table>
<tr>
<th>Alias</th>
<th>Type</th>
<th>Value</th>
</tr>
<tr>
<td>heading</td>
<td>Umbraco.TextBox</td>
<td>Like poems?</td>
</tr>
<tr>
<td>text</td>
<td>Umbraco.TextArea</td>
<td>Be careful with Vogon poetry!</td>
</tr>
</table>
</td>
</tr>
</table>

The resulting index data is listed in the table below.

Note that "system fields" (like document name and creation date) are included with a special "_umb_" prefix. The final prefix is yet to be decided, but there _will_ be a prefix to tell system fields apart from regular document properties.

<table>
<tr>
<th>Name</th>
<th>Value</th>
<th>Remarks</th>
</tr>
<tr>
<td>_umb_type</td>
<td>Keywords: ["bookPage"]</td>
<td>The document type alias is indexed as a keyword.</td>
</tr>
<tr>
<td>_umb_name</td>
<td>TextsH1: ["Hitchhiker's Guide"]</td>
<td>The document name is indexed as highest value text.</td>
</tr>
<tr>
<td>_umb_createDate</td>
<td>DateTimesOffsets: ["2024-11-27 12:00:00"]</td>
<td></td>
</tr>
<tr>
<td>_umb_updateDate</td>
<td>DateTimeOffsets: ["2024-11-27 14:30:00"]</td>
<td></td>
</tr>
<tr>
<td>_umb_tags</td>
<td>Keywords: ["Related", "Sequel", "Science Fiction", "Comedy"]</td>
<td>All tags on the page (including those in nested property editors) are indexed as individual keywords.</td>
</tr>
<tr>
<td>_umb_id</td>
<td>Keywords: ["0e88ec21-dc56-4f8c-8247-40f72b2fd676"]</td>
<td></td>
</tr>
<tr>
<td>_umb_ancestors</td>
<td>Keywords: ["33d41c8b-f541-4237-96ed-c5c85d4c3edd", "5ce303c9-8e0f-41cd-80d8-3f7906604e6a"]</td>
<td></td>
</tr>
<tr>
<td>title</td>
<td>Texts: ["The Hitchhiker's Guide to the Galaxy"]</td>
<td>Raw text inputs are indexed as lowest value text.</td>
</tr>
<tr>
<td>pages</td>
<td>Integers: [255]</td>
<td></td>
</tr>
<tr>
<td>published</td>
<td>DateTimeOffsets: ["1979-10-12"]</td>
<td></td>
</tr>
<tr>
<td>isbn13</td>
<td>Texts: ["9780330258647"]</td>
<td></td>
</tr>
<tr>
<td>genre</td>
<td>Keywords: ["Science Fiction", "Comedy"]</td>
<td>Tags are indexed as keywords.</td>
</tr>
<tr>
<td>subjects</td>
<td>Texts: ["Travel", "Philosophy", "Towels"]</td>
<td>Multiple textstrings are indexed as individual items (lowest value text).</td>
</tr>
<tr>
<td>price</td>
<td>Decimals: [119.95]</td>
</tr>
<td></td>
<tr>
<td>score</td>
<td>Decimals: [7.8, 9.3]</td>
<td></td>
</tr>
<tr>
<td>abstract</td>
<td>
<ul>
<li>TextsH1: ["A brilliant read"]</li>
<li>TextsH2: ["Science Fiction"]</li>
<li>Texts: ["This is the first book.", "A trilogy of five books.", "Fancy a bite?", "Check The Restaurant at the End of the Universe", "Food", "Restaurant", "Universe"]</li>
<li>Keywords: ["Related", "Sequel"]</li>
</ul>
</td>
<td>
<ul>
<li>The RTE takes headings into account, e.g. indexing H1 tags highest value texts.</li>
<li>Block properties are indexed as part of the RTE, adhering to their own indexing rules (i.e. tags become keywords).</li>
<li>Block document type aliases are not indexed.</li>
</ul>
</td>
</tr>
<tr>
<td>blocks</td>
<td>
<ul>
<li>TextsH1: ["Earthman and Englishman"]</li>
<li>Texts: ["Arthur Dent", "Arthur is mostly harmless.", "Like poems?", "Be careful with Vogon poetry!"]</li>
<li>Keywords: ["Main"]</li>
</ul>
</td>
<td>
<ul>
<li>The media picker is not indexed.</li>
<li>Fixed value selectors (dropdowns, radiobutton groups etc.) are indexed as keywords.</li>
</ul>
</td>
</tr>
</table>

The following table shows an example of how the default index data could be altered by means of extension points.

<table>
<tr>
<th>Name</th>
<th>Value</th>
<th>Changes from default</th>
</tr>
<tr>
<td>_umb_type</td>
<td>Keywords: ["bookPage"]</td>
<td></td>
</tr>
<tr>
<td>_umb_name</td>
<td>TextsH3: ["Hitchhiker's Guide"]</td>
<td>The document name has been given lesser text value (from TextsH1 to TextsH3).</td>
</tr>
<tr>
<td>_umb_createDate</td>
<td>DateTimesOffsets: ["2024-11-27 12:00:00"]</td>
<td></td>
</tr>
<tr>
<td>_umb_updateDate</td>
<td>DateTimeOffsets: ["2024-11-27 14:30:00"]</td>
<td></td>
</tr>
<tr>
<td>_umb_tags</td>
<td>Keywords: ["Related", "Sequel", "Science Fiction", "Comedy", "Robots"]</td>
<td>An extra tag ("Robots") has been added to the collection of page tags.</td>
</tr>
<tr>
<td>_umb_id</td>
<td>Keywords: ["0e88ec21-dc56-4f8c-8247-40f72b2fd676"]</td>
<td></td>
</tr>
<tr>
<td>_umb_ancestors</td>
<td>Keywords: ["33d41c8b-f541-4237-96ed-c5c85d4c3edd", "5ce303c9-8e0f-41cd-80d8-3f7906604e6a"]</td>
<td></td>
</tr>
<tr>
<td>title</td>
<td>TextsH4: ["The Hitchhiker's Guide to the Galaxy"]</td>
<td>The property value has been moved to a higher value text (from Texts to TextsH4).</td>
</tr>
<tr>
<td>featured</td>
<td>Keywords: ["featured"]</td>
<td>The toggle (boolean) is omitted by default, but has been included as a keyword here.</td>
</tr>
<tr>
<td>pages</td>
<td>Integers: [255]</td>
<td></td>
</tr>
<tr>
<td>published</td>
<td>DateTimeOffsets: ["1979-10-12"]</td>
<td></td>
</tr>
<tr>
<td>isbn13</td>
<td>Keywords: ["9780330258647", "0330258648"]</td>
<td>The ISBN 13 has been turned into a keyword, and the corresponding ISBN 10 has been added.</td>
</tr>
<tr>
<td>genre</td>
<td>Keywords: ["Science Fiction", "Comedy"]</td>
<td></td>
</tr>
<tr>
<td>subjects</td>
<td>Texts: ["Travel", "Philosophy", "Towels"]</td>
<td></td>
</tr>
<tr>
<td>price</td>
<td>Decimals: [119.95]</td>
<td></td>
</tr>
<tr>
<td>score</td>
<td>Decimals: [7.8, 9.3]</td>
<td></td>
</tr>
<tr>
<td>abstract</td>
<td>
<ul>
<li>TextsH1: ["A brilliant read", "Fancy a bite?"]</li>
<li>TextsH2: ["Science Fiction"]</li>
<li>Texts: ["This is the first book.", "A trilogy of five books.", "Check The Restaurant at the End of the Universe", "Food", "Restaurant", "Universe"]</li>
<li>Keywords: ["Related", "Sequel"]</li>
</ul>
</td>
<td>The "heading" property value from the block has been moved to higher value text (from Texts to TextsH1).</td>
</tr>
<tr>
<td>blocks</td>
<td>
<ul>
<li>TextsH1: ["Earthman and Englishman"]</li>
<li>TextsH4: ["Arthur Dent"]</li>
<li>Texts: ["Arthur is mostly harmless.", "Like poems?", "Be careful with Vogon poetry!", "Portrait of Author Dent in front of a spaceship"]</li>
<li>Keywords: ["Main"]</li>
</ul>
</td>
<td>
<ul>
<li>The "title" property value from the block has been moved to higher value text (from Texts to TextsH4).</li>
<li>The media picker ALT text (a property value from the picked media) has been added to Texts ("Portrait of...").</li>
</ul>
</td>
</tr>
</table>

## Appendix B: Code example for the search abstraction

The following is an example of what the search abstraction signature _could_ look like.

This is not necessarily how the final version will be implemented. It is meant as a supplement to illustrate the points made in this RFC.

```csharp
public PagedSearchResult Search(
    VariantFilter variantFilter,
    FullTextFilter? fullTextFilter = null,
    IEnumerable<Filter>? filters = null,
    IEnumerable<Facet>? facets = null,
    PrincipalFilter? principalFilter = null,
    Suggest? suggest = null,
    Sort? sort = null,
    int skip = 0,
    int take = 10
)
{
    // ...
}

public class VariantFilter
{
    // nulls means invariant
    public string? Culture { get; set; } // null = invariant
    
    // null means default (no) segment
    public string? Segment { get; set; }
}

public enum FullTextFilterQueryOperator
{
    Or,
    And
}

public class FullTextFilter 
{
    // query to be analyzed
    public required string Query { get; set; }

    // operator to use between the words in the query
    public FullTextFilterQueryOperator Operator { get; set; } = FullTextFilterQueryOperator.Or;
}

public static class SystemFields
{
    public const string Name = "_umb_name";
    public const string Id = "_umb_id";
    public const string CreateDate = "_umb_created";
    public const string UpdateDate = "_umb_updated";
    public const string ContentType = "_umb_type";
    public const string Ancestors = "_umb_ancestors";
}

public enum FilterOperator
{
    Is,
    IsNot
}

public abstract class Filter
{
    // the index field name (system field or property alias)
    public required string Field { get; set; }
}

public abstract class ExactFilter<T> : Filter
{
    // operator to use against the filter values
    public FilterOperator Operator { get; set; } = FilterOperator.Is;

    // the filter values (OR filtering applies if multiple values are specified)
    public required IEnumerable<T> Values { get; set; }
}

public class StringExactFilter : ExactFilter<string> {}

public class IntegerExactFilter : ExactFilter<int> {}

public class DecimalExactFilter : ExactFilter<decimal> {}

public class DateTimeOffsetExactFilter : ExactFilter<DateTimeOffset> {}

public class GuidExactFilter : ExactFilter<Guid> {}

public abstract class RangeFilter<T> : Filter
{
    // null means "no lower bounds"
    public T? MinValue { get; set; }

    // null means "no upper bounds"
    public T? MaxValue { get; set; }
}

public class IntegerRangeFilter : RangeFilter<int> {}

public class DecimalRangeFilter : RangeFilter<decimal> {}

public class DateTimeOffsetRangeFilter : RangeFilter<DateTimeOffset> {}

public abstract class Facet
{
    // the index field name (system field or property alias)
    public required string Field { get; set; }
}

public abstract class ExactFacet<T> : Facet
{
    // the facet values to filter by (OR filtering applies if multiple values are specified)
    public required IEnumerable<T> Values { get; set; }
}

public class StringExactFacet : ExactFacet<string> {}

public class IntegerExactFacet : ExactFacet<int> {}

public class DecimalExactFacet : ExactFacet<decimal> {}

public class DateTimeOffsetExactFacet : ExactFacet<DateTimeOffset> {}

public abstract class RangeFacet<T> : Facet
{
    // null means "no lower bounds"
    public T? MinValue { get; set; }

    // null means "no upper bounds"
    public T? MaxValue { get; set; }
}

public class IntegerRangeFacet : RangeFacet<int> {}

public class DecimalRangeFacet : RangeFacet<decimal> {}

public class DateTimeOffsetRangeFacet : RangeFacet<DateTimeOffset> {}

public class PrincipalFilter
{
    // the principal identifier
    public required Guid Id { get; set; }

    // the group memberships of the principal
    public required IEnumerable<string> Groups { get; set; }
}

public enum SortDirection
{
    Ascending = 0,
    Descending = 1,
}

public class Suggest
{
    // whether to include "suggested next queries" in the search result
    public bool SuggestNextQuery { get; set; } = false; 
}

public class Sort
{
    public SortDirection Direction { get; set; } = SortDirection.Descending;

    // the index field name (system field or property alias) - null means default (relevance)
    public string? Key { get; set; } = null; 
}

public class PagedSearchResult
{
    public int TotalCount { get; set; }

    public required IEnumerable<SearchResult> Results { get; set; }

    public IEnumerable<FacetResult>? FacetResults { get; set; }

    public IEnumerable<string>? SuggestedNextQuery { get; set; }
}

public class SearchResult
{
    public required Guid Id { get; set; }
}

public abstract class FacetResult
{
    // the index field name (system field or property alias) - null means default (relevance)
    public required string Field { get; set; }
}

public abstract class ExactFacetResult<T> : FacetResult
{
    // the available facet values for the query results
    public required IEnumerable<ExactFacetResultValue<T>> Values { get; set; }
}

public abstract class ExactFacetResultValue<T> 
{
    // the value in the index
    public required T Value { get; set; }

    // number of hits for this facet value
    public required int Count { get; set; }
}

public class StringExactFacetResult : ExactFacetResult<string> {}

public class IntegerExactFacetResult : ExactFacetResultValue<int> {}

public class DecimalExactFacetResult : ExactFacetResultValue<decimal> {}

public class DateTimeOffsetExactFacetResult : ExactFacetResultValue<DateTimeOffset> {}

public abstract class RangeFacetResult<T> : FacetResult 
{
    // the lower bounds in the index
    public T? MinValue { get; set; }

    // the upper bounds in the index
    public T? MaxValue { get; set; }
}

public class IntegerRangeFacetResult : RangeFacetResult<int> {}

public class DecimalRangeFacetResult : RangeFacetResult<decimal> {}

public class DateTimeOffsetRangeFacetResult : RangeFacetResult<DateTimeOffset> {}
```

