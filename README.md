![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DK Hostmaster RESTful WHOIS Service Specification

![GitHub Workflow build status badge markdownlint](https://github.com/DK-Hostmaster/whois-rest-service-specification/workflows/Markdownlint%20Workflow/badge.svg)

2019-11-19
Revision: 2.1

**PLEASE NOTE THAT THIS SERVICE IS CURRENTLY IN BETA AND CHANGES MIGHT BE IMPLEMENTED WHICH BREAK BACKWARDS COMPATIBILITY**

# Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3, 4" indent="  " autolink="true" -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
  - [Localization](#localization)
  - [Media-type / Format](#media-type--format)
  - [Encoding](#encoding)
  - [Rate Limiting](#rate-limiting)
  - [Security](#security)
- [Service](#service)
  - [domain](#domain)
    - [API](#api)
    - [Example](#example)
  - [domain list](#domain-list)
    - [API](#api-1)
    - [Example](#example-1)
  - [handle](#handle)
    - [API](#api-2)
    - [Example](#example-2)
  - [host](#host)
    - [API](#api-3)
    - [Example](#example-3)
  - [query](#query)
    - [API](#api-4)
    - [Domain Example](#domain-example)
    - [Host Example](#host-example)
    - [Handle Example](#handle-example)
- [Resources](#resources)
  - [Mailing list](#mailing-list)
  - [Issue Reporting](#issue-reporting)
  - [Additional Information](#additional-information)
- [Appendices](#appendices)
  - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
# Introduction

This document describes and specifies the a RESTful alternative to the WHOIS implementation offered by DK Hostmaster A/S. It is primarily aimed at a technical audience and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS RESTful service is optimized for structured querying in contrast to it's text-based origin implementing the [WHOIS service](https://github.com/DK-Hostmaster/whois-service-specification) responding on port 43,

<a id="about-this-document"></a>
# About this Document

This specification describes version 2 (2.X.X) of the DK Hostmaster WHOIS RESTful service implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS RESTful service implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/whois-rest-service-specification/blob/master/README.md), using the gitprint service.

<a id="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
## Document History

- 2.1 2019-11-19
  - Updated section on rate limiting based on changes to production environment

- 2.0 2019-04-30
  - Major update based on the changes with major release 2.0.0 of the WHOIS RESTful service
  - Documenting removal of public information on non-registrant users for handle (users) and domain name inquiries
  - Documenting removal of name server contacts for host (name server) inquiries

- 1.1 2017-05-01
  - Correction of typos and minor cleanups, thanks to the contributors for the PRs

- 1.0 2016-11-08
  - Initial revision

<a id="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

<a id="features"></a>
# Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Handle inquiry
- Support for multiple encodings (see: [Encoding](#encoding) below)
- Support for both IPv4 and IPv6

<a id="implementation-limitations"></a>
# Implementation Limitations

<a id="localization"></a>
## Localization

In general the service is not localized and all WHOIS information is provided in English.

<a id="media-type--format"></a>
## Media-type / Format

The service only supports **JSON**.

<a id="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8
- Punycode [RFC:3492][RFC:3492]
- ASCII
- URI encoding [RFC:3986][RFC:3986]

<a id="rate-limiting"></a>
## Rate Limiting

We only allow one request per second per source IP.

If rate is higher that acceptable, we return: `503 Service Temporarily Unavailable`

We reserve the right to adjust the rate limiting in order to provide a high quality of service.

<a id="security"></a>
## Security

The service is only available under **TLS 1.2**

<a id="service"></a>
# Service

The service is available at:

- `https://whois-api.dk-hostmaster.dk`

The service offers four APIs, one specialized for each entity type:

- domain (domain name)
- host (hostname/name server)
- handle (userid)

And a generic API:

- query

Which relays to the specific service APIs for the mapped entity being one of the ones listed above.

The service requires that the accept header is specified to be `application/json`, all of the below examples demonstrates this using the command line utilities `curl` or `httpie`.

As described under Implementation Limitations, the service only supports **JSON**. This has to be specified using the **HTTP** header: `Accept`

If the header is unspecified, not specified correctly or specified to an unsupported format, the service will error with HTTP status code: `415`

```bash
$ curl https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK
"Unsupported Media Type"
```

Correct specification using `curl` should be as follows:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK
{"attention":null,"city":"København S","countryregionid":"DK","message":"OK","mobilephone":null,"name":"DK HOSTMASTER A\/S","phone":null,"query_userid":"DKHM1-DK","status":200,"street1":"Ørestads Boulevard 108, 11.","street2":null,"street3":null,"telefax":null,"userid":"DKHM1-DK","useridtype":"V","validregistrant":"1","zipcode":"2300"}
```

And for `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'

HTTP/1.1 200 OK
Cache-Control: max-age=1, no-cache
Connection: keep-alive
Content-Encoding: gzip
Content-Type: application/json;charset=UTF-8
Date: Mon, 22 Apr 2019 11:25:24 GMT
Server: nginx
Strict-Transport-Security: max-age=15768000
Transfer-Encoding: chunked
Vary: Accept-Encoding
Vary: Origin

{
    "attention": null,
    "city": "København S",
    "countryregionid": "DK",
    "message": "OK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "status": 200,
    "street1": "Ørestads Boulevard 108, 11.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "2300"
}
```

<a id="domain"></a>
## domain

This service returns data on a given domain name.

As of version 2.X.X of the service, admin/proxy information is no longer part of the response data, same goes for name server administrators/zone contact handles.

<a id="api"></a>
### API

`https://whois-api.dk-hostmaster.dk/domain/{domainname}`

The service returns `200` if it can find a relevant object together with the public data.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/domain/eksempel.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/eksempel.dk | jq
```

```json
{
  "createddate": "1999/05/17",
  "dnssec": "J",
  "domain": "eksempel.dk",
  "domain_encoded": "eksempel.dk",
  "domain_type": "V",
  "message": "OK",
  "nameservers": {
    "auth01.ns.dk-hostmaster.dk": {
      "domain": "eksempel.dk",
      "domain_encoded": "eksempel.dk",
      "hostname": "auth01.ns.dk-hostmaster.dk",
      "hostname_encoded": "auth01.ns.dk-hostmaster.dk"
    },
    "auth02.ns.dk-hostmaster.dk": {
      "domain": "eksempel.dk",
      "domain_encoded": "eksempel.dk",
      "hostname": "auth02.ns.dk-hostmaster.dk",
      "hostname_encoded": "auth02.ns.dk-hostmaster.dk"
    }
  },
  "paiduntildate": "2022/06/30",
  "periodqty": "5",
  "public_deletedate": null,
  "public_domain_status": "A",
  "registrant": {
    "attention": null,
    "city": "København S",
    "countryregionid": "DK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "street1": "Ørestads Boulevard 108, 11.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "2300"
  },
  "registrant_userid": "DKHM1-DK",
  "status": 200
}
```

<a id="domain-list"></a>
## domain list

<a id="api-1"></a>
### API

`https://whois-api.dk-hostmaster.dk/domain/list/handle/{userid}/role/{role}`

The service returns `200` if it can find a relevant object holding the relevant role. Supported role types values is: `registrant` only.

As of version 2.X.X of the service, admin/proxy information is no longer supported.

<a id="example-1"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/domain/list/handle/DKHM1-DK/role/registrant Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/list/handle/DKHM1-DK/role/registrant | jq
```

```json
{
  "attention": null,
  "city": "København S",
  "countryregionid": "DK",
  "domains": [
    {
      "domain": "dk-hostmaster.dk",
      "domain_encoded": "dk-hostmaster.dk"
    },
    {
      "domain": "æøåöäüé.dk",
      "domain_encoded": "xn--4cabco7dk5a.dk"
    }
  ],
  "message": "OK",
  "mobilephone": null,
  "name": "DK HOSTMASTER A/S",
  "phone": null,
  "query_userid": "DKHM1-DK",
  "role": "registrant",
  "status": 200,
  "street1": "Ørestads Boulevard 108, 11.",
  "street2": null,
  "street3": null,
  "telefax": null,
  "userid": "DKHM1-DK",
  "useridtype": "V",
  "validregistrant": "1",
  "zipcode": "2300"
}
```

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="handle"></a>
## handle

This service returns data on a given handle/user-id. The service only returns data on active users with the `registrant` role.

As of version 2.X.X of the service, admin/proxy and name server administrator/zone contact information is no longer available via this service.

<a id="api-2"></a>
### API

`https://whois-api.dk-hostmaster.dk/handle/{userid}`

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example-2"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK | jq
```

```json
{
  "attention": null,
  "city": "København S",
  "countryregionid": "DK",
  "message": "OK",
  "mobilephone": null,
  "name": "DK HOSTMASTER A/S",
  "phone": null,
  "query_userid": "DKHM1-DK",
  "status": 200,
  "street1": "Ørestads Boulevard 108, 11.",
  "street2": null,
  "street3": null,
  "telefax": null,
  "userid": "DKHM1-DK",
  "useridtype": "V",
  "validregistrant": "1",
  "zipcode": "2300"
}
```

<a id="host"></a>
## host

This service returns data on a given hostname/name server.

As of version 2.X.X of the service, name server administrators/zone contact handle information is no longer part of the response data.

<a id="api-3"></a>
### API

`https://whois-api.dk-hostmaster.dk/host/{hostname}`

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example-3"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk | jq
```

```json
{
  "glue_spooled": "J",
  "hostname": "auth01.ns.dk-hostmaster.dk",
  "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
  "message": "OK",
  "nameserver_status": "A",
  "status": 200
}
```

<a id="query"></a>
## query

This service acts as a central entry points it relays to the above services:

<a id="api-4"></a>
### API

`https://whois-api.dk-hostmaster.dk/#query`

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="domain-example"></a>
### Domain Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/eksempel.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/eksempel.dk | jq
```

```json
{
  "createddate": "1999/05/17",
  "dnssec": "J",
  "domain": "eksempel.dk",
  "domain_encoded": "eksempel.dk",
  "domain_type": "V",
  "message": "OK",
  "nameservers": {
    "auth01.ns.dk-hostmaster.dk": {
      "domain": "eksempel.dk",
      "domain_encoded": "eksempel.dk",
      "hostname": "auth01.ns.dk-hostmaster.dk",
      "hostname_encoded": "auth01.ns.dk-hostmaster.dk"
    },
    "auth02.ns.dk-hostmaster.dk": {
      "domain": "eksempel.dk",
      "domain_encoded": "eksempel.dk",
      "hostname": "auth02.ns.dk-hostmaster.dk",
      "hostname_encoded": "auth02.ns.dk-hostmaster.dk"
    }
  },
  "paiduntildate": "2022/06/30",
  "periodqty": "5",
  "public_deletedate": null,
  "public_domain_status": "A",
  "registrant": {
    "attention": null,
    "city": "København S",
    "countryregionid": "DK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "street1": "Ørestads Boulevard 108, 11.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "2300"
  },
  "registrant_userid": "DKHM1-DK",
  "status": 200
}
```

<a id="host-example"></a>
### Host Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/auth01.ns.dk-hostmaster.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/auth01.ns.dk-hostmaster.dk | jq
```

```json
{
  "glue_spooled": "J",
  "hostname": "auth01.ns.dk-hostmaster.dk",
  "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
  "message": "OK",
  "nameserver_status": "A",
  "status": 200
}
```

<a id="handle-example"></a>
### Handle Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/DKHM1-DK Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/DKHM1-DK | jq
```

```json
{
  "attention": null,
  "city": "København S",
  "countryregionid": "DK",
  "message": "OK",
  "mobilephone": null,
  "name": "DK HOSTMASTER A/S",
  "phone": null,
  "query_userid": "DKHM1-DK",
  "status": 200,
  "street1": "Ørestads Boulevard 108, 11.",
  "street2": null,
  "street3": null,
  "telefax": null,
  "userid": "DKHM1-DK",
  "useridtype": "V",
  "validregistrant": "1",
  "zipcode": "2300"
}
```

<a id="resources"></a>
# Resources

Resources for DK Hostmaster WHOIS support can be found below.

<a id="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster RESTful WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

- `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a id="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

- `info@dk-hostmaster.dk`

<a id="additional-information"></a>
## Additional Information

The DK Hostmaster website service page

- `https://www.dk-hostmaster.dk/en/whois`

<a id="appendices"></a>
# Appendices

<a id="http-status-codes"></a>
## HTTP Status Codes

The services hold their own table of return codes, this is just a curated list to collect all possible return codes, please refer to the specific services also.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |
| `500` | Internal server error |

[RFC:3492]: https://tools.ietf.org/html/rfc3492
[RFC:3912]: https://tools.ietf.org/html/rfc3912
[RFC:3986]: https://tools.ietf.org/html/rfc3986
