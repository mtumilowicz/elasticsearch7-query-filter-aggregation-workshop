* https://www.elastic.co/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0
* https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html
* https://stackoverflow.com/questions/43530610/how-to-do-a-mapping-of-array-of-strings-in-elasticsearch
* https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html
* https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html
* 7.0 deprecated APIs that accept types, introduced new typeless APIs, and removed support for the _default_ mapping

* create index + mapping
    PUT /index-name
    {
      "mappings": {
        "properties": {
          "email":  { "type": "keyword"  }, 
          "name":   { "type": "text"  },
          "date": {
            "type":   "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
          }
        }
      }
    }
    
    {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "programming-user-groups"
    }
* customize analyzer
    * actually create mirror standard tokenizer
* get all indices
    * GET /_cat/indices
* indexing new data
    * method: PUT 
    * url: hostname/index-name/id
    * body: document as a JSON
    * reply
        You should get the following output:
        {
        "_index" : index-name,
        "_id" : id,
        "_version" : 1,
        "created" : true
        }
* retrieving data
    * method: GET
    * url: hostname/index-name/id
    * reply