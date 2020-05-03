* https://www.elastic.co/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0
* https://stackoverflow.com/questions/43530610/how-to-do-a-mapping-of-array-of-strings-in-elasticsearch
* https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html
* 7.0 deprecated APIs that accept types, introduced new typeless APIs, and removed support for the _default_ mapping
  
* search
    * Elasticsearch looks in the _all field by default
    * by default, Elasticsearch returns documents matching any of the specified words
      (the default operator is OR )
    * all REST search requests use the _search REST endpoint and can be either a GET
      request or a POST request. You can search an entire cluster or you can limit the scope
      by specifying the names of indices or types in the request URL .
    * Basic components of a search request
        wymienic
    * SORT ORDER FOR RESULTS
        * default: _score
    * Understanding the structure of a response
    * Match query   
        * score
    * filter query
        * Rather than comput-
          ing the score for a particular term as queries do, a filter on a search returns a simple
          binary “does this document match this query” yes-or-no answer
        * Filters require less processing and are cacheable because they don’t calculate the score.
        * Behind the scenes, Elasticsearch constructs a bitset, which is a binary set of bits
          denoting whether the document matches this filter.
        * Use query clauses in query context for conditions which should affect the score of matching 
        documents (i.e. how well does the document match), and use all other query clauses in filter context
    * TERM QUERY AND TERM FILTER
        * the term being searched for isn’t analyzed, it must match a term in the docu-
          ment exactly for the result to be found
        * a term filter can be used when you want to limit the results to
          documents that contain the term but without affecting the score. 
        * "terms": { - the terms query (note the s !) can search for multiple terms
          in a document’s field
    * Match query
        * "match": {
        * the two most important behaviors are boolean and phrase
            * boolean
                * By default, the match query uses Boolean behavior and the OR operator.
                * "operator": "and"
            * phrase
    * https://www.elastic.co/guide/en/elasticsearch/reference/6.8/query-dsl-filtered-query.html
        * The filtered query is replaced by the bool query.
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html
    * keyword datatype
        * A field to index structured content such as IDs, email addresses, hostnames, status codes, zip codes or tags.
        * typically used for filtering (Find me all blog posts where status is published)
        * are only searchable by their exact value
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html