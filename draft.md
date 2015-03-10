# JSKOS API

This document describes a JSON-API to access controlled vocabularies.

## General properties

* HTTPS GET requests only (later version may also support write operations)
* support CORS and optionally JSONP (`callback` parameter)
* pretty-print JSON and gzip compression by default (SHOULD)
* Simple Auth (token as username) or OAuth 2 for restricted access

## Request methods

### Core methods

* `schemes` - list known schemes
* `topConcepts/top` - list top concepts of a scheme
* `types` - list known types
* `concepts` - find concepts with given properties

If schemes have unique notations, request methods can be put in URL structure like this.

    /schemes
    /schemes/ddc
    /schemes/ddc/top
    /schemes/ddc/types
    /schemes/ddc/concepts
    /schemes/ddc/...
    
TODO: *how to publish, which methods are available where?*

### Utility methods

Can be created from the core method `concepts`:

* `concept` - get concept by URI
* `broader` (possibly with transitives)
* `narrower` (possibly with transitives)
* `related`
* `search` with HTTP POST JSON
* `suggest` - search with OpenSearch suggestions return format

## Request parameters

### Properties

The query parameter `properties` can be used to include a particular set of properties, given as comma-separated list of property names:

    ?properties=notation,prefLabel,altLabel,hiddenLabel
    ?properties=broader,narrower
    ...

The `uri` property is always included.

### Pagination

TODO: pagination with HTTP `Link` header (rel=next, rel=prev) and possible limit/count (?)

## Concept search

### Simple search (`concepts`)

Search via query parameters, combined as boolean AND (boolean OR is not supported). Concept search supports the following concept properties:

* `prefLabel`, `altLabel`, `hiddenLabel`
* `note` (includes any kind of note. more specific documentation properties may be added later)
* `notation`
* `type`
* `broader`, `narrower`, `related` (value must be an URI)

Label properties and documentation properties can be localized by appending `.` and the language tag, e.g. `prefLabel.en=fox`.
    
The field name `label` can be used to refer to any of `prefLabel`, `altLabel`, and `hiddenLabel`. For instance `label=fox` can be used to search for concepts with any kind of label having the value "fox". The name can also be localized e.g. `label.en=fox`.

### Search qualifiers

* Truncation (`*` or implicit?)
* case-folding (query parameter `case`)
* [accent-folding](http://alistapart.com/article/accent-folding-for-auto-complete) (query parameter `accent`) 

### Search with fields given as JSON

Including boolean operators on fields (`true`, `false`) for negation also, for instance to find concepts without notation or without English prefLabel:

    { "notation": [ ] }
    { "prefLabel": { "en": false } }
    
## Errors

HTTP Status code (400, 401, 403, 404, 405, 410, 415. 429...) and response body:

    {
      "code": ...,
      "message": ...,
      "description": ...details-such-as-field-names...
    }
    
## Additional data

For instance latitude, longitude, givenname, surname...

SHOULD be documented with a JSON-LD context document (to put where?).
