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

TODO...

## Motivation

For many years, search functionality in Umbraco has relied on Examine, utilizing a standard implementation of Lucene.NET. However, Lucene.NET is known for its issues with hosting on network drives, making deployments on platforms like Azure Web Services particularly challenging. This has required extensive configuration adjustments and has been a frequent source of support cases for Umbraco.

Additionally, we recognize that many of our partners already use more advanced search engines. As a result, we aim to simplify integrations with these platforms, regardless of the specific technology used.

Finally, we see an opportunity to optimize the index rebuilding process, enabling it to run more efficiently and faster.

## Detailed design

We suggest to introduce a complete new search and indexing abstraction, which will allow for different search engines to be used for searching Umbraco content.

During our investigation, we have looked into Elasticsearch, Algolia, Meilisearch and Examine (Lucene.NET) as potential search engines.

We suggest the following design, based on the these criteria:

1. The abstraction should be easy to use.
2. It should be possible to implement with any search engine.

### Terminology

- **Content**: Common denominator for Documents, Media and Members.
- **Search provider**: An implementation of the search and indexing abstraction for a specific search engine.
- **Field**: A container of data in the search index.

### Indexing

We suggest handling indexing for the search index via cache refreshers.

This approach allows different search providers to determine which server roles should perform a task. For all out-of-process search providers, indexing would only be performed on servers with the `Publisher` role.

The indexing will support both protected content (public access restrictions) and content variance (culture and segment).

The indexing will explicitly exclude sensitive data - e.g. property types marked as sensitive.

Umbraco will be responsible for gathering index data when content changes. This includes the gathering of index data for all relevant related content items - for example:

- Documents affected by the publishing of an ancestor Document.
- Changes made to Document protection (public access).

Subsequently, the index data will be handed off to the search provider, which is then responsible for updating the search index accordingly.

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

Note that as of today, media are included in both the "external index" and the "internal index". We want to change this moving forward, as we percieve documents and media as two different things - particularly from a search perspective.

TODO: Custom indexes - Umbraco Forms?

#### Index data per property value

Like today, we will use a property index value factory to generate the property level index data for property editors.

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

We will _not_ provide specifics on how index data should be indexed (e.g. specific analyzers for full text search). Instead, we will consider this an implementation detail for the individual search providers.

#### Index data per content item

All relevant properties of a content item will have their values indexed as per the previous section.

The property level index data will be passed to the index as individual fields by property type alias, thus allowing for filtering and faceting by property at query time.

Additionally, the following content level data will be included, using the same index data format:

- Name (as `TextsH1`)
- Creation date (as `DateTimeOffsets`)
- Update date (as `DateTimeOffsets`)
- Content type alias (as `Keywords`)
- ID (Guid) (as `Keywords`)
- Tags (as `Keywords`)
    - All tags accumulated across all property values.
- Ancestor IDs (Guids) (as `Keywords`)
    - Included to facilitate structural search queries.

#### Persisting index data in the database

We propose storing index data in the database, as calculating it can be highly CPU-intensive — particularly for content with deeply nested types, such as block lists within block lists.

By persisting these data, they only need to be calculated once per change, even for search providers that require a search index per machine, such as Examine (Lucene.NET).

The primary benefit of persisting index data is significantly faster indexing during cold-boot scenarios where the entire index needs to be rebuilt.

For regular indexing of individual content items, we anticipate no noticeable difference.

The only drawback we foresee is an increase in database size by approximately 5-10%, due to this new data.

##### Persisted index data format

The persisted index data will be stored in a new table, similarly to how we store published content today.

The table will be named `umbracoIndexData` and contain the following columns:

- `Key`: Guid (primary key)
- `Published`: Bit
- `DataRaw`: Binary (same binary serialization as the published content)

The `DataRaw` column will contain the serialized index data for all variants of an entire content item.

#### Indexing protected content

TODO: This can ONLY be for members - Users will use the indexed ancestor IDs for scoping (user start nodes)

When configuring protected content in Umbraco, one specifies the concrete members and/or member groups that should have access to the content.

