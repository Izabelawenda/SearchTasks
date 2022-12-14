<h1>Exercise:</h1>

1. Create a search request to the products index using a match_all query and terms aggregation by the category_path field. Try to execute it. Why does it fail with an exception?
2. Create the same request with a match_all query.

   Add to it aggregations:
   
    - terms aggregation by name
    - range aggregation by price with ranges: ..70, 70..130, 130..200, 200.. ; ranges must have the following keys in the response: very cheap, cheap, expensive, very expensive
    - sub aggregation in the previous one that is nested aggregation by skus path with terms aggregation by the skus.color field inside it with a reverse_nested aggregation
    - filter aggregation with filter “skus.color = Black” and a stats sub aggregation (it must have “count” in the response > 0).

    Add any post filter to this request that leaves > 0 and < 6 documents in the response.

    Check how it works.

<h3>Solution:</h3>

1. .


    GET products/_search
    {
      "query": {"match_all": {}},
      "aggs": {
        "category_aggregation": {
          "terms": {"field": "category_path"}
    }}}

Response:

    {
    "error": {
    "root_cause": [
    {
    "type": "illegal_argument_exception",
    "reason": "Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [category_path] in order to load field data by uninverting the inverted index. Note that this can use significant memory."
    }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
    {
    "shard": 0,
    "index": "products",
    "node": "cndfC5VETUSJpKa_YFdSDA",
    "reason": {
    "type": "illegal_argument_exception",
    "reason": "Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [category_path] in order to load field data by uninverting the inverted index. Note that this can use significant memory."
    [...]
2. .


      GET products/_search
      {
        "size": 0,
        "query": {
          "match_all": {}
        },
        "aggs": {
          "name_agg": {
            "terms": {
              "field": "name"
            }
          },
          "price_range_agg": {
            "range": {
              "field": "price",
              "ranges": [
                {"key": "very cheap", "to": 70},
                {"key": "cheap","from": 70, "to": 130},
                {"key": "expensive","from": 130, "to": 200},
                {"key": "very expensive", "from": 200}
              ]
            },
            "aggs": {
              "color_agg": {
                "nested": {
                  "path": "skus"
                },
                "aggs": {
                  "inner": {
                    "filter": {
                      "term": {
                        "skus.color": "Black"
                      }
                    },
                    "aggs": {
                      "color_nested_agg": {
                        "terms": {
                          "field": "skus.color"
                        },
                        "aggs": {
                          "product_cnt": {
                            "reverse_nested": {}
      }}}}}}}}}}}

With added post_filter, filtered by brand "Nike". Returns 3 documents.

      GET products/_search
      {
        "size": 0,
        "query": {
          "match_all": {}
        },
        "post_filter": {"term": {"brand": "Nike"}},
        "aggs": {
          "name_agg": {
            "terms": {
              "field": "name"
            }
          },
          "price_range_agg": {
            "range": {
              "field": "price",
              "ranges": [
                {"key": "very cheap", "to": 70},
                {"key": "cheap","from": 70, "to": 130},
                {"key": "expensive","from": 130, "to": 200},
                {"key": "very expensive", "from": 200}
              ]
            },
            "aggs": {
              "color_agg": {
                "nested": {
                  "path": "skus"
                },
                "aggs": {
                  "inner": {
                    "filter": {
                      "term": {
                        "skus.color": "Black"
                      }
                    },
                    "aggs": {
                      "color_nested_agg": {
                        "terms": {
                          "field": "skus.color"
                        },
                        "aggs": {
                          "product_cnt": {
                            "reverse_nested": {}
      }}}}}}}}}}}