# Introduction

This document specified **JSKOS API**, an experimental JSON-API to access
controlled vocabularies, also known as knowledge organization systems, or
taxonomies, expressed in [JSKOS](http://gbv.github.io/jskos/) format.

## Synopsis

A JSKOS API consists of a base URL and a set of request endpoints to query
[concepts](#concept-endpoint), [concept schemes](#schemes-endpoint), [concept
types](#types-endpoint), and [concept mappings] with HTTP GET
requests.

## Status of this document

*The current version of JSKOS API specification is a very early draft!*

The specification is hosted in a public git repository at
<https://github.com/gbv/jskos-api>. Comments and contributions are welcome!

JSKOS API is being defined as part of project
[coli-conc](https://coli-conc.gbv.de).

## Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in RFC 2119.

# Request and response format

## Request

* HTTPS GET, HEAD, OPTIONS requests only (a later version may also support write operations)
* support CORS and optionally JSONP (`callback` parameter)

Core endpoints:

* `/concepts`
* `/schemes`
* `/types`
* `/mappings`

A JSKOS API endpoint MAY support [additional endpoints](#utility-endpoints)
which can all be implemented based on the core endpoints.

Each URL query parameters is one of

* query parameter of some kind (text, URI, numeric, geospatial...)
* [query modifier](#query-modifiers)

## Request headers

...

## Response body

All HTTP responses MUST be valid JSON.

* based on [JSKOS](http://gbv.github.io/jskos/) format
* SHOULD pretty-print JSON and gzip compression supported by default.

Possible response types include:

* [description document]
* JSKOS object or list of JSKOS objects
* [error response]

## Response headers

A JSKOS API MUST support the following HTTP response headers:

Content-Type
  : The value `application/json` or `application/json; charset=utf-8` for 
    JSON response; the value `application/javascript` or 
    `application/javascript; charset=utf-8` for JSONP response
X-Total-Count
  : Total number of results for [pagination].
Link
  : Links for [pagination].
Access-Control-Expose-Headers
  : The value `Link X-Total-Count`
Access-Control-Allow-Origin
  : The value `*` or another origin domain in response to a `Origin` request
    header.
Allow
  : A list of supported HTTP verbs (`GET, HEAD, OPTIONS`) in response to 
    OPTIONS requests or errors with status 405.

## Description Documents

A **description document** is a JSON object that describes properties and
capabilities of a JSKOS API. A description document MUST be returned for
HTTP GET requests at its base URL.

It SHOULD also be returned for HTTP OPTIONS requests at its base URL and core
methods, except when request header `Origin` is included for a CORS preflight
request.

A description document MUST contain the property `jskos-api` giving which
version of this specification the API conforms to.

Description documents at core endpoints (concepts endpoint, schemes endpoint,
types endpoint, mappings endpoint) MUST contain at least property `href` with
an absolute or relative URI reference of the described method. 

Other properties of description documents describe which optional features 
a JSKOS API supports.

A description document at base URL contains description documents for each
supported core endpoints at properties `concepts`, `schemes`, `types`, and
`mappings` respectively. The property `jskos-api` can be omitted in included
description documents as it can be derived from the base document.

**Example**

The following description document returned for HTTP OPTIONS requests at
<https://example.org/jskos/concepts> all describe the same concepts method:

```json
{ "jskos-api": "1.0.0", "href": "concepts" } 
{ "jskos-api": "1.0.0", "href": "./concepts" } 
{ "jskos-api": "1.0.0", "href": "//example.org/jskos/concepts" } 
{ "jskos-api": "1.0.0", "href": "https://example.org/jskos/concepts" } 
```

A minimal description document for this service returned at the base URL 
<https://example.org/jskos/> would be:

```json
{ 
  "jskos-api": "1.0.0", 
  "concepts": {
    "href": "concepts" 
  } 
}
```

## Error responses

[error response]: #error-responses

An error response body is a JSON document with properties `code`, `message`,
and `description`.

...


# Endpoints

* [concepts endpoint]
* [schemes endpoint]
* [types endpoint]
* [mappings endpoint]

Each endpoint supports a set of query parameters, which are combined as
boolean AND (boolean OR is not supported).

Label properties and documentation properties can be localized by appending `.`
and the language tag to the corresponding query parameter, e.g.
`prefLabel.en=fox`.

<div class="note">
Truncation of language tags has to be discussed. For instance
should `prefLabel.en` also match language tag `en-GB`?
</div>

## Concepts

[concepts endpoint]: #concepts

A **concepts endpoint** provides information about concepts in JSKOS format.

### Query parameters {.unnumbered}

* **`uri`**
* **`type`** (full URIs)
* **`prefLabel`**, **`altLabel`**, **`hiddenLabel`**
* **`label`** (any of `prefLabel`, `altLabel`, `hiddenLabel`)
* **`notation`**
* **`broader`**, **`narrower`**, **`related`** (full URIs)
* **`note`** (any kind of note)
* **`scheme`** (inScheme URI)
* **`schemeNotation`**

The field name `label` can be used to refer to any of `prefLabel`, `altLabel`, and `hiddenLabel`. For instance `label=fox` can be used to search for concepts with any kind of label having the value "fox". The name can also be localized e.g. `label.en=fox`.

## Schemes

[schemes endpoint]: #schemes

A **schemes endpoint** provides information about concept schemes in JSKOS
concept schemes format.

A schemes endpoint SHOULD at least return information about concept schemes
that are references in concepts of a corresponding [concepts endpoint].

### Query parameters {.unnumbered}

* **`uri`**
* **`type`**
* **`prefLabel`**, **`altLabel`**, **`hiddenLabel`**
* **`label`** (any of `prefLabel`, `altLabel`, `hiddenLabel`)
* **`notation`**

<div class="note">
Concept schemes in JSKOS API are expressed in same terms as concepts to provide
a unfiform access format.
</div>

## Types

[types endpoint]: #types

Concepts MAY have multiple types in addition to the implicit type "Concept"
from SKOS ontology. A **types endpoint** provides information about concept
types expressed as JSKOS concepts of their own. 

A types endpoint SHOULD at least return information about concept types that
are referenced in concepts of a corresponding concepts endpoint.

### Query parameters {.unnumbered}

Query parameters are equivalent to query parameters of [concepts endpoint].

<div class="example">
The following list contains three concept types for people, places, and
subjects.

```json
[
  {
    "uri": "http://www.w3.org/2004/02/skos/core#Concept",
    "prefLabel": { "en": "concept" },
    "narrower": [
        "http://example.org/types/people",
        "http://example.org/types/places",
        "http://example.org/types/subjects"
    ]
  },
  {
    "uri": "http://example.org/types/people",
    "prefLabel": { "en": "people" },
    "broader": ["http://www.w3.org/2004/02/skos/core#Concept"]
  },
  {
    "uri": "http://example.org/types/places",
    "prefLabel": { "en": "places" },
    "broader": ["http://www.w3.org/2004/02/skos/core#Concept"]
  },
  {
    "uri": "http://example.org/types/subjects",
    "prefLabel": { "en": "subjects" },
    "broader": ["http://www.w3.org/2004/02/skos/core#Concept"]
  }
]
```
</div>

<div class="note">
Concept types in RDF are likely modelled with RDFS ontology properties. In
JSKOS API these properties are mapped to JSKOS, and by this SKOS ontology
(e.g. `rdfs:subClassOf` ⇒  `skos:broader`, `rdfs:label` ⇒  `skos:prefLabel`),
to access concepts and concept types in a uniform format. 
</div>

## Mappings

[mappings endpoint]: #mappings

A **mappings endpoint** provides information about concepts mappings in JSKOS
mappings format.

Concepts and concept schemes referenced in concept mappings at a given mappings
endpoint SHOUDL also be accessible via a corresponding [concepts endpoint] and
[schemes endpoint], respectively.

### Query parameters {.unnumbered}

Mappings can be searched with query parameters, combined as boolean AND:

* **`uri`**
* **`type`** and **`relevance`**
* **`fromScheme`** and **`toScheme`** to select concept schemes by URI
* **`fromSchemeNotation`** and **`toSchemeNotation`** to select concept schemes by notation
* **`from`** and **`to`** to select concepts by URI
* **`fromNotation`** and **`toNotation`** to select concepts by notations
* **`fromType`** and **`toType`** to select concepts by type URI
* **`creator`**, **`publisher`**, **`contributor`**, **`source`**,
  **`provenance`**, **`dateAccepted`**, **`modified`**... for metadata
  about mappings


# Query modifiers

Some special URL query parameters control how a request is processed
and returned.

## Properties

All core endpoints MUST support the query parameter **`properties`**, expecting
a comma-separated list of property names to be included in JSKOS response
objects. 

Examples:

    ?properties=notation,prefLabel,altLabel,hiddenLabel
    ?properties=broader,narrower
    ...

The property name `label` is a alias for `prefLabel,altLabel,hiddenLabel`.

The `uri` property SHOULD always be included in responses, if available.

## Pagination

The core endpoints can return multiple response items. The number of items
returned can be controlled with request parameter **`limit`**. The default
value SHOULD be 20. Paging can be controlled with request parameter
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

 without `unique` (HTTP 200) | with `unique`
---------------------------- | --------------------
`[ { ... } ]`                | { ... } (HTTP 200)
`[ ]`                        | error with HTTP 404
`[ { ... }, { ... } ... ]`   | error with HTTP 300

## Mediated result lists

To get the list of broader, narrower, or related concepts instead of the full concept, use parameter **`list`** with possible values `broader`, `narrower`, `related`.

...

## JSKOS expansion and normalization

...

# Extensions

A JSKOS-API MAY support the following extensions and propagate their support
in [description document].

## Truncation

The search parameter **`truncate`** set to `truncate=right` enables
right-truncation for all query parameters or for fixed set of query
parameters (specified in [description document]) except URIs.

<div class="example">
The query 
`?prefLabel=A&inScheme=http://example.org/schemes/ABC&truncate=right`
at a concepts endpoint will return all concepts in concept scheme
`http://example.org/schemes/ABC` with preferred label starting with letter "A".
Such concepts in concept scheme `http://example.org/schemes/ABCDE` will not be
included.
</div>

<div class="note">
Name and value of this parameter may be changed in a later version to
also support other query extensions (regular expressions, ranking...).
</div> 

## Normalization

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

<div class="note">
Basic support of Unicode normalization is likely to become mandatory
in a later version of this specification.
</div>

<div class="example">
The following example illustrates truncation and normalization:

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
</div>
 
## Utility endpoints

A JSKOS API SHOULD support additional URL paths to simplify requests. These
additional methods can all be mapped to the core endpoints, for instance with a
HTTP proxy that can rewrite URL path and HTTP GET parameters.

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
* ...

Note that these utility URLs may problematic if notations contain `/`. 

## Restricted access

*Support of Simple Auth (token as username) & OAuth 2 for restricted access...*

## Search suggestions

*Support of OpenSearch Suggestions for typeahed (`list=suggest`)...*

## Reconciliation service

*Support of Reconciliations API...*

## Range search

*Support of `>`, `<`, `<=`, `>=` to search ranges for selected properties...*

## Geospatial search

*Support of geospatial search...*

# References

## Normative References

* Bradner, S. 1997. “RFC 2119: Key words for use in RFCs to Indicate Requirement Levels”.
  <http://tools.ietf.org/html/rfc2119>.

* Crockford, D. 2006. “RFC 6427: The application/json Media Type for JavaScript Object Notation (JSON)”.
  <http://tools.ietf.org/html/rfc4627>.

* RFC 2396: “  Uniform Resource Identifiers (URI): Generic Syntax”.
  <https://tools.ietf.org/html/rfc2396>.

* Fielding, R. 1999. “RFC 2616: Hypertext Transfer Protocol”.
  <http://tools.ietf.org/html/rfc2616>.

* van Kesteren, Anne. 2014. “Cross-Origin Resource Sharing”.
  <http://www.w3.org/TR/cors/>

* Rescorla, E. 2000. “RFC 2818: HTTP over TLS”.
  <http://tools.ietf.org/html/rfc2818>.

* Voß, J. 2015. “JSKOS data format for knowledge organization systems”.
  <https://gbv.github.io/jskos/>. 

* ...

## Informative References

* van Kesteren, A. 2014. “Cross-Origin Resource Sharing”.
  <http://www.w3.org/TR/cors/>

* Miles, A. and Bechhofer, S. 2009: “SKOS Reference”.
  W3C Recommendation. <http://www.w3.org/TR/skos-reference>

* ...

## Revision history

This is version **{VERSION}** of JSKOS API specification, last modified at
{GIT_REVISION_DATE} with revision {GIT_REVISION_HASH}.

Version numbers follow [Semantic Versioning](http://semver.org/): each number
consists of three numbers, optionally followed by `+` and a suffix:

* The major version (first number) is increased if changes require
  a modification of JSKOS API clients
* The minor version (second number) is increased if changes require
  a modification a JSKOS API servers
* The patch version (third number) is increased for backwards compatible
  fixes or extensions, such as the introduction of new optional features
* The optional suffix indicates informal changes in documentation

### Releases {.unnumbered}

Releases with functional changes are tagged with a version number and included
at <https://github.com/gbv/jskos-api/releases> with release notes.

#### 0.0.0 (unpublished) {.unnumbered}

The current version is an early draft so it has not been tagged as release yet.

### Full changelog {.unnumbered}

{GIT_CHANGES}