When Umbraco hands off protected content to be indexed by the search provider, the configured member and member group IDs (GUIDs) will be passed to the search provider. The search provider must either:

- Index the content accordingly, so it can be made available for the relevant members at query time, or
- Discard the content, thus not indexing it at all (efficiently not supporting protected content).

#### Indexing variants of content

For variant content, Umbraco will append the relevant variance qualifiers (culture and/or segment) to the index data at property level. The search provider must either:

- Index the variance qualifiers, so variant content can be found at query time (given a concrete variant context), or
- Discard (parts of) the variance qualifiers and the corresponding property data (e.g. discard segmented property data if not supporting segmented querying).

#### Full re-indexing versus partial index updates

Some search engines support zero downtime for full re-indexing, commonly known as index swapping.

Umbraco will be handing off either individual content items or collections of content for indexing, but for large sites it will eventually not be possible to hand off _all_ content. However, we still want to support index swapping if at all possible when performing full re-indexing.

This requires us to introduce an instruction protocol for communicating re-indexing start and end. The specifics are yet to be defined, but it will require some queueing mechanism to ensure atomic operations ("transactions") across mutiple batches of content.

#### Extensibility

We envision an extension model for indexing, which will allow for changing the default behavior (altering, adding and removing index data before it is sent to the search provider).

Part of this extension model will be the property index value factories themselves. They will be interchangeable, so custom indexing can be performed for _all_ properties of a given property editor.

However, this will not cover all cases, so additional extension points will be called for. These are yet to be defined, but _could_ include:

- An "also index these items" option to include additional content items for re-indexing when changes are made to specific content.
- A "last minute" option to manipulate all index data for a single content item before it is passed to the search provider for indexing.

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

The goal is to cover the most common use cases, not to define an exhaustive search abstraction for all search engines. For concrete search engine features outside the scope of the search abstraction, implementors are expected to utilize the search engine directly instead of going through this abstraction.

TODO: appendix B

#### Full text search

Full text search will be defined as:

- A query (`string`) containing one or more words to search for.
- An operator (`enum`) specifying if the words in the query should be matched as `OR` or `AND`. 

The search should be performed across the `Texts` from the property level index data.

Relevant boosting should be applied for the individual `Texts` data, e.g. ensuring higher search result relevance for data from `TextsH1` than `TextsH4` (where `Texts` is considered of least relevance value).

We do _not_ plan to include detailed boosting levels as part of the search abstraction. We consider this an implementation detail for the individual search providers. 

#### Filtering

Filtering allows for narrowing down the search results by exact matching against the following from the property level index data:

- `Keywords`: For exact matches on text.
- `Decimals`: For exact matches on decimal values.
- `Integers`: For exact matches on integer values.
- `DateTimeOffsets`: For exact matches on date and time.

The abstraction will support:
- Filtering against specific fields, thus narrowing down by content property value.
  - For example: The "color" property should have the value "Red".
- Providing multiple filters in a single query. Filters will be combined with an AND.
  - For example: ("color" must be "Red") AND ("price" must be less than 100).
- Providing multiple values per filter. Filter values will be combined with an OR.
    - For example: ("color" must be "Red" OR "Blue") AND ("size" must be "M" OR "L").

TODO: Ranged filtering can be obtained by chaining filters? Introduce range filters?

#### Faceting

From a search abstraction point of view, faceting works in much the same way as filtering; faceting can be performed against the same fields as filtering, and follows the same combination rules.

Faceting yields the same content results as filtering, but adds facet values to the search result.

From a search provider perspective, faceting likely works very differently from filtering, and might incur a performance penalty over filtering.

We propose splitting faceting into two parts:

- **Facet filtering** for concrete facet values.
  - For example: "color" is "Red" OR "Blue".
- **Range filtering** for range bound facet values.
  - For example: "price" is between 100 AND 200.

#### Autocomplete/suggestions

We suggest including an optional "suggested next query" in the search results. This can be used by the search provider to supply options for autocompletion or query suggestions for a given query.

The "suggested next query" will be a collection of strings, meant to replace or complement the active full text search query (if any).

#### Variance

