# 4. Searching your data

## 4.1 Structure of a search request

### 4.1.1 Specifying a search scope

### 4.1.2 Basic components of a search request

### 4.1.3 Request body-based search request

### 4.1.4 Understanding the structure of a response

## 4.2 Introducting the query and filter DSL

### 4.2.1 Match query and term filter

### 4.2.2 Most used basic queries and filters

1. `match_all` query

    - `match_all` query is the default query, so

        ```json
        POST get-together/_search?size=20
        ```

        is equivalent to

        ```json
        POST get-together/_search
        {
          "query": {
              "match_all": {}
          },
          "size": 20
        }
        ```

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

        is equivalent to

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

    is equivalent to

    ```json
    POST get-together/_search
    {
      "query": {
          "match": {
            "name": {
                "type": "boolean", 
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

1. `phrase_prefix` type

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

### 4.2.4 Matching multiple fields with `multi_match`

`nulti_match` query allows you to search for a value across multiple fields

1. Boolean query behavior

    ```json
    POST get-together/_search
    {
      "query": {
          "multi_match": {
            "type": "boolean",
            "query": "big data",
            "fields": [
                "name",
                "description"
            ],
            "operator": "AND"
          }
      }
    }
    ```

1. Phrase query behavior

    ```json
    POST get-together/_search
    {
      "query": {
          "multi_match": {
            "type": "phrase",
            "query": "hadoop elasticsearch",
            "fields": [
                "title",
                "description"
            ],
            "slop": 1
          }
      }
    }
    ```

1. `phrase_prefix` type

    ```json
    POST get-together/_search
    {
      "query": {
          "multi_match": {
            "type": "phrase_prefix",
            "query": "real-time elas",
            "fields": [
                "title",
                "description"
            ]
          }
      }
    }
    ```

## 4.3 Combining queries or compound queries

### 4.3.1 `bool` query

The `bool` query allows you to combine any number of queries into a single query by specifying a query clause that indicates which parts `must`, `should`, or `must_not` match the data in your Elasticsearch index:

```json
POST get-together/_search
{
   "query": {
      "bool": {
         "must": [
            {
               "term": {
                  "host": {
                     "value": "andy"
                  }
               }
            },
            {
               "match": {
                  "title": {
                     "query": "big data",
                     "type": "phrase",
                     "slop": 0
                  }
               }
            }
         ],
         "should": [
            {
               "term": {
                  "attendees": "michael"
               }
            },
            {
               "term": {
                  "attendees": "andy"
               }
            },
            {
               "term": {
                  "attendees": "simon"
               }
            }
         ],
         "must_not": [
            {
               "range": {
                  "date": {
                     "lt": "2013-06-20T00:00"
                  }
               }
            }
         ],
         "minimum_number_should_match": 2
      }
   }
}
```

### 4.3.2 `bool` filter

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "query": {
            "match_all": {}
         },
         "filter": {
            "bool": {
               "must": [
                  {
                     "term": {
                        "host": {
                           "value": "andy"
                        }
                     }
                  },
                  {
                     "match": {
                        "title": {
                           "query": "big data",
                           "type": "phrase",
                           "slop": 0
                        }
                     }
                  }
               ],
               "should": [
                  {
                     "term": {
                        "attendees": "michael"
                     }
                  },
                  {
                     "term": {
                        "attendees": "andy"
                     }
                  },
                  {
                     "term": {
                        "attendees": "simon"
                     }
                  }
               ],
               "must_not": [
                  {
                     "range": {
                        "date": {
                           "lt": "2013-06-20T00:00"
                        }
                     }
                  }
               ],
               "minimum_number_should_match": 2
            }
         }
      }
   }
}
```

## 4.4 Beyond match and filter queries

### 4.4.1 `range` query and filter

1. `range` query

    ```json
    POST get-together/_search
    {
      "query": {
          "range": {
            "date": {
                "from": "2013-04-08T00:00",
                "to": "2013-06-01T00:00"
            }
          }
      }
    }
    ```

1. `range` filter

    ```json
    POST get-together/_search
    {
      "query": {
          "filtered": {
            "query": {
                "match_all": {}
            },
            "filter": {
                "range": {
                  "date": {
                      "gt": "2013-05-01T00:00",
                      "lte": "2013-06-17T18:00"
                  }
                }
            }
          }
      }
    }
    ```

### 4.4.2 `prefix` query and filter

Similar to the `term` query, the `prefix` query and filter allow you to search for a term containing the given prefix, where the prefix isn't analyzed before searching.

```json
POST get-together/_search
{
   "query": {
      "prefix": {
         "title": "cloj"
      }
   }
}
```

and

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "query": {
            "match_all": {}
         },
         "filter": {
            "prefix": {
               "title": "cloj"
            }
         }
      }
   }
}
```

### 4.4.3 `wildcard` query

The `wildcard` query is close to the way shell wildcard globbing works

```json
POST get-together/_search
{
   "query": {
      "wildcard": {
         "description": "e?ter*"
      }
   }
}
```

## 4.5 Querying for field existence with filters

### 4.5.1 `exists` filter

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "filter": {
            "exists": {
               "field": "location_event.geolocation"
            }
         }
      }
   }
}
```

### 4.5.2 `missing` filter

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "filter": {
            "missing": {
               "field": "reviews",
               "existence": true,
               "null_value": true
            }
         }
      }
   }
}
```

## 4.6 Transforming any query into a filter

Sometimes you may want to take a query such as `query_string`, which has no filter equivalent, and turn it into a filter. Elasticsearch allows you to do this with the `query` filter, which takes any query and turns it into a filter.

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "filter": {
            "query": {
               "query_string": {
                  "query": "title:elasticsearch AND (description:logstash OR description:kibana)"
               }
            }
         }
      }
   }
}
```

You can also choose to cache using `fquery` filter and `_cache` key

```json
POST get-together/_search
{
   "query": {
      "filtered": {
         "filter": {
            "fquery": {
               "query": {
                  "query_string": {
                     "query": "title:elasticsearch AND (description:logstash OR description:kibana)"
                  }
               },
               "_cache": true
            }
         }
      }
   }
}
```

## 4.7 Choosing the best query for the job

| Use case | Query type to use |
| -------- | ----------------- |
| You want to take input from a user, similar to a Google-style interface, and search for documents with the input. | Use a `match` query or the `simple_query_string` query if you want to support +/- and search in specific fields. |
| You want to take input as a phrase and search for documents containing that phrase, perhaps with some amount of leniency (slop). | Use a `match_phrase` query with an amount of slop to find phrases similar to what the use is search for. |
| You want to search for a single word in a `not_analyzed` field, knowing exactly how the word should appear. | Use a `term` query because query terms aren't analyzed. |
| You want to combine many different searches or types of searches, creating a single search out of themm. | Use the `bool` query to combine any number of subqueries into a single query. |
| You want to search for centain words across many fields in a document. | Use the `multi_match` query, which behaves similarly to the `match` query but on multiple fields. |
| You want to return every document from a search. | Use the `match_all` query to return all documents from a search. |
| You want to search a field for values that are between two specified values. | Use a `range` query to search within documents with values between a certain range. |
| You want to search a field for values that start with a specified string. | Use a `prefix` query to search for terms starting with a given string. |
| You want to autocomplete the value of a single word based on what the user has already typed in. | Use a `prefix` query to send what the user has typed in and get back exact matches starting with the text |
| You want to search for all documents that have no value for a specified field. | Use the `missing` filter to filter out documents that are missing fields. |
