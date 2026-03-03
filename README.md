# JSKOS-API

The **[JSKOS data format for Knowledge Organization Systems](https://gbv.github.io/jskos/)** has been established to share data from and about classifications, thesauri, authority files, and knowledge graphs in a uniform way. Its JSON format is defined by a detailled specification and JSKOS data can be [validated](https://gbv.github.io/jskos/#validation) with JSON Schemas, [tools](https://www.npmjs.com/package/jskos-validate) and services.

**JSKOS-API**, an HTTP API to read and write JSKOS data, lacks a concise specification. The reference implementation [JSKOS Server](https://github.com/gbv/jskos-server) contains many features not required by use cases such as just making data accesible. Other implementations provide a reduced set of features and JSKOS API clients expect different sets of features. So a JSKOS API specification is needed to improve interoperability, and to help implementers of JSKOS API servers and clients.

*This document collects some incomplete notes that will hopefully result in a formal specification.* 

## Table of Contents

- [Endpoints](#endpoints)
  - [status](#status)
- [Authorized access](#authorized-access)
- [Paginated results](#paginated-results)
- [HTTP headers](#http-headers)
- [Response codes](#response-codes)
- [Error response](#error-response)
- [References](#references)

## Endpoints

A server MUST provide a [status](#status) endpoint at an URL that ends with `/status`.

### status

Returns a JSON object listing capabilities and endpoints of the server.

*Example*

~~~json
{
  "schemes": "http://example.org/api/voc",
  "concepts": "http://example.org/api/concepts"
  "top": "http://example.org/api/top"
}
~~~

### schemes

Returns an array of [JSKOS Concept Schemes](https://gbv.github.io/jskos/#concept-scheme). The list can be filtered with the following query parameters:

- `uri` - URIs of concept schemes separated by `|`. The list is filtered to concept schemes having one of the URIs in field `uri` or `identifier`
- ...

Servers MUST support query parameter `uri`. Support of other parameters is OPTIONAL.

*See [corresponding jskos-server endpoint](https://github.com/gbv/jskos-server?tab=readme-ov-file#get-voc)*

### concepts

Returns an array of [JSKOS Concepts]. The list is controlled by the following query parameters:

- `uri` - URIs of concepts separated by `|`. The list is filtered to concepts having one of the URIs in field `uri` or `identifier`.
- `voc` - URIs of concept schemes separated by `|`. The list is filtered to concepts having one of the URIs in field `uri` or `identifier` of an element of field `inScheme` of the concept.
- ...

Servers MUST support query parameters `uri` and `voc`. Support of other parameters is OPTIONAL.

Servers MUST return an empty list of none of these query parameters are given.

*See [corresponding jskos-server endpoint](https://github.com/gbv/jskos-server?tab=readme-ov-file#put-concepts)*

### narrower

...

### ancestors

...

### search

...

### suggest

...

### data

...

*See [corresponding DANTE endpoint](https://api.dante.gbv.de/vokabular_methoden/)

## Authorized access

A server MAY require authorization for selected requests. This is done by including request header `Authorization` with value `Bearer` followed by a space and an access token. The form, exchange and validation of access tokens is not part of this specification but JSON Web Tokens (JWT) are RECOMMENDED.

## Paginated results

A paginated result is a slice of a larger, total result array. Paginated results are controlled by query parameters `limit` (length of the result array) and `offset` (index of first element of result array), both non-negative integers, and by optional internal limits of a server. The default value of `offset` is `0`.

- Servers MUST return a paginated result when the value of request paramater `limit` is smaller then the size of the total result set. For instance `limit=1` MUST result in a response array of length zero or one and `limit=0` MUST always return an empty array.

- Servers MAY return a paginated result when the size of the total result set exeeds some internal limit, defined by the server.

- Servers SHOULD include response header `X-Total-Count` with the size of the total result set.

- Servers SHOULD include response header `Link` as defined in [RFC 8288](https://datatracker.ietf.org/doc/html/rfc8288) with link relation types `next`, `previous`, `first`, and `last` linking to other paginated results of the same request with different value of query parameter `offset`

Servers MAY choose to cut the retrievable result set at some internal limit. In this case the response MUST include value `null` as last element of response array if the response array is smaller than `limit` and the total result set is larger than the response array plus `offset`.

*Example: server can only return 10 records at the same time and retrieveable results are cut after 100 records. Result to a query with total result size 120 should include header `X-Tital-Count: 120`. Query with offset=99 will then return an array with last retrieveable result, followed by `null`.*

## HTTP headers

Clients and servers SHOULD sent the HTTP request and response headers, respectively.

*Note: Just include (i) Content-Type (ii) CORS for access with browsers (iii) `X-Total-Count` and `Link` for pagination*

### Request headers

**Content-Type**: set to `application/json` or `application/json; charset=utf-8` for POST, PUT, and PATCH requests

A OPTIONS preflight request for Cross-Origin Resource Sharing (CORS) MUST include the cross-origin request headers:

**Origin**: where the cross-origin request originates from 

**Access-Control-Request-Method**: the HTTP verb of the actual request (GET, POST, PUT, or PATCH)

**Access-Control-Request-Headers**: the value `Authorization` for authorized access

### Response headers

**Content-Type**: the value `application/json` or `application/json; charset=utf-8`

**Access-Control-Allow-Origin**: the value `*` or another origin domain in response to a `Origin` request header

**Access-Control-Allow-Headers**: MUST include the value `Content-Type`. MUST also include the value `Authorization` for authorized access

**X-Total-Count**: size of a total result sets, in particular on paginated results

**Link**: links to other pages in paginated results

## Response codes

- GET and PATCH requests MUST return code 200 on success, or error response otherwise
- POST requests MUST return code 201 on success, or error response otherwise
- DELETE requests MUST return code 204 on success, or error response otherwise

## Error response

Servers MUST sent appropriate response code on errors:

- 404 unknonw endpoint 
- 403 when authorized access is required or it was not granted
- 500 internal errors not caused by the client request

Error response SHOULD include a JSON response body with a JSON object including field `message` containing a human-readable error message.

*Note: empty result lists are no errors!*

## References

### Normative References

- JSKOS <https://gbv.github.io/jskos/>
- Web Linking. RFC 8288 <https://datatracker.ietf.org/doc/html/rfc8288>

### Implementations

- jskos-server (server)
- KonSys (server)
- ICONCLASS (server)
- cocoda-sdk (client)
- TS4NFDI API Gateway (client)

