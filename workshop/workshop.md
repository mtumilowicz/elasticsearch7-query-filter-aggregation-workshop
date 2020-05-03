1. we want to index documents
    ```
    {
        "name": "Denver Clojure",
        "organizer": ["Daniel", "Lee"],
        "description": "Group of Clojure enthusiasts from Denver who want to hack on code together and learn more about Clojure",
        "created_on": "2012-06-15",
        "tags": ["clojure", "denver", "functional programming", "jvm", "java"],
        "members": ["Lee", "Daniel", "Mike"],
        "location_group": "Denver, Colorado, USA"
    }
    ```
    ```   
    {
        "name": "Elasticsearch Denver",
        "organizer": "Lee",
        "description": "Get together to learn more about using Elasticsearch, the applications and neat things you can do with ES!",
        "created_on": "2013-03-15",
        "tags": ["denver", "elasticsearch", "big data", "lucene", "solr"],
        "members": ["Lee", "Mike"],
        "location_group": "Denver, Colorado, USA"
    }
    ```
    ```
    {
        "name": "Elasticsearch San Francisco",
        "organizer": "Mik",
        "description": "Elasticsearch group for ES users of all knowledge levels",
        "created_on": "2012-08-07",
        "tags": ["elasticsearch", "big data", "lucene", "open source"],
        "members": ["Lee", "Igor"],
        "location_group": "San Francisco, California, USA"
    }
    ```
    ```
    {
        "name": "Boulder/Denver big data get-together",
        "organizer": "Andy",
        "description": "Come learn and share your experience with nosql & big data technologies, no experience required",
        "created_on": "2010-04-02",
        "tags": ["big data", "data visualization", "open source", "cloud computing", "hadoop"],
        "members": ["Greg", "Bill"],
        "location_group": "Boulder, Colorado, USA"
    }
    ```  
    ```
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
1. propose appropriate index name and mapping
    * at least two shards and one replica (per primary shard)
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html
    * at least three different field types
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html
    * at least one dedicated field analyzer
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html
        * https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html
1. create index, analyzer and mapping
    * attach analyzer to at least one field
1. save (index) documents mentioned above
1. verify
    * that index has an analyzer
    * analyze exemplary query using analyzer
    * analyze exemplary query using default analyzer
    * verify that all documents were indexed
    * verify terms for some field of selected document
        * hint: `_termvectors`
1. search
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html
    * field `description` contains clojure or group (ignore case); better score if contains all
        * hint: `query.match`
    * document contains clojure and group, the more the higher the score
        * hint: `query.query_string`
    * description contains group clojure in that order in proximity - suppose at most 1 words between
        * compare with different slop and order
        * hint: `query.match_phrase`
    * find all created after 2011, first chunk: 10
        * manipulate from and size
        * hint: `query.range`
    * find all organized by Lee (not lee)
        * compare with lee - draw conclusions
        * hint: `query.term`
    * name has to contain elasticsearch and organizer cannot be Lee
        * compare with filter
        * hint: `query.bool.must`
    * event must contain group and organizer should be Lee
        * hint: `query.bool.should`
    * filter events that has tag clojure or lucene
        * hint: `query.bool.filter`
1. aggregations
    * https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html
    * group by tags and display count in each group
        * hint: `terms`
    * group by tags and display date of the latest event in each group
        * hint: pipeline aggregation
    * group by tags and display id and date of the latest event in each group
        * hint: `top_hits`