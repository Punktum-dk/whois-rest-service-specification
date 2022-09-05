![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DK Hostmaster RESTful WHOIS Service Specification

![Markdownlint Action](https://github.com/DK-Hostmaster/whois-rest-service-specification/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/whois-rest-service-specification/workflows/Spellcheck%20Action/badge.svg)

2021-10-04
Revision: 4.2

**PLEASE NOTE THAT THIS SERVICE IS CURRENTLY IN BETA AND CHANGES MIGHT BE IMPLEMENTED WHICH BREAK BACKWARDS COMPATIBILITY**

# Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true"  autoanchor="true" -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Available Environments](#available-environments)
  - [Production Environment](#production-environment)
  - [Sandbox Environment](#sandbox-environment)
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
  - [host](#host)
    - [API](#api-3)
    - [Example](#example-3)
  - [query](#query)
    - [API](#api-4)
    - [Domain Example](#domain-example)
    - [Host Example](#host-example)
- [Test Data](#test-data)
- [References](#references)
- [Resources](#resources)
  - [Mailing list](#mailing-list)
  - [Issue Reporting](#issue-reporting)
  - [Additional Information](#additional-information)
- [Appendices](#appendices)
  - [HTTP Status Codes](#http-status-codes)
  - [Public Status Values](#public-domain-status-values)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
# Introduction

This document describes and specifies the a RESTful alternative to the WHOIS implementation offered by DK Hostmaster A/S. It is primarily aimed at a technical audience and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS RESTful service is optimized for structured querying in contrast to it's text-based origin implementing the [WHOIS service](https://github.com/DK-Hostmaster/whois-service-specification) responding on port 43,

With the introduction of the registrar model, the WHOIS capabilities are being extended so these can serve information on registrars.

The registrar model, [description here conceptually][concept], gives registrars the option to manage a customer’s .dk domain name if the customer would prefer this. This is referred to as "registrar management". The alternative allowing the customer to manage their domain name themselves is referred to as "registrant management".

<a id="about-this-document"></a>
# About this Document

This specification describes version 4 (4.X.X) of the DK Hostmaster WHOIS RESTful service implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS RESTful service implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

<a id="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
## Document History

- 4.2 2021-10-04
  - Added information on [Available Environments](#available-environments) with the availability of the WHOIS API service in the sandbox environment and related [Test Data](#test-data)

- 4.1 2021-10-02
  - Added [References](#references) section
  - Corrected a few links and cleaned up some of the resources

- 4.0 2021-08-26
  - Added documentation on data retrieval for domain names offered from a waiting list
  - Date fields in responses have changed format from: `YYYY/MM/DD` to `YYYY-MM-DDTHH:MM:SSTZ` adhering to [ISO:8601][ISO:8601]
  - Responses to domain name queries have been extended with registrar information
  - Above extensions was introduced with version 4.0.0 of the DK Hostmaster WHOIS RESTful service

- 3.1 2021-03-15
  - Added appendix on status values and clarified the explanation on status

- 3.0 2019-11-25
  - Support for queries using user handles are no longer supported and the API endpoint `https://whois-api.dk-hostmaster.dk/handle/{userid}` has been removed
  - Support for queries using roles are not longer supported and the API endpoint `https://whois-api.dk-hostmaster.dk/domain/list/handle/{userid}/role/{role}` has been removed
  - Support for user handles in domain query result sets have been removed (`registrant_userid`, `query_userid` and `userid`), for API endpoints: `https://whois-api.dk-hostmaster.dk/domain/{domainname}` and `https://whois-api.dk-hostmaster.dk/#query`
  - Above deprecations was introduced with version 3.0.0 of the DK Hostmaster WHOIS RESTful service

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
- Support for multiple encodings (see: [Encoding](#encoding) below)
- Support for both IPv4 and IPv6
- Dates in [ISO:8601] format

<a id="available-environments"></a>

## Available Environments

DK Hostmaster offers the following environments:

| Environment | Role | Policies |
| ----------- | ---- | ----------- |
| production  | production | This environment is the production environment for the DK Hostmaster WHOIS API Service |
| sandbox     | development | This environment is intended for client development towards the DK Hostmaster WHOIS API Service |

For information on what service and specification is applicable and available, consult the [DK Hostmaster WHOIS Service Wiki][WIKI]

For use please see the section on [Test Data](#test-data).

<a id="production-environment"></a>
### Production Environment

- requests made to this environment will reflect live production data

Production is available at: `whois.dk-hostmaster.dk` port `43`

<a id="sandbox-environment"></a>
### Sandbox Environment

- Queries made to this environment will reflect data only available in the isolated sandbox environment, please see the [sandbox environment specification](https://github.com/DK-Hostmaster/sandbox-environment-specification) for details.
- Please additional data please, see the section on [Test Data](#test-data).

<a id="implementation-limitations"></a>
# Implementation Limitations

<a id="localization"></a>
## Localization

In general the service is not localized and all WHOIS information is provided in English.

Dates are provided in in [ISO:8601][ISO:8601] format.

<a id="media-type--format"></a>
## Media-type / Format

The service only supports **JSON**.

<a id="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8
- Punycode [RFC:5891][RFC:5891]
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

And a generic API:

- query

Which relays to the specific service APIs for the mapped entity being one of the ones listed above.

The service requires that the accept header is specified to be `application/json`, all of the below examples demonstrates this using the command line utilities `curl` or `httpie`.

As described under Implementation Limitations, the service only supports **JSON**. This has to be specified using the **HTTP** header: `Accept`

If the header is unspecified, not specified correctly or specified to an unsupported format, the service will error with HTTP status code: `415`

```bash
$ curl https://whois-api.dk-hostmaster.dk/domain/eksempel.dk
"Unsupported Media Type"
```

Correct specification using `curl` should be as follows:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/eksempel.dk
{"createddate": "1999-05-17T00:00:00+02:00","dnssec": "J","domain": "eksempel.dk","domain_encoded": "eksempel.dk","domain_type": "V","message": "OK","nameservers": {"auth01.ns.dk-hostmaster.dk": {"domain": "eksempel.dk","domain_encoded": "eksempel.dk","hostname": "auth01.ns.dk-hostmaster.dk","hostname_encoded": "auth01.ns.dk-hostmaster.dk"},"auth02.ns.dk-hostmaster.dk": {"domain": "eksempel.dk","domain_encoded": "eksempel.dk","hostname": "auth02.ns.dk-hostmaster.dk","hostname_encoded": "auth02.ns.dk-hostmaster.dk"}},"paiduntildate": "2022-06-30T00:00:00+02:00","periodqty": "5","public_deletedate": null,"public_domain_status": "A","registrant": {"city": "København S","countryregionid": "DK","name": "DK HOSTMASTER A/S","phone": null,"street1": "Ørestads Boulevard 108, 11.","street2": null,"street3": null,"useridtype": "V","zipcode": "2300"},"status": 200}
```

And for `httpie`

```bash
$ http https://whois-api.dk-hostmaster.dk/domain/eksempel.dk Accept:'application/json'

HTTP/1.1 200 OK
Cache-Control: max-age=1, no-cache
Connection: keep-alive
Content-Encoding: gzip
Content-Type: application/json;charset=UTF-8
Date: Mon, 25 Nov 2019 09:29:41 GMT
Server: nginx
Strict-Transport-Security: max-age=15768000
Transfer-Encoding: chunked
Vary: Accept-Encoding
Vary: Origin

{
    "createddate": "1999-05-17T00:00:00+02:00",
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
    "paiduntildate": "2022-06-30T00:00:00+02:00",
    "periodqty": "5",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "city": "København S",
        "countryregionid": "DK",
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "street1": "Ørestads Boulevard 108, 11.",
        "street2": null,
        "street3": null,
        "useridtype": "V",
        "zipcode": "2300"
    },
    "status": 200
}
```

<a id="domain"></a>
## domain

This service returns data on a given domain name.

As of version 2.X.X of the service, admin/proxy information is no longer part of the response data, same goes for name server administrators/zone contact handles.

As of version 3.X.X of the service, registrant user-id/handle information is no longer part of the response data.

As of version 4.X.X of the service:

- information on a domain name offered from a waiting list is available.
- registrar information is included in the response data.

With the new registrar information included where applicable the responses can and will vary depending on the model chosen for administration of the given domain name.

If the domain name is under registrar management, a registrar data section is included:

```json
"registrar": {
    "city": "København S",
    "countryregionid": "DK",
    "is_public": false,
    "logo": "",
    "name": "DK Hostmaster A/S",
    "phone": "+45 33 64 60 60",
    "street1": "Ørestads Boulevard 108, 11.",
    "street2": null,
    "street3": null,
    "url": "www.dk-hostmaster.dk",
    "useridtype": "V",
    "zipcode": "2300"
},
```

NB: The above example is constructed.

The fields are:

- `city`, city part of address
- `countryregionid`, country part of address
- `is_public`, flag indicating if the registrar is listed in the public registrar listing
- `logo`, URL to public logo
- `name`, name
- `phone`, public phone number
- `street1`, first street part of address
- `street2`, second street part of address
- `street3`, third street part of address
- `useridtype`, internal type indicating whether the entity is an association, company, individual or public organization
- `zipcode`, zipcode part of address

Do note that the listing on the public registrar listing and have address information redacted from public interfaces is not that same.

If the domain name is under registrar management, but the registrars address information is not public, the section look as follows:

```json
"registrar": {
    "is_public": false
}
```

If the domain name is under registrant management, not registrar data is included.

So the responses can be divided into two groups based on the registrar data section

- Registrant managed, section is not included
- Registrar managed, section is included, excluding address data if the address is not public

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
    "createddate": "1999-05-17T00:00:00+02:00",
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
    "paiduntildate": "2022-06-30T00:00:00+02:00",
    "periodqty": "5",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "city": "København S",
        "countryregionid": "DK",
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "street1": "Ørestads Boulevard 108, 11.",
        "street2": null,
        "street3": null,
        "useridtype": "V",
        "zipcode": "2300"
    },
    "status": 200
}
```

If the queried domain name is offered from a waiting list, the response will look as follows:

```json
{
    "domain": "eksempel.dk",
    "domain_encoded": "eksempel.dk",
    "message": "OK",
    "public_domain_status": "W",
    "status": 200
}
```

Do note the special status: `W`.

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
    "createddate": "1999-05-17T00:00:00+02:00",
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
    "paiduntildate": "2022-06-30T00:00:00+02:00",
    "periodqty": "5",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "city": "København S",
        "countryregionid": "DK",
        "name": "DK HOSTMASTER A/S",
        "phone": null,
        "street1": "Ørestads Boulevard 108, 11.",
        "street2": null,
        "street3": null,
        "useridtype": "V",
        "zipcode": "2300"
    },
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

<a id="test-data"></a>
## Test Data

The sandbox uses a combination of a predefined set of test data and data added to the sandbox environment by use.

<a id="domains"></a>
### Domains

| Domain name | Status | Notes |
|-------------|--------|-------|
| `dk-hostmaster.dk` | `A` | The domain is visible and active |
| `æøåöäüé.dk` | `A` | The domain is visible and active |
| `waiting-list.dk` | `W` | The domain status is awaiting the designated registrant |
| * | * | Depending on what domains have been registered with the sandbox environment. Please see the [sandbox environment specification](https://github.com/DK-Hostmaster/sandbox-environment-specification) for details. |

<a id="references"></a>
# References

Here is a list of documents and references used in this document

1. [RFC:3912 WHOIS Protocol Specification][RFC:3912]
1. [RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol][RFC:5891]
1. [RFC:3986 Uniform Resource Identifier (URI): Generic Syntax][RFC:3986]
1. [ISO-8601: International date format][ISO:8601]

<a id="resources"></a>
# Resources

Resources for DK Hostmaster WHOIS support can be found below.

<a id="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster RESTful WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

- `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a id="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us. You are of course welcome to post these to the mailing list mentioned above, otherwise use the regular support channels.

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

<a id="public-domain-status-values"></a>
## Public Domain Status Values

| Status | Description                                                                                        |
| ------ | -------------------------------------------------------------------------------------------------- |
| `A`    | Domain name is or being published to the zone                                                      |
| `B`    | Domain name is blocked and not being published to the zone (special status)                        |
| `H`    | Domain name is being withheld from being published to the zone (general status)                    |
| `I`    | Domain name is inactive and is not being published to the zone (activation required by registrant) |
| `W`    | Domain name is offered from a waiting list to (acceptance is required by designated registrant) |

[RFC:3492]: https://tools.ietf.org/html/rfc3492
[RFC:3912]: https://tools.ietf.org/html/rfc3912
[RFC:3986]: https://tools.ietf.org/html/rfc3986
[ISO:8601]: https://www.iso.org/iso-8601-date-and-time-format.html
[CONCEPT]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[RFC:5891]: https://tools.ietf.org/html/rfc5891
[WIKI]: https://github.com/DK-Hostmaster/whois-rest-service-specification/wiki
