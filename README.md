# DK Hostmaster RESTful WHOIS Service Specification

2016/11/08
Revision: 1.0

**PLEASE NOTE THAT THIS SERVICE IS CURRENTLY IN BETA AND CHANGES MIGHT BE IMPLEMENTED WHICH BREAK BACKWARDS COMPATIBILITY**

# Table of Contents

<!-- MarkdownTOC depth=3 -->

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
        - [Eksempel](#eksempel)
    - [host](#host)
        - [API](#api-3)
        - [Example](#example-2)
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

<a name="introduction"></a>
# Introduction

This document describes and specifies the a RESTful alternative to the WHOIS implementation offered by DK Hostmaster A/S. It is primarily aimed at a technical audience and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS RESTful service is optimized for structured querying in contrast to it's text-based origin implementing the [WHOIS service](https://github.com/DK-Hostmaster/whois-service-specification) responding on port 43,

<a name="about-this-document"></a>
# About this Document

This specification describes version 1 (1.0.x) of the DK Hostmaster WHOIS RESTful service implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS RESTful service implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/whois-rest-service-specification/blob/master/README.md), using the gitprint service.

<a name="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a name="document-history"></a>
## Document History

* 1.0 2016-11-08
  * Initial revision

<a name="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

<a name="features"></a>
# Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Handle inquiry
- Support for multiple encodings (see: [Encoding](#encoding) below)
- Support for both IPv6 and IPv6

<a name="implementation-limitations"></a>
# Implementation Limitations

<a name="localization"></a>
## Localization

In general the service is not localized and all WHOIS information is provided in English. 

<a name="media-type--format"></a>
## Media-type / Format

The service only supports **JSON**.

<a name="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8
- Punycode [RFC:3492][RFC:3492]
- ASCII
- URI encoding [RFC:3986][RFC:3986]

<a name="rate-limiting"></a>
## Rate Limiting

We only allow a certain number of requests per minute. We reserve the right to adjust the rate limit in order to provide a high quality of service. 

Current limit is set to 1 request per second.

In addition the service only allow 1 TCP-connection per. (IPv4)/24.

Meaning that `192.0.2.41` and `192.0.2.52` can not have simultanous connections, but `192.0.2.41` and `192.0.3.52` can.

<a name="security"></a>
## Security

The service is only available under **TLS 1.2**

<a name="service"></a>
# Service

The service is available at:

- `https://whois-api.dk-hostmaster.dk`

The service offers four APIs, one specialized for each entity type:

- domain (domainname)
- host (hostname/nameserver)
- handle (userid)

And a generic API:

- query

Which relays to the specific service APIs for the mapped entity being one of the ones listed above.

The service requires that the accept header is specified to be `application/json`, all of the below examples demonstrates this using the commandline utilities `curl` or `httpie`.

As described under Implementation Limitations, the service only supports **JSON**. This has to be specified using the **HTTP** header: `Accept`

If the header is unspecified, not specified correctly or specified to an unsupported format, the service will error with HTTP status code: `415`

```bash
$ curl https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK 
"Unsupported Media Type"
```

Correct specification using `curl` should be as follows:

```bash
> curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK
{"attention":null,"city":"København V","countryregionid":"DK","message":"OK","mobilephone":null,"name":"DK HOSTMASTER A\/S","phone":null,"query_userid":"DKHM1-DK","status":200,"street1":"Kalvebod Brygge 45, 3.","street2":null,"street3":null,"telefax":null,"userid":"DKHM1-DK","useridtype":"V","validregistrant":"1","zipcode":"1560"}
```

And for `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'

HTTP/1.1 200 OK
Cache-Control: max-age=1, no-cache
Connection: keep-alive
Content-Encoding: gzip
Content-Type: application/json;charset=UTF-8
Date: Tue, 08 Nov 2016 10:19:18 GMT
Server: nginx
Strict-Transport-Security: max-age=15768000
Transfer-Encoding: chunked
Vary: Accept-Encoding
Vary: Origin

{
    "attention": null,
    "city": "København V",
    "countryregionid": "DK",
    "message": "OK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "status": 200,
    "street1": "Kalvebod Brygge 45, 3.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "1560"
}
```

<a name="domain"></a>
## domain

This service returns data on a given domain name.

<a name="api"></a>
### API

    https://whois-api.dk-hostmaster.dk/domain/{domainname}

The service returns `200` if it can find a relevant object together with the public data.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="example"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/domain/eksempel.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/eksempel.dk | jq
```

```json
{
    "admin": {
        "attention": null,
        "city": "København V",
        "countryregionid": "DK",
        "mobilephone": null,
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "query_userid": "DKHM1-DK",
        "street1": "Kalvebod Brygge 45, 3.",
        "street2": null,
        "street3": null,
        "telefax": null,
        "userid": "DKHM1-DK",
        "useridtype": "V",
        "validregistrant": "1",
        "zipcode": "1560"
    },
    "admin_userid": "DKHM1-DK",
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
            "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
            "zonecontact_userid": "DKHM1-DK"
        },
        "auth02.ns.dk-hostmaster.dk": {
            "domain": "eksempel.dk",
            "domain_encoded": "eksempel.dk",
            "hostname": "auth02.ns.dk-hostmaster.dk",
            "hostname_encoded": "auth02.ns.dk-hostmaster.dk",
            "zonecontact_userid": "DKHM1-DK"
        }
    },
    "paiduntildate": "2017/06/30",
    "periodqty": "5",
    "proxy_userid": "DKHM1-DK",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "attention": null,
        "city": "København V",
        "countryregionid": "DK",
        "mobilephone": null,
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "query_userid": "DKHM1-DK",
        "street1": "Kalvebod Brygge 45, 3.",
        "street2": null,
        "street3": null,
        "telefax": null,
        "userid": "DKHM1-DK",
        "useridtype": "V",
        "validregistrant": "1",
        "zipcode": "1560"
    },
    "registrant_userid": "DKHM1-DK",
    "status": 200
}
```

<a name="domain-list"></a>
## domain list

<a name="api-1"></a>
### API

    https://whois-api.dk-hostmaster.dk/domain/list/userid/{handle}/{role}

The service returns `200` if it can find a relevant object holding the relevant role.

<a name="example-1"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/domain/list/handle/DKHM1-DK/role/proxy Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/list/handle/DKHM1-DK/role/proxy | jq
```

```json
{
    "attention": null,
    "city": "København V",
    "countryregionid": "DK",
    "domains": [
        {
            "domain": "eksempel.dk",
            "domain_encoded": "eksempel.dk"
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
    "role": "proxy",
    "status": 200,
    "street1": "Kalvebod Brygge 45, 3.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "1560"
}
```

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="handle"></a>
## handle

This service returns data on a given handle/user-id.

<a name="api-2"></a>
### API

    https://whois-api.dk-hostmaster.dk/handle/{userid}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="eksempel"></a>
### Eksempel

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/handle/DKHM1-DK | jq
```

```json
{
    "attention": null,
    "city": "København V",
    "countryregionid": "DK",
    "message": "OK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "status": 200,
    "street1": "Kalvebod Brygge 45, 3.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "1560"
}
```

<a name="host"></a>
## host

This service returns data on a given hostname/nameserver.

<a name="api-3"></a>
### API

    https://whois-api.dk-hostmaster.dk/host/{hostname}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="example-2"></a>
### Example

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk | jq
```

```json
{
    "glue_spooled": "J",
    "hostname": "auth01.ns.dk-hostmaster.dk",
    "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
    "message": "OK",
    "nameserver_status": "A",
    "status": 200,
    "zonecontact_userid": "DKHM1-DK"
}
```

<a name="query"></a>
## query

This service acts as a central entry points it relays to the above services:

<a name="api-4"></a>
### API

    https://whois-api.dk-hostmaster.dk/#query

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="domain-example"></a>
### Domain Example 

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/eksempel.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/eksempel.dk | jq
```

```json
{
    "admin": {
        "attention": null,
        "city": "København V",
        "countryregionid": "DK",
        "mobilephone": null,
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "query_userid": "DKHM1-DK",
        "street1": "Kalvebod Brygge 45, 3.",
        "street2": null,
        "street3": null,
        "telefax": null,
        "userid": "DKHM1-DK",
        "useridtype": "V",
        "validregistrant": "1",
        "zipcode": "1560"
    },
    "admin_userid": "DKHM1-DK",
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
            "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
            "zonecontact_userid": "DKHM1-DK"
        },
        "auth02.ns.dk-hostmaster.dk": {
            "domain": "eksempel.dk",
            "domain_encoded": "eksempel.dk",
            "hostname": "auth02.ns.dk-hostmaster.dk",
            "hostname_encoded": "auth02.ns.dk-hostmaster.dk",
            "zonecontact_userid": "DKHM1-DK"
        }
    },
    "paiduntildate": "2017/06/30",
    "periodqty": "5",
    "proxy_userid": "DKHM1-DK",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "attention": null,
        "city": "København V",
        "countryregionid": "DK",
        "mobilephone": null,
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "query_userid": "DKHM1-DK",
        "street1": "Kalvebod Brygge 45, 3.",
        "street2": null,
        "street3": null,
        "telefax": null,
        "userid": "DKHM1-DK",
        "useridtype": "V",
        "validregistrant": "1",
        "zipcode": "1560"
    },
    "registrant_userid": "DKHM1-DK",
    "status": 200
}
```

<a name="host-example"></a>
### Host Example 

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/auth01.ns.dk-hostmaster.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/auth01.ns.dk-hostmaster.dk | jq
```

