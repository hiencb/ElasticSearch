# 4. Searching your data

## 4.2 Introducting the query and filter DSL

### 4.2.1 Match query and term filter

### 4.2.2 Most used basic queries and filters

1. `match_all` query

    - `match_all` query is the default query, so

        ```json
        POST get-together/_search?size=20
        ```

        and

        ```json
        POST get-together/_search
        {
          "query": {
              "match_all": {}
          },
          "size": 20
        }
        ```

        are the same.

    - `match_all` query can be combined with a filter

        ```json
        POST get-together/_search
        {
          "query": {
              "filtered": {
                "query": {
                    "match_all": {}
                },
                "filter": {
                    "term": {
                        "host": "andy"
                    }
                }
              }
          },
          "size": 20
        }
        ```

1. `query_string` query

    - `query_string` search can be performed either from the URL or send in a request body, so

        ```json
        POST get-together/_search?q=andy
        ```

        and

        ```json
        POST get-together/_search
        {
          "query": {
              "query_string": {
                "query": "andy"
              }
          }
        }
        ```

        are the same.

    - By default a `query_string` query searches the _all field, but we can specify a field with the query, such as `description:nosql`, or by specify a `default_field` with the request

        ```json
        POST get-together/_search?q=title:elasticsearch
        ```

        and

        ```json
        POST get-together/_search
        {
          "query": {
              "query_string": {
                "query": "title:elasticsearch"
              }
          }
        }
        ```

        or

        ```json
        POST get-together/_search
        {
          "query": {
              "query_string": {
                "default_field": "title",
                "query": "elasticsearch"
              }
          }
        }
        ```

    - Using Lucene query syntax

        ```json
        POST get-together/_search
        {
          "query": {
              "query_string": {
                "query": "(tags:elasticsearch OR tags:lucene) AND (organizer:lee OR organizer:mik)"
              }
          }
        }
        ```

1. `term` query and `term` filter

    - The term being searched for isn't analyzed, it must match a term in the document exactly for the result to found

        ```json
        POST get-together/_search
        {
          "query": {
              "term": {
                "tags": "elasticsearch"
              }
          }
        }
        ```

    - A `term` filter can be used when you want to limit the results to documents that contain the term but without affecting the score.

        ```json
        POST get-together/_search
        {
          "query": {
              "filtered": {
                "filter": {
                    "term": {
                      "tags": "hadoop"
                    }
                }
              }
          }
        }
        ```

1. `terms` query

    - The `terms` query can search for multiple terms in a document's field.

        ```json
        POST get-together/_search
        {
          "query": {
              "terms": {
                "tags": [
                    "elasticsearch",
                    "hadoop"
                ]
              }
          }
        }
        ```

### 4.2.3 `match` query and `term` filter

The `match` query is a hash map containing the field you'd like to search as well as the string you want to search for, which can be either a field or the speciel `_all` field to search all fields at once.

```json
POST get-together/_search
{
  "query": {
      "match": {
        "_all": "elasticsearch"
      }
  }
}
```

1. Boolean query behavior

    By default, the `match` query uses Boolean behavior and the OR operator, so

    ```json
    POST get-together/_search
    {
      "query": {
          "match": {
            "name": "elasticsearch denver"
          }
      }
    }
    ```

    is equal to

    ```json
    POST get-together/_search
    {
      "query": {
          "match": {
            "name": {
                "query": "elasticsearch denver",
                "operator": "OR"
            }
          }
      }
    }
    ```

    To search for results that contain both "elasticsearch" and "denver", set the `operator` field to `AND`

    ```json
    POST get-together/_search
    {
      "query": {
          "match": {
            "name": {
                "query": "elasticsearch denver",
                "operator": "AND"
            }
          }
      }
    }
    ```

1. Phrase query behavior

    A `phrase` query is useful when searching for a specific phrase within a document, with some amount of leeway between the positions of each word. This leeway is called `slop`, which is a number representing the distance between tokens in a phrase.

    ```json
    POST get-together/_search
    {
      "query": {
          "match": {
            "title": {
                "type": "phrase",
                "query": "liberator immutant",
                "slop": 1
            }
          }
      }
    }
    ```

    It is equivalent to

    ```json
    POST get-together/_search
    {
      "query": {
          "match_phrase": {
            "title": {
                "query": "liberator immutant",
                "slop": 1
            }
          }
      }
    }
    ```

### 4.2.4 `phrase_prefix` query

The `match_phrase_prefix` query allows prefix matching on the last term in the phrase. This behavior is extremely useful for providing a running autocomplete for a search box.

```json
POST get-together/_search
{
   "query": {
      "match": {
         "title": {
             "type": "phrase_prefix", 
            "query": "elasticsearch trai"
         }
      }
   }
}
```

is equivalent to

```json
POST get-together/_search
{
   "query": {
      "match_phrase_prefix": {
         "title": {
            "query": "elasticsearch trai"
         }
      }
   }
}
```