Given a concrete variation context (culture and/or segment), the search provider should be able to yield content results relevant to this context.

See the indexing section for more details.

#### Protected content

Given the context of a "current principal" (by means of a concrete principal identifier and/or group memberships), the search provider should be able to include relevant protected content in search results.

See the indexing section for more details.




TODO skal vi have 4 searchers: Documents, DraftDocuments, Media, Members
```cs

PagedSearchResult Search(
    VariantFilter variantFilter,
    FullTextFilter? fullTextFilter = null,
    IEnumerable<Filter>? filters = null, // this is necessary to perform filtering without facets - or facets within a filtered scope
    IEnumerable<FacetFilter>? facetFilters = null, // e.g category:blogposts, color:black
    IEnumerable<RangeFilter>? rangeFilters = null, // e.g price.numerics<200, __published.Dates>2022-01-01 // TODO explict model what about AND vs OR // TODO specific datatime vs int vs decimal..
    PrincipalFilter? principalFilter = null, // expands the gross scope of the search to include all content allowed for this principal (explicitly by ID or implicitly by group membership)
    SortDefinition sortDefinition = SortDirection.Descending,
    int skip = 0, // Number of results to skip
    int take = 10 // Number of results to return
)

public class VariantFilter
{
    public string? Culture { get; set; } // null = invariant
    
    public string? Segment { get; set; } // null = default segment (language)
}

public enum FullTextFilterQueryOperator
{
    Or,
    And
}

public class FullTextFilter 
{
    public string Query { get; set; } // query to be analyzed, using OR between words

    public FullTextFilterQueryOperator Operator { get; set; } = FullTextFilterQueryOperator.Or;
}
    
public class SortDefinition
{
    public SortDirection Direction { get; set; } = SortDirection.Descending;

    // TODO: key? field? consistency!
    public string? Key { get; set; } = null; // null means by relevance, otherwise use one of SystemFields or a relevant proprety alias 
}

public static class SystemFields
{
    public const string Name = "_umb_name";
    public const string CreateDate = "_umb_created";
    public const string UpdateDate = "_umb_updated";
    public const string ContentType = "_umb_type";
}

public enum FilterOperator
{
    Is,
    IsNot,
    LessThan,
    LessThanOrEqual,
    GreaterThan,
    GreaterThanOrEqual,
}

public abstract class Filter 
{
    public string Key { get; set; } // The property alias

    public FilterOperator Operator { get; set; } = FilterOperator.IS;
}

public abstract class Filter<T> : Filter
{
    public IEnumerable<T> Values { get; set; } // OR
}

public class StringFilter : Filter<string> {}

public class IntegerFilter : Filter<int> {}

public class DecimalFilter : Filter<decimal> {}

public class DateTimeOffsetFilter : Filter<DateTimeOffset> {}

public abstract class FacetFilter 
{
    public string Key { get; set; } // The property alias
}

public abstract class FacetFilter<T> : FacetFilter
{
    public IEnumerable<T> Values { get; set; } // OR
}

public class StringFacetFilter : FacetFilter<string> {}

public class IntegerFacetFilter : FacetFilter<int> {}

public class DecimalFacetFilter : FacetFilter<decimal> {}

public class DateTimeOffsetFacetFilter : FacetFilter<DateTimeOffset> {}

public abstract class RangeFilter
{
    public string Key { get; set; } // The property alias
}

public abstract class RangeFilter<T> : RangeFilter
{
    public T MinValue { get; set; }
    public T MaxValue { get; set; }
}

public class IntegerRangeFilter : RangeFilter<int> {}

public class DecimalRangeFilter : RangeFilter<decimal> {}

public class DateTimeOffsetRangeFilter : RangeFilter<DateTimeOffset> {}

public class PrincipalFilter
{
    // TODO: Id? Key?
    public Guid Id { get; set; }

    public IEnumerable<string> Groups { get; set; }
}

// Filters:
// Algolia has a dedicated model for filters, facetfilers and numericFilers: https://www.algolia.com/doc/guides/managing-results/refine-results/filtering/in-depth/filters-and-facetfilters/
// ElasticSearch has dedicated model for filters, facetfilers and numericFilers https://medium.com/hepsiburadatech/how-to-create-faceted-filtered-search-with-elasticsearch-75e2fc9a1ae3
// Meilisearch, has a just string filters https://www.meilisearch.com/docs/reference/api/search

// TODO abstraction kan godt håndtere at opløse en key til IPublishedContent
public class PagedSearchResult
{
    public int TotalCount { get; set; }
    public IEnumerable<SearchResult> Results { get; set; }
    public IEnumerable<Facet> Facets { get; set; }
    public IEnumerable<RangeFacet> RangeFacets { get; set; }
    public IEnumerable<string> SuggestedNextQuery { get; set; }
}

public SearchResult
{
    // TODO: Key? Id?
    public Guid Id { get; set; }
}

public class Facet
{
    public string Key { get; set; } // The property alias
    public IEnumerable<FacetValue> Values { get; set; }
}

public class FacetValue 
{
    public string Value { get; set; } // The value from keyword
    public int Count { get; set; }
}

public abstract class RangeFacet
{
    public string Key { get; set; } // The property alias
}

public abstract class RangeFacet<T> : RangeFacet
{
    public T MinValue { get; set; }
    public T MaxValue { get; set; }
}

public class IntegerRangeFacet : RangeFacet<int> {}

public class DecimalRangeFacet : RangeFacet<decimal> {}

public class DateTimeOffsetRangeFacet : RangeFacet<DateTimeOffset> {}

```
### Configuration
The following configuration is planned to be in the abstraction. Any further configurations will most likely be search provider specific.

