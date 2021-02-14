[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

# elasticsearch7-query-filter-aggregation-workshop

* references
   * https://www.manning.com/books/elasticsearch-in-action
   * https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
   * https://www.elastic.co/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0
   * https://stackoverflow.com/questions/26001002/elasticsearch-difference-between-term-match-phrase-and-query-string
   * https://stackoverflow.com/questions/43530610/how-to-do-a-mapping-of-array-of-strings-in-elasticsearch
   * https://marcobonzanini.com/2015/02/09/phrase-match-and-proximity-search-in-elasticsearch/

## preface
* goals of this workshop
    * https://github.com/mtumilowicz/java12-elasticsearch-inverted-index-workshop
    * introduction to the basics of API
        * creating index, customize analyzers
        * define mappings
        * indexing documents
        * searching: query, filter
            * analyzing output
        * aggregating
* in `docker-compose` there is elasticsearch + kibana (7.6) prepared for local testing
    * cd `docker/compose`
    * `docker-compose up -d`
* workshop and answers are in `workshop` directory

## index
* note that 7.0 deprecated APIs that accept types, introduced new typeless APIs
* create index
    ```
    PUT /index-name
    {
        "settings": {
            "index" : {
                ... // configure index
            },
            "analysis": {
                ... // customize analyzer
            }
        },
        "mappings": {
            "properties": {
                ... // fields
            }
        }
    }
    ```
* field datatypes
    * any field can contain zero or more values by default, however, all values in the array 
    must be of the same datatype
    * string
        * text
            * full-text indexed (analyzed)
            * are not used for sorting and seldom used for aggregations
            * example: body of an email or the description of a product
            * sometimes it is useful to have multiple version of the same field: one for full 
            text search and the other for aggregations and sorting
        * keyword
            * are only searchable by their exact value (not analyzed)
            * typically used for filtering, sorting, and aggregations
            * example: IDs, email addresses, hostnames, status codes, zip codes or tags
    * numeric
        * byte, short, integer, long ...
    * date
        ```
        "date": {
            "type":   "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
        ```
        * internally, dates are converted to UTC (if the time-zone is specified) and stored as 
        a long number representing milliseconds-since-the-epoch
        * queries are internally converted on this long representation, and the result is converted 
        back to a string according to the field's date format
    * many more: range, object, nested, geo-points...
* indexing documents
    * each document indexed is versioned
    ```
    POST /index-name/_create/id
    {
        ... // fields
    }
    ```
* deleting documents
    * optimistic locking - version can be specified
    ```
    DELETE /index-name/_doc/id
    ```
* updating documents
    * optimistic locking - version can be specified
    * partial update
    * by default updates detect if they don’t change anything and return `"result": "noop"`
    ```
    POST /index-name/_update/id
    {
        "doc" : {
            "name" : "new_name"
        }
    }
    ```
  
## search
### mechanics
* every node in the cluster can handle HTTP and Transport traffic by default
* all nodes know about all the other nodes in the cluster and can forward client requests 
to the appropriate node
* search requests may involve data held on different data nodes
* a search request is executed in two phases which are coordinated by the node which receives the 
client request  —  the coordinating node
    * scatter phase
        * coordinating node forwards the request to the data nodes which hold the data
        * each data node executes the request locally and returns its results to the coordinating node
    * gather phase, the coordinating node reduces each data node’s results into a single global resultset

### API
* template for search requests
    ```
    GET index-name/_search // you can search entire cluster: GET /_search {...}
    {
        "query": {
            ...
        }
    }
    ```
* query match
    * text is analyzed before matching
    * standard query for performing a full-text search
    * per field
    ```
    "match" : {
        "field-name" : {
            "query" : "..."
        }
    }
    ```
* query query_string
    * text is analyzed before matching
    * if no field default - search all document
    * create a complex search that includes wildcard characters, searches across multiple fields
    ```
    "query_string" : {
        "query" : "..."
    }
    ```
* query match_phrase
    * text is analyzed before matching
    * all the terms must appear in the field
    * terms must have the same order
        * configured `slop` - how far we allow the terms to be
            * `this is a brown dog` and `the dog is brown` are OK with `slope = 1`
    ```
    "match_phrase" : {
        "field-name" : {
            "query": "...",
            "slop": 1
        }
    }
    ```
* query range
    ```
    "range" : {
        "field-name" : {
            "gte" : 10,
            "lte" : 20,
        }
    }
    ```
* query term
    * text is NOT analyzed before matching
    * exact matching
    ```
    "term": {
        "field-name": "..."
    }
    ```
* query bool
    * boolean combinations of other queries
    ```
    "bool" : {
        "must" : {
            ...
        },
        "filter": {
            ...
        },
        "must_not" : {
            ...
        },
        "should" : [
            ...
        ],
    }
    ```
    * must
        * must appear in matching documents
        * contributes to the score
    * must_not
        * must not appear in the matching documents
        * scoring is ignored
        * considered for caching
    * filter
        * must appear in matching documents
        * bypass analysis
            * filtering 'Andy' when indexed is 'andy' will give no hit
        * score of the query will be ignored
        * considered for caching
        * Elasticsearch constructs a bitset, which is a binary set of bits denoting whether 
        the document matches this filter
    * should
        * should appear in the matching document
        * contributes to the score
### response body
```
{
    "took" : 1, // in milliseconds
    "timed_out" : false,
    "_shards" : { // count of shards used for the request
        "total" : 1, // total number of shards that require querying
        "successful" : 1, // number of shards that executed the request successfully
        "skipped" : 0,
        "failed" : 0 // number of shards that failed to execute the request
    },
    "hits" : { // documents and metadata
        "total" : { // metadata about the number of returned documents
            "value" : 2, // total number of returned documents
            "relation" : "eq"
        },
        "max_score" : 0.9395274, // highest returned document _score
        "hits" : [ // array of returned document objects
            {
                "_index" : "programming-user-groups",
                "_id" : "2",
                "_score" : 0.9395274,
                "_source" : { ... } // original JSON body
            } 
        ]
    }
}

```
* took
    * measuring the time elapsed between receipt of a request on the coordinating node and the time at 
    which the coordinating node is ready to send the response
* _shard.skipped
    * skipped the request because a lightweight check helped realize that no documents could possibly 
    match on this shard
    * typically happens when a search request includes a range filter and the shard only has values that 
    fall outside of that range
* hits.relation
    * indicates whether the number of returned documents in the value parameter is accurate or a lower bound
        * eq: Accurate
        * gte: Lower bound, including returned documents

## aggregate
* template
    ```
    GET /programming-user-groups/_search
    {
        "aggs": {
            "agg-name" : {
                "agg-type": { ... },
                "aggs":{ // sub aggregations
                    "sub-agg-name": {
                        "sub-agg-type": { ... }
                    }
                }
            }
        }
    }
    ```
* types
    * bucketing
        * each bucket is associated with a key and a document criterion
        * when a criterion matches, the document is considered to "fall in" the relevant bucket
        * aggregations can be nested
            * bucketing aggregations can have sub-aggregations (bucketing or metric)
        * types: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html
        * response
            ```
            {
                ... // same as search response
                "aggregations" : {
                    "genres" : {
                        ...
                        "buckets" : [ 
                            {
                                "key" : "electronic",
                                "doc_count" : 6
                            },
                            {
                                "key" : "rock",
                                "doc_count" : 3
                            },
                            {
                                "key" : "jazz",
                                "doc_count" : 2
                            }
                        ]
                    }
                }
            }
            ```
    * metrics
        * refer to the statistical analysis
        * example: minimum value, maximum value, standard deviation, and much more
        * types: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html
        * response
            ```
            {
                ... // same as search response
                "aggregations": {
                    "max_price": {
                        "value": 200.0
                    }
                }
            }
            ```
