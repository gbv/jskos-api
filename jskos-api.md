# Introduction

This document specified **JSKOS API**, an experimental JSON-API to access
controlled vocabularies, also known as knowledge organization systems, or
taxonomies, expressed in [JSKOS] format.

[JSKOS]: https://gbv.github.io/jskos/

## Synopsis

A JSKOS API consists of a base URL and a set of [request endpoints] to query
[concepts](#concepts), [concept schemes](#schemes), [concept types](#types),
and [concept mappings](#mappings) in [JSKOS] format.

## Status of this document

<div class="note">
The current version of JSKOS API specification is a very early draft!
</div>

JSKOS API is being defined as part of project
[coli-conc](https://coli-conc.gbv.de).

The specification is hosted in a public git repository at
<https://github.com/gbv/jskos-api>. Comments and contributions are welcome!

## Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in RFC 2119.

# Request and response format

## Basics

A JSKOS API service MUST respond to HTTP GET, HEAD, and OPTIONS requests. The
response body MUST be valid JSON or empty. A JSON response body SHOULD be
pretty-printed.  A JSON response MUST be one of

* a [service description] or endpoint description
* a [JSKOS](http://gbv.github.io/jskos/) object (concept, concept scheme, or
  mapping) or an array of JSKOS objects,
* an [error response]

A JSKOS API service SHOULD provide a base URL to request a [service
description], a set of [request endpoints], and optional [utility endpoints]. 

Endpoints MUST be queried via with HTTP GET with optional URL query fields.
Each URL query field is one of

* query parameter (depending on the request endpoint)
* [query modifier](#query-modifiers)

## URL query fields

## HTTP headers

### Request headers {.unnumbered}

The following HTTP request headers SHOULD be included in requests to JSKOS API:

Accept
  : The value `application/json`
User-Agent
  : An appropriate client name and version number
Accept-Language
  : *See <https://github.com/gbv/jskos-api/issues/2> for discussion!*

A OPTIONS preflight request for Cross-Origin Resource Sharing (CORS) MUST
include the cross-origin request headers:

Origin
  : Where the cross-origin request originates from
Access-Control-Request-Method 
  : The HTTP verb of the actual request (GET or OPTIONS)

### Response headers {.unnumbered}

A JSKOS API MUST support the following HTTP response headers:

Content-Type
  : The value `application/json` or `application/json; charset=utf-8` for 
    JSON response; the value `application/javascript` or 
    `application/javascript; charset=utf-8` for JSONP response
X-Total-Count
  : Total number of results if the response body is a JSON array
    (see [pagination] for details).
Link
  : Links for [pagination] if the response body is a JSON array.
Access-Control-Expose-Headers
  : The value `Link X-Total-Count`
Access-Control-Allow-Origin
  : The value `*` or another origin domain in response to a `Origin` request
    header.
Allow
  : A list of supported HTTP verbs (`GET, HEAD, OPTIONS`) in response to 
    a OPTIONS request or with an [error response] with status code 405.

## Service description

[service description]: #service-description

A JSKOS API service SHOULD provide a **base URL**. A service description MUST
be returned at the base URL for HTTP GET requests.  and it SHOULD be returned for
HTTP OPTIONS requests. A JSKOS API service SHOULD return a service description
for HTTP OPTIONS requests at [request endpoints].

A **service description** is a JSON object that describes properties and
capabilities of a JSKOS API service with the following fields:

* **`jskosapi`** (mandatory) which version of this specification the service conforms to
* **`title`** (optional) human-readable name of the service

A service description returned at base URL SHOULD include endpoint descriptions
with the fields `concepts`, `schemes`, `types`, and `mappings` if available.

An **endpoint description** is a JSON object with the following fields:

* **`href`** (mandatory) absolute or relative URI reference of the described method. 
* **`title`** (optional) human-readable name of the endpoint

<div class="note">
Later versions of this specification will include support of [extensions] and
optional features in service descriptions and endpoint descriptions.
</div>

<div class="example">
The following service description returned at a base URL
<https://example.org/jskos/> includes an endpoint description of a
[concepts endpoint] at <https://example.org/jskos/concepts> and 
[types endpoint] at <https://example.org/jskos/types>. 

```json
{ 
   "jskos-api": "1.0.0", 
   "title": "Funny Terminology Server",
   "concepts": { 
     "href": "./concepts",
     "title": "Funny Concepts"
   },
   "types": { 
     "href": "./types",
     "title": "Funny Concept Types"
   }

} 
```

The concept URL could also be specified as `./concepts`,
`//example.org/jskos/concepts`, or `https://example.org/jskos/concepts`.
An HTTP OPTIONS request at this URL returns at least

```json
{ 
   "jskos-api": "1.0.0", 
   "title": "Funny Terminology Server",
   "concepts": { 
     "href": "./concepts",
     "title": "Funny Concepts"
   }
}
```

</div>

## Error responses

[error response]: #error-responses

Error responses with HTTP status code 4xx (client error) or 5xx (server error)
SHOULD be returned with a JSON object response body having the following
fields:

* **`code`** (REQUIRED) the HTTP status error code.
* **`error`** (REQUIRED) a custom error code. 
  Allowed characters include `a-z`, `0-9` and underscore (`_`).
* **`message`** (OPTIONAL) a human-readable error message.
  Intended to be shown to an end user.
* **`error_description`** (OPTIONAL) a human-readable error description.
  Intended for a developer, not an end user.
* **`error_uri`** (OPTIONAL) a URL of a human-readable web page with
  information about the error.

The response headers SHOULD include a `Content-Language` header to indicate the
language of human-readable message and description. The `Accept-Language`
request header SHOULD be used to select a language if multiple languages are
supported.

# Request endpoints

[request endpoints]: #request-endpoints

A JSKOS API service SHOULD provide URLs as

* [concepts endpoint]
* [schemes endpoint]
* [types endpoint]
* [mappings endpoint]

Each endpoint supports a set of query parameters, which are combined as boolean
AND. 

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
that are referenced in concepts of a corresponding [concepts endpoint].

### Query parameters {.unnumbered}

* **`uri`**
* **`type`**
* **`prefLabel`**, **`altLabel`**, **`hiddenLabel`**
* **`label`** (any of `prefLabel`, `altLabel`, `hiddenLabel`)
* **`notation`**

<div class="note">
Concept schemes in JSKOS API are expressed in same terms as concepts to provide
a uniform access format.
</div>

## Types

[types endpoint]: #types

Concepts MAY have multiple types in addition to the implicit type "Concept"
from SKOS ontology. A **types endpoint** provides information about concept
types expressed as JSKOS concepts of their own. 

A types endpoint SHOULD at least return information about concept types that
are referenced in concepts of a corresponding concepts endpoint. The default
type `http://www.w3.org/2004/02/skos/core#Concept` MAY be omitted.

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

The following URL query fields control how a request is processed and returned.
They can be used at all [request endpoints] and [utility endpoints] unless
noted otherwise.

* [`properties`](#properties)
* [`limit`, `page`, `unique`](#pagination)
* [`callback`](#jsonp)

JSKOS properties with datatype language map (`prefLabel`, `altLabel`,
`scopeNote`...) can further be qualified by [language qualifiers].

Additional query modifiers MAY be supported by [extensions].

## Language qualifiers
[language qualifiers]: #language-qualifiers

Label properties and documentation properties by default refer to any language
(JSKOS language range "`-`"). The query can be limited to selected languages
by appending a dot ("`.`") and a language tag or JSKOS language range to the
query parameter.

<div class="examples"
```
prefLabel=fox       # ... (any language)
prefLabel.en=fox    # en
prefLabel.en-=fox   # en-GB, en-US, ..
```
</div>

<div class="note">
Should support of language ranges better be an optional extension?
See <https://github.com/gbv/jskos-api/issues/10> for discussion.
</div>

## Properties

The **`properties`** query field MUST be supported to allow selection of object
properties in JSKOS response objects. The value of this field is a
comma-separated list of property names. Unknown property names SHOULD be
ignored. The `uri` property SHOULD always be included in response objects, if
available, no matter if included in the properties field or not.

The property name `label` is a alias for `prefLabel,altLabel,hiddenLabel`.

The empty string MUST be interpreted equal to no `properties` field.  A
[service description] SHOULD include the set of default properties returned if
no `properties` field is given. 

<div class="example">

    ?properties=notation,prefLabel,altLabel,hiddenLabel
    ?properties=broader,narrower
    ...
</div>

<div class="note">
See also <https://github.com/gbv/jskos-api/issues/4>
for discussion of JSKOS expansion with this query field.
</div>

## Pagination
[pagination]: #pagination

The core endpoints can return multiple response items. The number of items
returned can be controlled with request parameter **`limit`**. The default
value SHOULD be 20. Paging can be controlled with request parameter
**`page`**, starting with the default value `page=1`.

Links to previous, next, first, and last pages MUST be included in the HTTP
`Link` header:

    Link: <https://example.org/v1/schemes?page=2&limit=20>; rel="next",
          <https://example.org/v1/schemes?page=2&limit=3>; rel="last",
          <https://example.org/v1/schemes?limit=20>; rel="first"

The total number of items SHOULD also be sent back with the HTTP Header
`X-Total-Count`.

## unique

The special request parameter **`unique`** (set to any value but `0` or the
empty string) can be used to transform a list result into either a single item
(if total count is 1) or a HTTP 300 response:

 without `unique` (HTTP 200) | with `unique`
---------------------------- | --------------------
`[ { ... } ]`                | { ... } (HTTP 200)
`[ ]`                        | error with HTTP 404
`[ { ... }, { ... } ... ]`   | error with HTTP 300

## JSONP

The **`callback`** parameter makes the JSON response body wrapped in a callback
function. 

<div class="note">
Clients SHOULD prefer CORS over JSONP if needed.

## list

To get the list of broader, narrower, or related concepts instead of the full
concept, use parameter **`list`** with possible values `broader`, `narrower`,
`related`.

<div class="note">
This parameter might be extended.
See <https://github.com/gbv/jskos-api/issues/6> for discussion.
</div>

## JSKOS normalization

A JSKOS-API service MUST return normalized JSKOS.
See <https://github.com/gbv/jskos/issues/16> and
<https://github.com/gbv/jskos-api/issues/5> 
for discussion.

## JSKOS expansion

See <https://github.com/gbv/jskos-api/issues/18> for discussion.

# Extensions
[extensions]: #extensions

A JSKOS-API MAY support the following extensions and propagate their support
in [service description].

## Mediated result lists

To get the list of broader, narrower, or related concepts instead of the full
concept, use query modification parameter **`list`** with possible values
`broader`, `narrower`, `related`, `ancestors`.

See <https://github.com/gbv/jskos-api/issues/17> for discussion.

## Truncation

[truncation]: #truncation

The search parameter **`truncate`** set to `truncate=right` enables
right-truncation for all query parameters or for fixed set of query
parameters (specified in [service description]) except URIs.

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
See <https://github.com/gbv/jskos-api/issues/11> for discussion.
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
See <https://github.com/gbv/jskos-api/issues/7> for discussion.
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

[utility endpoints]: #utility-endpoints

A JSKOS API SHOULD support additional URL paths to simplify requests. These
additional methods can all be mapped to the core [request endpoints], for
instance with a HTTP proxy that can rewrite URL path and HTTP GET parameters.

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

<div class="note">
Support of Simple Auth (token as username) & OAuth 2 for restricted access...
</div>

## Search suggestions

<div class="note">
Support of OpenSearch Suggestions for typeahed (`list=suggest`)...
See <https://github.com/gbv/jskos-api/issues/12> for discussion.
</div>

## Reconciliation service

<div class="note">
Support of Reconciliations API...
</div>

## Range search

<div class="note">
Support of `>`, `<`, `<=`, `>=` to search ranges for selected properties...
</div>

## Geospatial search

<div class="note">
Support of geospatial search...
</div>

## Write operations

<div class="note">
Support of write-operations with HTTP POST, PUT, DELETE...
</div>

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
  a modification of JSKOS API client
* The minor version (second number) is increased if changes require
  a modification a JSKOS API services
* The patch version (third number) is increased for backwards compatible
  fixes or extensions, such as the introduction of new optional features
* The optional suffix indicates informal changes in documentation

<div class="note">
As long as major version is 0, changes requiring modification of JSKOS API
clients may also be indicated by increasing the minor version number!
</div>

### Releases {.unnumbered}

Releases with functional changes are tagged with a version number and included
at <https://github.com/gbv/jskos-api/releases> with release notes.

#### 0.1.0 {.unnumbered}

Semi-stable pre-release.

### Full changelog {.unnumbered}

{GIT_CHANGES}