TODO: Any?

### Multiple indexes
We plan to have a separate index for website search (published data) and backoffice search (draft data). This is to ensure that the website search is not affected by the backoffice search, and vice versa.


TODO Hvad med media og members indexes, kun relevant for backoffice.
## Technical considerations

### Protected documents

### Analyzed data types
- Strings
### Specialized Data Types
- GeoPoint
- Numeric
- DateTimes
- Boolean
- String keywords
- Stored Fields // Data you wanna have returned in the search result. Not intented for rebuilding an IPublishedContent.
### Auto complete / Suggestions

### Facets
Facets needs to be handled on the specific search provider implementations, as they are not supported by all search providers.
We expect the abstraction to support facets, due to the data that is sent to each search provider for indexing has specified data types for analyzed vs non-analysed is data.
- Only supported by some implementations, but can be handled by the keywords field.

### Ranges
- E.g. time range or numeric range
### Analyzing
Due to the fact that analyzing needs vary greatly between different search providers, we propose this to happen in the implementations. E.g. most structured search providers needs to have the data analyzed before it is indexed, while AI based search providers not need this.

The abstraction will indicate what data is expected to be analysed and what is expected to be stored as is.




##### Examples of published content and index data

TODO..

TODO - What about protected content

[//]: # (We could extend the RTE in other ways than with blocks but believe that by building this on top of the block technology, we open up for benefiting from future block improvements while at the same time ensuring a more aligned developer- and editor experience. )

[//]: # (For the Content Delivery API, we propose that blocks should be listed in an array in the Json output and then referenced from within the RTE HTML element. While not used in a headless context, a partial view is needed, just like with blocks for the block list and the block grid. )

## Out of Scope
We do only initially plan to ship a single implementation of the search abstraction, which will be based on Examine to be backward compatible.
We will not support other search providers in the initial release.

TODO: out of scope - index values at property level
- `GeoPoints` (GeoPoint[])
- `Dates` (DateOnly[])
- `Times` (TimeOnly[])
- `Booleans` (string[]) // TODO Save the property alias?, og how can this work well for nested types
- `IsKeywordFacet`(bool) TODO: implementation specific configuration???

### Unresolved issues

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
</td>
</li>
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
</td>
</li>
<td>
<ul>
<li>The "title" property value from the block has been moved to higher value text (from Texts to TextsH4).</li>
<li>The media picker ALT text (a property value from the picked media) has been added to Texts ("Portrait of...").</li>
</ul>
</td>
</tr>
</table>


