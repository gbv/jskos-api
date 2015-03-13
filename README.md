# JSKOS API

This document describes an experimental JSON-API to access controlled
vocabularies in [JSKOS](http://gbv.github.io/jskos/) format.

The specification is hosted in a public git repository at
<https://github.com/gbv/jskos-api>. The repository includes a textual
description (this document) and a [Swagger](http://swagger.io/) file
(`jskos-api.yaml`). Comments and contributions are welcome!

## Synopsis

A JSKOS API endpoint for *one concept scheme* ("JSKOS API ONE") MUST provide at
least three URL paths for HTTPS GET requests on a common base URL
(`https://example.org/jskos/` in this example):

* `https://example.org/jskos` - get data about the concept scheme
* `https://example.org/jskos/concepts` - get data about its concepts
* `https://example.org/jskos/types` - get data about its concept types

A JSKOS API endpoint for *multiple concept schemes* ("JSKOS API MANY") MUST
provide an URL path ending with `/schemes` for HTTPS GET requests on a common
base URL (`https://example.org/jskos/` in this example), and a JSKOS API for
each concept scheme on a sub-path as following.

* `https://example.org/jskos/schemes` - get data about concept schemes
* `https://example.org/jskos/schemes/{scheme}` - JSOS API ONE for a concept scheme
* `https://example.org/jskos/schemes/{scheme}/concepts` - JSOS API ONE for a concept scheme
* `https://example.org/jskos/schemes/{scheme}/types` - JSOS API ONE for a concept scheme

A JSKOS API endpoint *may* support [additional methods](#additional-methods)
which can all be implemented based on the core methods.

## Basics

* HTTPS GET requests only (a later version may also support write operations)
* JSON content type only, based on [JSKOS](http://gbv.github.io/jskos/) format.
* support CORS and optionally JSONP (`callback` parameter)
* *should* pretty-print JSON and gzip compression by default
* *may* support Simple Auth (token as username) or OAuth 2 for restricted access
* HTTP error responses must return a JSON document as response body with fields
  `code`, `message`, and `description`.
 
## Request methods

### Core methods

* `/schemes`
* `/schemes/{scheme}`
* `/schemes/{scheme}/types`
* `/schemes/{scheme}/concepts`

### Additional methods

A JSKOS API ONE *should* support additional URL paths to simplify requests.
These additional methods can all be mapped to the core methods, for instance
with a HTTP proxy that can rewrite URL path and HTTP GET parameters.

* `/topConcepts` - list top concepts of a scheme
* `/notation/{notation}` - get unique concept by notation
* `/notation/{notation}/broader` - get broader concepts of a unique concept
* `/notation/{notation}/narrower` - get narrower concepts of a unique concept
* `/notation/{notation}/related` - get related concepts of a unique concept
* `/suggest` - search with OpenSearch suggestions return format (requires truncation)
* ...

In addition the api endpoint *may* support more complex search with HTTP POST
requests in JSKOS and/or SPARQL to be specified later.

A later version of this specification *may* require some of this additional methods
and/or provide means to query the capabilities of an API endpoint.

## Request parameters

### Properties

All request methods *must* support the query parameter **`properties`**,
expecting a comma-separated list of property names to be included in JSKOS
response objects. 

Examples:

    ?properties=notation,prefLabel,altLabel,hiddenLabel
    ?properties=broader,narrower
    ...

The property name `label` is a alias for `prefLabel,altLabel,hiddenLabel`.

The `uri` property *should* always be included in responses, if available.

### Pagination

The requests methods `/schemes`, `/schemes/{scheme}/types`, and
`/schemes/{scheme}/concepts` can return multiple response items. The number of
items returned can be controlled with request parameter **`limit`**. The
default value *should* be 20. Paging can be controlled with request parameter
**`page`**, starting with the default value `page=1`.

Links to previous, next, first, and last pages *must* be included in the HTTP
`Link` header:

    Link: <https://example.org/jskos/schemes?page=2&limit=20>; rel="next",
          <https://example.org/jskos/schemes?page=2&limit=3>; rel="last",
          <https://example.org/jskos/schemes?limit=20>; rel="first"

The total number of items should also be sent back with the HTTP Header
`X-Total-Count`.

The special request parameter **`unique`** (set to any value but `0` or the
empty string) can be used to transform a list result into either a single item
(if total count is 1) or a HTTP 300 response:

 without `unique`          | with `unique`
-------------------------- | --------------
`[ { ... } ]`              | { ... }
`[ { ... }, { ... } ... ]` | HTTP 300


## Search parameters

### Concept search

Search via query parameters, combined as boolean AND (boolean OR is not supported). Concept search supports the following concept properties:

* **`prefLabel`**, **`altLabel`**, **`hiddenLabel`**
* **`label`** (any of `prefLabel`, `altLabel`, `hiddenLabel`)
* **`notation`**
* **`type`** (full URIs)
* **`broader`**, **`narrower`**, **`related`** (values must be full URIs)
* **`note`** (any kind of note)

Label properties and documentation properties can be localized by appending `.` and the language tag, e.g. `prefLabel.en=fox`.
    
The field name `label` can be used to refer to any of `prefLabel`, `altLabel`, and `hiddenLabel`. For instance `label=fox` can be used to search for concepts with any kind of label having the value "fox". The name can also be localized e.g. `label.en=fox`.

### Truncation

The search parameter **`truncate`** set to `truncate=right` enables
right-truncation.
 
### Normalization

When searching all strings must be Unicode normalized to not distinguish
composed and decomposed characters sequences.  The final JSON response *must*
always be normalized in NFC.

Search normalization can be enforced with parameter **`fold`** expecting
a comma-separated set with the following possible values:

* `canonical` compares strings by canonical equivalence after NFKC. For
  instance ligature `ﬃ` is equivalent to `ffi`.
* `case` compares strings after conversion to uppercase.
* `canonical,case` to apply both
* `all` applies more character folding such as removal of accents. See
  applications of <http://unicode.org/reports/tr30/> for examples and
  implementations. Language tags should be respected to fold differently
  depending on languages.

Additional folding methids *may* be supported. For instance `mark` compares
strings after NFKC and removing all characters in the categories Nonspacing
Mark (Mn), Spacing Mark (Mc) and Enclosing Mark (Me), thus ignoring most
accents and diacritics aka "accent folding". See Unicode 
[character properties](http://unicode.org/reports/tr44/#Property_Definitions)
for possible criteria.

### Examples

    /concepts?prefLabel=weisskopf&truncate=right&fold=all

    [ 
      {
        "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
        "prefLabel": {
          "de": "Weißköpfe"
        }
      }
    ]


The concept is found because

* `prefLabel` also finds `prefLabel.de`
* `fold=all` normalizes "Weißköpfe" to "WEISSKOPFE"
* `truncate=right` finds "WEISSKOPFE" if only "WEISSKOPF" was searched