```json
{
    "glue_spooled": "J",
    "hostname": "auth01.ns.dk-hostmaster.dk",
    "hostname_encoded": "auth01.ns.dk-hostmaster.dk",
    "message": "OK",
    "nameserver_status": "A",
    "status": 200,
    "zonecontact_userid": "DKHM1-DK"
}
```

<a name="handle-example"></a>
### Handle Example 

Using `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/query/DKHM1-DK Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl -H "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/DKHM1-DK | jq
```

```json
{
    "attention": null,
    "city": "København V",
    "countryregionid": "DK",
    "message": "OK",
    "mobilephone": null,
    "name": "DK HOSTMASTER A/S",
    "phone": null,
    "query_userid": "DKHM1-DK",
    "status": 200,
    "street1": "Kalvebod Brygge 45, 3.",
    "street2": null,
    "street3": null,
    "telefax": null,
    "userid": "DKHM1-DK",
    "useridtype": "V",
    "validregistrant": "1",
    "zipcode": "1560"
}
```

<a name="resources"></a>
# Resources

Resources for DK Hostmaster WHOIS support can be found below.

<a name="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster RESTful WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a name="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `info@dk-hostmaster.dk`

<a name="additional-information"></a>
## Additional Information

The DK Hostmaster website service page

  * `https://www.dk-hostmaster.dk/en/whois`

<a name="appendices"></a>
# Appendices

<a name="http-status-codes"></a>
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

