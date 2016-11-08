# DK Hostmaster RESTful WHOIS Service Specification

2016/11/08
Revision: 1.0

*PLEASE NOTE THAT THIS SERVICE IS CURRENTLY IN BETA AND CHANGES MIGHT BE IMPLEMENTED WHICH BREACK BACKWARDS COMPATIBILITY*

# Table of Contents

<!-- MarkdownTOC -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
    - [License](#license)
    - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
    - [Encoding](#encoding)
    - [Rate Limiting](#rate-limiting)
- [Service](#service)
    - [domain](#domain)
    - [domain list](#domain-list)
    - [handle](#handle)
    - [host](#host)
    - [query](#query)
- [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)
- [Appendices](#appendices)
    - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a name="introduction"></a>
# Introduction

This document describes and specifies the implementation offered by DK Hostmaster A/S for interaction with the central registry for the ccTLD dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service in not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by DK Hostmaster. If you are a registrar, you might be interested in [the DK Hostmaster Domain Availability Service (DAS)](https://github.com/DK-Hostmaster/das-service-specification) as an alternative.

<a name="about-this-document"></a>
# About this Document

This specification describes version 2 (2.0.x) of the DK Hostmaster WHOIS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
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

The WHOIS service offered by DK Hostmaster A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

<a name="features"></a>
# Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Handle inquiry
- Support for multiple encodings (see: Encodings below)
- Support for both IPv6 and IPv6

<a name="implementation-limitations"></a>
# Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English. 

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

<a name="service"></a>
# Service

<a name="domain"></a>
## domain

This service returns data on a given domain name.

<a name="api"></a>
### API

    https://whois.dk-hostmaster.dk/domain/{domainname}

The service returns `200` if it can find a relevant object together with the public data.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |

<a name="example"></a>
### Example

Using `httpie`

```bash
$ http http://localhost:3000/domain/eksempel.dk?format=json
```

Using `curl` and `jq`

```bash
$ curl http://localhost:3000/domain/eksempel.dk?format=json | jq
```

```json
{
    "admin": {
        "attention": null, 
        "city": "København V.", 
        "countryregionid": "DK", 
        "email": "test@dk-hostmaster.dk", 
        "name": "DK Hostmaster A/S", 
        "phone": "+45 33 64 60 60", 
        "publicstatus": "J", 
        "street1": "Kalvebod Brygge 45, 3.", 
        "street2": null, 
        "street3": null, 
        "telefax": "+45 33 64 60 66", 
        "userid": "DKHM1-DK", 
        "zipcode": "1560"
    }, 
    "createddate": "2016-07-05", 
    "dnssec": "J", 
    "domain": "eksempel.dk", 
    "domain_encoded": "eksempel.dk", 
    "domain_type": null, 
    "message": "OK", 
    "nameservers": {
        "ns1.eksempel.dk": {
            "domain": "eksempel.dk", 
            "domain_encoded": "eksempel.dk", 
            "hostname": "ns1.eksempel.dk", 
            "hostname_encoded": "ns1.eksempel.dk", 
            "zonecontact_userid": "DKHM1-DK"
        }, 
        "ns1.xn--4cabco7dk5a.dk": {
            "domain": "eksempel.dk", 
            "domain_encoded": "eksempel.dk", 
            "hostname": "ns1.æøåöäüé.dk", 
            "hostname_encoded": "ns1.xn--4cabco7dk5a.dk", 
            "zonecontact_userid": "DKHM1-DK"
        }, 
        "ns2.eksempel.dk": {
            "domain": "eksempel.dk", 
            "domain_encoded": "eksempel.dk", 
            "hostname": "ns2.eksempel.dk", 
            "hostname_encoded": "ns2.eksempel.dk", 
            "zonecontact_userid": "DKHM1-DK"
        }, 
        "ns2.xn--4cabco7dk5a.dk": {
            "domain": "eksempel.dk", 
            "domain_encoded": "eksempel.dk", 
            "hostname": "ns2.æøåöäüé.dk", 
            "hostname_encoded": "ns2.xn--4cabco7dk5a.dk", 
            "zonecontact_userid": "DKHM1-DK"
        }
    }, 
    "paiduntildate": "2021-07-05", 
    "periodqty": 5, 
    "proxy_userid": "DKHM1-DK", 
    "public_deletedate": null, 
    "public_domain_status": "A", 
    "registrant": {
        "attention": null, 
        "city": "København V.", 
        "countryregionid": "DK", 
        "email": "test@dk-hostmaster.dk", 
        "name": "DK Hostmaster A/S", 
        "phone": "+45 33 64 60 60", 
        "publicstatus": "J", 
        "street1": "Kalvebod Brygge 45, 3.", 
        "street2": null, 
        "street3": null, 
        "telefax": "+45 33 64 60 66", 
        "userid": "DKHM1-DK", 
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

    https://whois.dk-hostmaster.dk/domain/list/userid/{handle}/{role}

The service returns `200` if it can find a relevant object holding the relevant role.

<a name="example-1"></a>
### Example

Using `httpie`

```bash
$ http http://localhost:3000/domain/list/handle/DKHM1-DK/role/proxy?format=json
```

Using `curl` and `jq`

```bash
$ curl http://localhost:3000/domain/list/handle/DKHM1-DK/role/proxy?format=json | jq
```

```json
{
    "attention": null, 
    "city": "København V.", 
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
    "email": "test@dk-hostmaster.dk", 
    "message": "OK", 
    "name": "DK Hostmaster A/S", 
    "phone": "+45 33 64 60 60", 
    "publicstatus": "J", 
    "role": "proxy", 
    "status": 200, 
    "street1": "Kalvebod Brygge 45, 3.", 
    "street2": null, 
    "street3": null, 
    "telefax": "+45 33 64 60 66", 
    "userid": "DKHM1-DK", 
    "zipcode": "1560"
}
```

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |

<a name="handle"></a>
## handle

This service returns data on a given handle/user-id.

<a name="api-2"></a>
### API

    https://whois.dk-hostmaster.dk/handle/{userid}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |

<a name="eksempel"></a>
### Eksempel

Using `httpie`

```bash
$ http http://localhost:3000/handle/DKHM1-DK?format=json
```

Using `curl` and `jq`

```bash
$ curl http://localhost:3000/handle/DKHM1-DK?format=json | jq
```

```json
{
    attention: null,
    city: "København V.",
    countryregionid: "DK",
    email: "test@dk-hostmaster.dk",
    message: "OK",
    name: "DK Hostmaster A/S",
    phone: "+45 33 64 60 60",
    publicstatus: "J",
    status: 200,
    street1: "Kalvebod Brygge 45, 3.",
    street2: null,
    street3: null,
    telefax: "+45 33 64 60 66",
    userid: "DKHM1-DK",
    zipcode: "1560"
}
```

<a name="host"></a>
## host

This service returns data on a given hostname/nameserver.

<a name="api-3"></a>
### API

    https://whois.dk-hostmaster.dk/host/{hostname}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |

<a name="example-2"></a>
### Example

Using `httpie`

```bash
$ http http://localhost:3000/host/ns1.eksempel.dk?format=json
```

Using `curl` and `jq`

```bash
$ curl http://localhost:3000/host/ns1.eksempel.dk?format=json | jq
```

```json
{
    glue_spooled: "172.168.1.1",
    hostname: "ns1.eksempel.dk",
    hostname_encoded: "ns1.eksempel.dk",
    message: "OK",
    nameserver_status: "A",
    status: 200,
    zonecontact_userid: "DKHM1-DK"
}
```

<a name="query"></a>
## query

This service acts as a central entry points it relays to the above services:

<a name="api-4"></a>
### API

    https://whois.dk-hostmaster.dk/#query

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |


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
[RFC:3986]: https://tools.ietf.org/html/rfc3986 

