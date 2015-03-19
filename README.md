# JSKOS API

This document describes an experimental JSON-API to access controlled
vocabularies in [JSKOS](http://gbv.github.io/jskos/) format.

The specification is hosted in a public git repository at
<https://github.com/gbv/jskos-api>. The repository includes a textual
description (this document) and a [Swagger](http://swagger.io/) draft
(`jskos-api.yaml`). Comments and contributions are welcome!

## Synopsis

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

* `/concepts`
* `/schemes`
* `/types`
* `/mappings`

### Utility methods

A JSKOS API *should* support additional URL paths to simplify requests.
These additional methods can all be mapped to the core methods, for instance
with a HTTP proxy that can rewrite URL path and HTTP GET parameters.

* `/schemes/{scheme}`
  = `/schemes?notation={scheme}`
* `/schemes/{scheme}/concepts`
  = `/concepts?inScheme.notation={scheme}`
* `/schemes/{scheme}/topConcepts`
  = `/concepts?topConceptOf.notation={scheme}`
* `/schemes/{scheme}/types`
  = `/types?inScheme.notation={scheme}`
* `/concepts/{notation}`
  = `/concepts?notation={notation}`
* `/concepts/{notation}/narrower`
  = `/concepts?notation={notation}&list=narrower`
* `/concepts/{notation}/broader`
  = `/concepts?notation={notation}&list=broader`
* `/concepts/{notation}/related`
  = `/concepts?notation={notation}&list=related`
* `/schemes/{scheme}/concepts/{notation}`
  = `/concepts?inScheme.notation={scheme}`
* `/schemes/{scheme}/concepts/{notation}/narrower`
  = `/concepts?inScheme.notation={scheme}&notation={notation}&list=narrower`
* `/schemes/{scheme}/concepts/{notation}/broader`
  = `/concepts?inScheme.notation={scheme}&notation={notation}&list=broader`
* `/schemes/{scheme}/concepts/{notation}/related`
  = `/concepts?inScheme.notation={scheme}&notation={notation}&list=broader`

Note that these utility URLs may problematic if notations contain `/`. 

In addition (TODO):

* `/suggest` - search with OpenSearch suggestions return format (requires truncation) = `list=suggest`
* ...

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

    Link: <https://example.org/v1/schemes?page=2&limit=20>; rel="next",
          <https://example.org/v1/schemes?page=2&limit=3>; rel="last",
          <https://example.org/v1/schemes?limit=20>; rel="first"

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

### Schemes

...

### Concepts

Search via query parameters, combined as boolean AND (boolean OR is not supported). Concept search supports the following concept properties:

* **`uri`**
* **`prefLabel`**, **`altLabel`**, **`hiddenLabel`**
* **`label`** (any of `prefLabel`, `altLabel`, `hiddenLabel`)
* **`notation`**
* **`type`** (full URIs)
* **`broader`**, **`narrower`**, **`related`** (values must be full URIs)
* **`note`** (any kind of note)
* **`inScheme`** (URI)
* **`inScheme.notation`**

Label properties and documentation properties can be localized by appending `.` and the language tag, e.g. `prefLabel.en=fox`.
    
The field name `label` can be used to refer to any of `prefLabel`, `altLabel`, and `hiddenLabel`. For instance `label=fox` can be used to search for concepts with any kind of label having the value "fox". The name can also be localized e.g. `label.en=fox`.

To get the list of broader, narrower, or related concepts instead of the full concept, use parameter **`list`** with possible values `broader`, `narrower`, `related`.

### Types

The `/types` endpoint returns a list of concept types, each represented as JSKOS Concept. The default type `skos:Concept` may be omitted. Search parameters are equivalent to concept search parameters except `type` is not allowed.

### Mappings

...

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

