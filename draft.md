# JSKOS API

This document describes a JSON-API to access controlled vocabularies.

## General properties

* HTTPS GET requests only (later version may also support write operations)
* support CORS and JSONP (`callback` parameter)
* pretty-print JSON and gzip compression by default (SHOULD)

Requests

* pagination with HTTP `Link` header (rel=next, rel=prev) and possible limit/count (?)
* `properties` query parameter to include/exclude fields: comma-separated list of names, optionally prepended with `-`

## Endpoints

Core methods

* `schemes` - list known schemes
* `topConcepts` - list top concepts of a scheme
* `types` - list known types
* `concept` - get concept by URI
* `search` - find concepts with given properties (also supports POST with JSON body)

Utility methods

* `broader` (possibly with transitives)
* `narrower` (possibly with transitives)
* `related`
* `suggest` - search with OpenSearch suggestions return format

## Search

...
