# Task
* Create ES index that will serve autocomplete needs with leveraging typos and errors (max 3 typos if word length is bigger than 7).
* Please use english voc. Ang look at google as a ref.

# Solution

1. Run docker compose
```shell
$ docker-compose up -d
```

2. Access to Kibana console
http://127.0.0.1:5601/app/dev_tools#/console

3. Create index

```http request
PUT /autocomplete
{
  "mappings": {
       "properties": {
           "word": {
               "type": "search_as_you_type"
           }
       }
   }
}
```
4. Load data from words.json.es
```http request
POST _bulk
{ "index" : { "_index" : "autocomplete" } }
{ "word" : "a"}
{ "word" : "aam"}
{ "index" : { "_index" : "autocomplete" } }
{ "word" : "aaronic"}
{ "index" : { "_index" : "autocomplete" } }
{ "word" : "aaronical"}
...
```

OR

Load data from words.json.es
```shell
curl -s -H "Content-Type: application/x-ndjson" -XPOST 127.0.0.1:9200/_bulk --data-binary "@words.json.es";
```

5. Check that all documents loaded
```http request
GET /autocomplete/_count
```

```json
{
  "count": 113846,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

6. Simple search with mistakes
```http request
GET /autocomplete/_search
{
  "query": {
    "fuzzy": {
      "word": "obedieqciara"
    }
  }
}
```

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 9.422148,
    "hits": [
      {
        "_index": "autocomplete",
        "_id": "E73X9IEBSJZlDoA525cc",
        "_score": 9.422148,
        "_source": {
          "word": "obedienciary"
        }
      }
    ]
  }
}
```
7. Advanced `fuzzy` search
```http request
GET /autocomplete/_search?_source
{
  "query": {
    "fuzzy": {
      "word": {
        "value": "bedie",
        "fuzziness": 2,
        "prefix_length": 3,
        "max_expansions": 7
      }
    }
  }
}
```

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 8,
      "relation": "eq"
    },
    "max_score": 8.634077,
    "hits": [
      {
        "_index": "autocomplete",
        "_id": "vbzX9IEBSJZlDoA517Bu",
        "_score": 8.634077,
        "_source": {
          "word": "bedim"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "8rzX9IEBSJZlDoA517Bv",
        "_score": 8.634077,
        "_source": {
          "word": "bedye"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "nrzX9IEBSJZlDoA517Bu",
        "_score": 8.094447,
        "_source": {
          "word": "bede"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "mrzX9IEBSJZlDoA517Bt",
        "_score": 6.475558,
        "_source": {
          "word": "bedded"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "m7zX9IEBSJZlDoA517Bt",
        "_score": 6.475558,
        "_source": {
          "word": "bedded"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "przX9IEBSJZlDoA517Bu",
        "_score": 6.475558,
        "_source": {
          "word": "bedel"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "qbzX9IEBSJZlDoA517Bu",
        "_score": 6.475558,
        "_source": {
          "word": "beden"
        }
      },
      {
        "_index": "autocomplete",
        "_id": "srzX9IEBSJZlDoA517Bu",
        "_score": 6.475558,
        "_source": {
          "word": "bedew"
        }
      }
    ]
  }
}
```