1. index
    ```
    PUT /programming-user-groups
    {
        "settings": {
            "index" : {
                "number_of_shards" : 2, 
                "number_of_replicas" : 2 
            },
            "analysis": {
                "analyzer": {
                    "english_standard": {
                        "type": "standard",
                        "stopwords": "_english_"
                    }
                }
            }
        },
        "mappings": {
            "properties": {
                "name": { "type": "text"  },
                "organizer": { "type": "text"  },
                "description": {
                    "type": "text",
                    "analyzer": "english_standard"
                },
                "created_on": {
                    "type":   "date",
                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"},
                "tags":  { "type": "keyword"  },
                "members":   { "type": "text"  },
                "location_group":   { "type": "text"  }
            }
        }
    }
    ```
1. indexing documents
    ```
    POST /programming-user-groups/_create/1
    {
        "name": "Denver Clojure",
        "organizer": ["Daniel", "Lee"],
        "description": "Group of Clojure enthusiasts from Denver who want to hack on code together and learn more about Clojure",
        "created_on": "2012-06-15",
        "tags": ["clojure", "denver", "functional programming", "jvm", "java"],
        "members": ["Lee", "Daniel", "Mike"],
        "location_group": "Denver, Colorado, USA"
    }
    
    POST /programming-user-groups/_create/2
    {
        "name": "Elasticsearch Denver",
        "organizer": "Lee",
        "description": "Get together to learn more about using Elasticsearch, the applications and neat things you can do with ES!",
        "created_on": "2013-03-15",
        "tags": ["denver", "elasticsearch", "big data", "lucene", "solr"],
        "members": ["Lee", "Mike"],
        "location_group": "Denver, Colorado, USA"
    }
    
    POST /programming-user-groups/_create/3
    {
        "name": "Elasticsearch San Francisco",
        "organizer": "Mik",
        "description": "Elasticsearch group for ES users of all knowledge levels",
        "created_on": "2012-08-07",
        "tags": ["elasticsearch", "big data", "lucene", "open source"],
        "members": ["Lee", "Igor"],
        "location_group": "San Francisco, California, USA"
    }
    
    POST /programming-user-groups/_create/4
    {
        "name": "Boulder/Denver big data get-together",
        "organizer": "Andy",
        "description": "Come learn and share your experience with nosql & big data technologies, no experience required",
        "created_on": "2010-04-02",
        "tags": ["big data", "data visualization", "open source", "cloud computing", "hadoop"],
        "members": ["Greg", "Bill"],
        "location_group": "Boulder, Colorado, USA"
    }
    
    POST /programming-user-groups/_create/5
    {
        "name": "Enterprise search London get-together",
        "organizer": "Tyler",
        "description": "Enterprise search get-togethers are an opportunity to get together with other people doing search.",
        "created_on": "2009-11-25",
        "tags": ["enterprise search", "apache lucene", "solr", "open source", "text analytics"],
        "members": ["Clint", "James"],
        "location_group": "London, England, UK"
    }
    ```
1. verify
    * that index has an analyzer
        ```
        GET programming-user-groups/_settings      
        ```
    * analyze exemplary query using analyzer
        ```
        POST programming-user-groups/_analyze
        {
          "analyzer": "english_standard",
          "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
        }
        ```
    * analyze exemplary query using default analyzer
        ```
        POST programming-user-groups/_analyze
        {
          "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
        }
        ```
    * verify that all documents were indexed
        ```
        GET /programming-user-groups/_doc/1
        ```
    * verify terms for some selected document
        ```
        GET /programming-user-groups/_termvectors/1
        {
          "fields" : ["description"],
          "offsets" : true,
          "payloads" : true,
          "positions" : true,
          "term_statistics" : true,
          "field_statistics" : true
        }
        ```
1. search
    * description contains clojure or group (ignore case); better score if contains all
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "match" : {
                    "description": {
                        "query" : "group clojure"
                    }
                }
            }
        }      
        ```
    * contains clojure and group, the more the higher the score
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "query_string" : {
                    "query" : "clojure AND group"
                }
            }
        }
        ```
    * description contains group clojure in that order in proximity - suppose at most 1 words between
        * compare with different slop and order
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "match_phrase" : {
                    "description" : {
                        "query": "group clojure",
                        "slop": 1
                    }
                }
            }
        }
        ```
    * find all created after 2011, first chunk: 10
        * manipulate from and size
        ```
        GET programming-user-groups/_search
        {
            "from" : 0, "size" : 2,
            "query": {
                "range" : {
                    "created_on" : {
                        "gte" : "2011-01-01"
                    }
                }
            }
        }
        ```
    * find all organized by Lee (not lee)
        * compare with lee - draw conclusions
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "term": {
                    "organizer": "lee"
                }
            }
        }
        ```
    * name has to contain elasticsearch and organizer cannot be Lee
        * compare with filter
        ```
        GET programming-user-groups/_search
        {
          "query": {
            "bool" : {
              "must" : {
                "match" : { "name" : "elasticsearch" }
              },
              "must_not" : {
                  "match": { "organizer" : "Lee" }
              }
            }
          }
        }
        ```
    * event must contain group and organizer should be Lee
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "bool" : {
                    "must" : {
                        "query_string" : {
                          "query": "group"
                        }
                    },
                    "should": {
                        "match": { "organizer" : "Lee" }
                    }
                }
            }
        }      
        ```
    * filter events that has tag clojure or lucene
        ```
        GET programming-user-groups/_search
        {
            "query": {
                "bool" : {
                    "filter" : {
                        "terms" : {
                            "tags": ["clojure", "lucene"]
                        }
                    }
                }
            }
        }
        ```
1. aggregations
    * group by tags and display count in each bucket
        ```
        POST /programming-user-groups/_search
        {
            "aggs": {
                "by_tag" : { 
                    "terms": { "field" : "tags" }
                  }
            }   
        }
        ```
    * group by tags and display date of the newest group in each bucket
        ```
        POST /programming-user-groups/_search
        {
            "aggs": {
                "by_tag" : {
                    "terms": { "field" : "tags" },
                    "aggs":{
                        "max_date_in_bucket": {
                            "max": { "field": "created_on" }
                        }
                    }
                }
            }
        }
        ```
    * group by tags and display id and date of the newest group in each bucket
        ```
        POST /programming-user-groups/_search
        {
            "aggs": {
                "by_tag": {
                    "terms": { "field": "tags" },
                    "aggs": {
                        "latest_event_id": {
                            "top_hits": {
                                "_source": ["_id", "created_on"], 
                                "size": 1,
                                "sort": [ { "created_on": { "order": "desc" } } ]
                            }
                        }
                    }
                }
            }
        }
        ```