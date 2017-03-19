---
layout:     post
title:      "Elasticsearch using CloudConnect"
date:       2017-03-19 18:10:00
author:     "Nikolai Magnussen"
header-img: "img/elastic-logo.png"
---

# Telenor CloudConnect
[Telenor CloudConnect](http://www.telenorconnexion.com/services/cloud-connect) is a new service focusing on providing users with all parts of the infrastructure related to IoT devices; among others.
In particular, it can be used together with [LPWAN](https://en.wikipedia.org/wiki/LPWAN).
Once connected to the platform, devices can send data, which at some later will be retrieved.
Because Telenor CloudConnect utilize AWS as it's backend, it is highly scalable as well as providing out-of-the-box [elasticsearch](https://www.elastic.co/) capabilities for all data being sent to the platform.

Elasticsearch provide flexible, scalable and distributed searching based on indexed attributes. Because all data sent to Telenor CloudConnect, all attributes will be indexed, and thus searchable by taking advantage of the [elasticsearch DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

## Elastic Search DSL
The elasticsearch capabilities can be taken advantage of through the Observation API for Telenor CloudConnect.

The payload is on the following form:
```JSON
{
    "action": "FIND",
    "query": {
        <Your Elastic Search DSL query here>
    }
}
```

### Simple leaf queries
The interesting part is, for obvious reasons; the Elastic Search query.
The simplest query is simply a `"match_all": {}` statement, which will return all observations. You will probably notice that less than the number of hits will get displayed.
This query can look like this:
```JSON
{
    "action": "FIND",
    "query": {
        "size": 100,
        "query": {
            "match_all": {}
        }
    }
}
```
And you can add a result size, to either reduce or increase the number of result returned.

To do something more exciting, we can search for some term, which is on the following form:
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "term": {
                "state.<attribute>": <exact-value>
            }
        }
    }
}
```

Which mean that we can search for all observations where the lsnr (a measure of signal-to-noise ratio) is equal to 5:
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "term": {
                "state.lsnr": 5
            }
        }
    }
}
```

But knowing exactly which value to search for is somewhat boring, or you simply want all observations in a certain range:
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "range": {
                "state.<attribute>": {
                    "<parameter>": <value>,
                    "<optional parameter>": <value>
                }
            }
        }
    }
}
```
Where normal range parameters can be `gte`, `gt`, `lte`, `lt`.

Thus, we can find all observations of lsnr whose value is between 2.5 and 5:
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "range": {
                "state.lsnr": {
                    "gte": 2.5,
                    "lt": 5
                }
            }
        }
    }
}
```

### Compound queries
Currently, we have only used very simple queries, by only searching for one term, which is called a [leaf query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html), but we can also use [compound queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html):
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "bool": {
                "must": {
                    "<query that should contribute to score and must be present>"
                },
                "filter": {
                    "<query that must be present, but does not contribute to score>"
                }
            }
        }
    }
}
```

To apply this to our case, we probably only want result regarding our thing, which can be achieved by using compound queries:
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "bool": {
                "must": {
                    "range": {
                        "state.lsnr": {
                            "lte": 5,
                            "gt": 2.5
                        }
                    }
                },
                "filter": {
                    "term": { "thingName": "00000263" }
                }
            }
        }
    }
}
```

If you only want to get the result where a particular field exists, [range](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html) can be replaced with [exists](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-exists-query.html):
```JSON
{
    "action": "FIND",
    "query": {
        "query": {
            "bool": {
                "must": {
                    "exists": { "field": "state.<attribute>"}
                },
                "filter": {
                    "term": { "thingName": "<8-digit thing id with leading 0's>" }
                }
            }
        }
    }
}
```

It is easy to see that many of the searches result in a huge number of attributes being returned. This can be reduced by applying the [fields](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-request-fields.html) filtering:
```JSON
{
    "action": "FIND",
    "query": {
        "fields": ["state.<attribute>", "<other attributes to retain>"],
        "query": {
            "bool": {
                "must": {
                    "exists": { "field": "state.<attribute>" }
                },
                "filter": {
                    "term": { "thingName": "<8-digit thing id with leading 0's>" }
                }
            }
        }
    }
}
```

### Aggregations
```JSON
{
    "action": "FIND",
    "query": {
        "fields": ["state.<attribute>", "<other attributes to retain>"],
        "query": {
            "bool": {
                "must": {
                    "exists": { "field": "state.<attribute>" }
                },
                "filter": {
                    "term": { "thingName": "<8-digit thing id with leading 0's>" }
                }
            }
        },
        "aggs": {
            "hist": {
                "date_histogram": {
                    "field": "timestamp",
                    "interval": "second",
                    "time_zone": "+01:00"
                }
            }
        }
    }
}
```

I hope this has been enlightening, and that you further explore the flexibility and capabilities elasticsearch provide.
