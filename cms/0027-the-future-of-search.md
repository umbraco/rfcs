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
## Detailed Design

We suggest to introduce a complete new search and indexing abstraction, which will allow for different search providers to be implemented. 
During our investigation, we have looked into Elasticsearch, Algolia, Meilisearch and Examine (Lucene.NET) as potential search providers.
To find a good balance between that the abstraction should be easy to use and that it should be possible to implement any search provider, we  suggest the following design.

### Persisting Index Data in the Database
We propose storing index data in the database, as calculating it can be highly CPU-intensive—particularly for documents with deeply nested types, such as block lists within block lists.
By persisting these data, they only need to be calculated once per change, even for search providers that require a search index per machine, such as Examine(Lucene.NET).

We suggest handling indexing within the search index via cache refreshers.
This approach allows different search implementations to determine which server roles should perform a task.
For all out-of-process search implementations, indexing would only be performed on servers with the `Publisher` role.

The primary benefit of persisting index data is significantly faster indexing during cold-boot scenarios where the entire index needs to be rebuilt.
For regular indexing of individual documents, we anticipate no noticeable difference.

The only drawback we foresee is an increase in database size by approximately 5-10%, due to this new data.

#### Persisted Index Data
The persisted index data will be stored in a new table, very similar to the published content, `umbracoIndexData`, with the following columns:

- `Key` (Guid, PK)
- `Published` (Bit)
- `DataRaw` (Binary) // Same binary serialization as the published content

The DataRaw column will contain the serialized index data for an entire document for all languages, which will be deserialized and sent for indexing by the search provider.

#### Property Index Value Factories
Like today, we will have to use a factory to generate the index data for each property editor.
We expect this property value factory to return a similar model for each property editor.

The index data per culture will include the following:
- `Texts` (string[]) // Analysed
- `TextsH1` (string[]) // Analysed but boosted like an H1 in html
- `TextsH2` (string[]) // Analysed but boosted like an H2 in html
- `TextsH3` (string[]) // Analysed but boosted like an H3 in html
- `TextsH4` (string[]) // Analysed but boosted like an H4 in html
- `TextsH5` (string[]) // Analysed but boosted like an H5 in html
- `TextsH6` (string[]) // Analysed but boosted like an H6 in html
- `Keywords` (string[]) // Used for exact matches
- `GeoPoints` (GeoPoint[])
- `Decimals` (Decimal[])
- `Integers` (int[])
- `Dates` (DateOnly[])
- `Times` (TimeOnly[])
- `DateTimesOffsets` (DateTimeOffset[])
- `Booleans` (string[]) // TODO Save the property alias?, og how can this work well for nested types
- `IsKeywordFacet`(bool)

Furthermore, we expect to use the same model for the native document fields, like Name, Creation Time, Update time, Creator.

### Search abstraction
TODO skal vi have 4 searchers: Documents, DraftDocuments, Media, Members
```cs
PagedSearchResult<IPublishedContent> Search(
    string query, // query to be analyzed, is this TODO: AND or OR between words?  //BOOST er ikke konfigurarbart, men skal det være det om det er AND or OR?
    string isoCode =null, // isoCode to filter by.
    ISet? facetKeys = null, // facets to be calculated // Should it be a sorted Set? // What about the order of the facets returned - Is that postprocessing?
    string[]? filters = null, // is this nessary, if we have range and facets? It is supported by the search providers. // TODO hvad med at !
    string[]? rangeFilters = null, // e.g price.numerics<200, __published.Dates>2022-01-01 // TODO explict model what about AND vs OR // TODO specific datatime vs int vs decimal..
    string[]? facetFilters = null, // e.g category:blogposts, color:black
    SortDirection sortDirection = SortDirection.Descending, // Sort direction
    string? sort = null // null means by relevance, otherwise it has to be a property type alias but what more? e.g. title.keywords //TODO can we do this without it becomes too tightly bound to the implementation
    int skip = 0, // Number of results to skip
    int take = 10 // Number of results to return
)
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
    public IPublishedContent Content { get; set; }
}

public class Facet
{
    public string Key { get; set; } // The property alias
    public IEnumerable<FacetValue> Values { get; set; }
}

public class RangeFacet
{
    public string Key { get; set; } // The property alias
    public T MinValue { get; set; }
    public T MaxValue { get; set; }
}

public class FacetValue 
{
    public string Value { get; set; } // The value from keyword
    public int Count { get; set; }
}
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

### Unresolved issues

## Contributors
This RFC was compiled by:

- Bjarke Berg (Umbraco HQ)
- Kenn Jacobsen (Umbraco HQ)
