![Punktum dk Logo](https://punktum.dk/sites/default/files/logo/dk_logo_symbol_1.png)

# Punktum dk RESTful WHOIS Service Specification

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
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
    - [Example waiting list](#example-waiting-list)
    - [Field Values](#field-values)
  - [host](#host)
    - [API](#api-1)
    - [Example](#example-1)
    - [Field Values](#field-values-1)
  - [query](#query)
    - [API](#api-2)
    - [Domain Example](#domain-example)
    - [Host Example](#host-example)
- [Test Data](#test-data)
- [References](#references)
- [Issue Reporting](#issue-reporting)
- [Appendices](#appendices)
  - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
# Introduction

This document describes and specifies a RESTful alternative to the WHOIS service offered by Punktum dk. It is primarily aimed at a technical audience, and the reader is expected to have prior knowledge of the WHOIS protocol and DNS registration.

The RESTful WHOIS service is optimized for structured querying, in contrast to the text-based [WHOIS service](https://github.com/Punktum-dk/whois-service-specification) responding on port 43.

The service provides information on domain names and, where applicable, on the registrars managing them.

A .dk domain name is administered under one of two models.

- Under "registrar management", a registrar administers the domain name on behalf of the registrant.
- Under "registrant management", the registrant administers the domain name directly.

The WHOIS response reflects which model applies to a given domain name.

<a id="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

Punktum dk is the registry for the ccTLD for Denmark (.dk). The current model is based on a sole registry, with Punktum dk maintaining the central DNS registry for .dk.

<a id="about-this-document"></a>
# About this Document

This specification describes version 4.2.1 of the Punktum dk RESTful WHOIS service implementation. Future releases will be reflected in updates to this document; please refer to the [Document History](#document-history) below for changes.

The document describes the current Punktum dk RESTful WHOIS service implementation. The specification aims to describe the latest release of the service, and the service version is listed in the [Document History](#document-history).

This document is not the authoritative source for business and policy rules, and possible discrepancies between this and any authoritative sources are regarded as errors in this document. This document is the external technical specification describing the implementation facing the users, and is an interpretation of authoritative sources and can therefore be erroneous.

Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

All examples provided in the document are fabricated/modified from real data to demonstrate commands etc. Any resemblance to actual data is coincidental.

This document is owned and maintained by Punktum dk A/S and must not be distributed without this information.

<a id="license"></a>
## License

This document is copyright by Punktum dk A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
## Document History

- 4.2.1 2026-07-24
  - Rewritten and modernized following Punktum dk's transition to registrar management
  - Refreshed all examples and field documentation based on current service responses

- 4.2 2021-10-04
  - Added information on [Available Environments](#available-environments) with the availability of the WHOIS API service in the sandbox environment and related [Test Data](#test-data)

- 4.1 2021-10-02
  - Added [References](#references) section
  - Corrected a few links and cleaned up some of the resources

- 4.0 2021-08-26
  - Added documentation on data retrieval for domain names offered from a waiting list
  - Date fields in responses have changed format from: `YYYY/MM/DD` to `YYYY-MM-DDTHH:MM:SSTZ` adhering to ISO 8601
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

<a id="features"></a>
# Features

The service implements the following features:

- Domain name inquiry
- Host name inquiry
- Registrar information for registrar-managed domain names
- Support for multiple encodings (see [Encoding](#encoding) below)

<a id="available-environments"></a>
# Available Environments

Punktum dk offers the following environments:

| Environment | Role        | Policies                                                                                        |
| ----------- | ----------- | ----------------------------------------------------------------------------------------------- |
| production  | production  | This environment is the production environment for the Punktum dk RESTful WHOIS Service         |
| sandbox     | development | This environment is intended for client development towards the Punktum dk RESTful WHOIS Service |

Access to the RESTful WHOIS service requires IP whitelisting in both environments. IP whitelisting for this service is administered solely by Punktum dk and cannot be managed by registrars through the Registrar Portal (RP). To request IP whitelisting, please contact Punktum dk via the [registrar contact form][CONTACTFORM].

<a id="production-environment"></a>
## Production Environment

- Requests made to this environment will reflect live production data.

Production is available at: `https://whois-api.dk-hostmaster.dk`

<a id="sandbox-environment"></a>
## Sandbox Environment

- Requests made to this environment will reflect data only available in the isolated sandbox environment, please see the [sandbox environment specification](https://github.com/Punktum-dk/sandbox-environment-specification) for details.

Sandbox is available at: `https://whois-api-sandbox.dk-hostmaster.dk/`

For test data available in the sandbox environment, please see the section on [Test Data](#test-data).

<a id="implementation-limitations"></a>
# Implementation Limitations

<a id="localization"></a>
## Localization

In general the service is not localized, and all WHOIS information is provided in English.

Dates are provided in [ISO 8601][ISO:8601] format.

<a id="media-type--format"></a>
## Media-type / Format

The service only supports **JSON**.

<a id="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8
- Punycode (see also [RFC:5891])
- ASCII
- URI encoding

<a id="rate-limiting"></a>
## Rate Limiting

The service does not enforce a rate limit. You can query the service as often as your integration requires, in both the production and the sandbox environment.

👉 Query volumes are expected to be proportional to actual business use. Punktum dk reserves the right to introduce a rate limit, or to remove IP-whitelisting, if use of the service affects the overall quality of the service.

<a id="security"></a>
## Security

The service supports the following protocols for transport security:

- TLSv1.2

<a id="service"></a>
# Service

The service offers the following APIs:

- `domain`, specialized for domain name inquiries
- `host`, specialized for host (name server) inquiries
- `query`, a generic API that relays to the specialized API matching the entity type of the inquiry

The base URLs for the production and sandbox environments are listed under [Available Environments](#available-environments).

The service only supports **JSON**, which must be requested using the HTTP `Accept` header set to `application/json`. If the header is omitted, malformed, or set to an unsupported format, the service responds with HTTP status code `415 Unsupported Media Type`:

```bash
$ curl https://whois-api.dk-hostmaster.dk/domain/eksempel.dk
"Unsupported Media Type"
```

The `Accept` header must be specified as follows:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/eksempel.dk
```

The examples in this document use `curl`, but any HTTP client capable of setting the `Accept` header, such as [HTTPie](https://httpie.io/), can be used.

<a id="domain"></a>
## domain

This API returns data on a given domain name.

<a id="api"></a>
### API

Domain name information is requested using the following endpoint:

```
GET /domain/{domainname}
```

Where `{domainname}` is the domain name being inquired about. The domain name can be provided using any of the encodings listed under [Encoding](#encoding); for IDNA domain names, either the Unicode form or the punycode (ACE) form can be used.

<a id="example"></a>
### Example

Requesting information for the domain name `eksempel.dk`:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/domain/eksempel.dk
```

The response for a registrar-managed domain name looks as follows:

```json
{
    "createddate": "1999-05-17T00:00:00+02:00",
    "dnssec": "J",
    "domain": "eksempel.dk",
    "domain_encoded": "eksempel.dk",
    "domain_type": "N",
    "message": "OK",
    "nameservers": {
        "ns1.example.dk": {
            "domain": "eksempel.dk",
            "domain_encoded": "eksempel.dk",
            "hostname": "ns1.example.dk",
            "hostname_encoded": "ns1.example.dk"
        },
        "ns2.example.dk": {
            "domain": "eksempel.dk",
            "domain_encoded": "eksempel.dk",
            "hostname": "ns2.example.dk",
            "hostname_encoded": "ns2.example.dk"
        }
    },
    "paiduntildate": "2027-06-30T00:00:00+02:00",
    "periodqty": "1",
    "public_deletedate": null,
    "public_domain_status": "A",
    "registrant": {
        "city": "København S",
        "countryregionid": "DK",
        "email": "info@punktum.dk",
        "id_status": "N",
        "name": "Punktum dk A/S",
        "phone": "+45 33646060",
        "street1": "Ørestads Boulevard 108, 11.",
        "street2": null,
        "street3": null,
        "zipcode": "2300"
    },
    "registrar": {
        "city": "København S",
        "countryregionid": "DK",
        "email": "info@example.dk",
        "is_public": true,
        "logo": "https://selvbetjening.dk-hostmaster.dk/img/registrar/0000",
        "name": "Example Registrar A/S",
        "phone": "+45 00000000",
        "street1": "Eksempelvej 1",
        "street2": null,
        "street3": null,
        "url": "https://example.dk",
        "zipcode": "2300"
    },
    "status": 200
}
```

The top-level fields are:

| Field                  | Description                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `createddate`          | Date of registration in [ISO 8601][ISO:8601] format                                                               |
| `dnssec`               | Indicates whether DNSSEC is active for the domain name, see [Field Values](#field-values)                         |
| `domain`               | The domain name inquired about, in Unicode form                                                                   |
| `domain_encoded`       | The domain name in encoded (punycode/ACE) form, identical to `domain` for ASCII domain names                      |
| `domain_type`          | Type of the domain name, see [Field Values](#field-values)                                                        |
| `message`              | Status message accompanying the response, e.g. `OK`                                                               |
| `nameservers`          | The name servers serving the domain name, see below                                                               |
| `paiduntildate`        | Date the domain name is paid until, in [ISO 8601][ISO:8601] format                                                |
| `periodqty`            | Registration period in years                                                                                      |
| `public_deletedate`    | Date the domain name is scheduled for deletion, in [ISO 8601][ISO:8601] format, `null` if not set                 |
| `public_domain_status` | The public status of the domain name, see [Field Values](#field-values)                                           |
| `registrant`           | The registrant of the domain name, see below                                                                      |
| `registrar`            | The registrar managing the domain name, included only for registrar-managed domain names, see below               |
| `status`               | The HTTP status code of the response                                                                              |

Each entry in the `nameservers` section contains:

| Field              | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `hostname`         | The host name of the name server, in Unicode form        |
| `hostname_encoded` | The host name in encoded (punycode/ACE) form             |
| `domain`           | The domain name served by the name server, in Unicode form |
| `domain_encoded`   | The served domain name in encoded (punycode/ACE) form    |

The `registrant` section contains:

| Field             | Description                                                             |
| ----------------- | ----------------------------------------------------------------------- |
| `city`            | City part of the address                                                |
| `countryregionid` | Country part of the address, as an [ISO 3166-1] alpha-2 code            |
| `email`           | Email address, `null` if not available                                  |
| `id_status`       | The identity verification status of the registrant, see [Field Values](#field-values) |
| `name`            | Name                                                                    |
| `phone`           | Phone number, `null` if not available                                   |
| `street1`         | First street part of the address                                        |
| `street2`         | Second street part of the address, `null` if not set                    |
| `street3`         | Third street part of the address, `null` if not set                     |
| `zipcode`         | Zipcode part of the address                                             |

The `registrar` section is included only for registrar-managed domain names. For registrant-managed domain names, the section is omitted entirely. The registrar's address details are included only when the registrar is listed in the public registrar listing; when it is not, the section contains only `is_public` and `name`.

| Field             | Description                                                                    |
| ----------------- | ------------------------------------------------------------------------------ |
| `city`            | City part of the address                                                       |
| `countryregionid` | Country part of the address, as an [ISO 3166-1] alpha-2 code                   |
| `email`           | Public email address, `null` if not available                                  |
| `is_public`       | Flag indicating whether the registrar is listed in the public registrar listing |
| `logo`            | URL to the registrar's public logo                                             |
| `name`            | Name                                                                           |
| `phone`           | Public phone number, `null` if not available                                   |
| `street1`         | First street part of the address                                               |
| `street2`         | Second street part of the address, `null` if not set                           |
| `street3`         | Third street part of the address, `null` if not set                            |
| `url`             | The registrar's public website                                                 |
| `zipcode`         | Zipcode part of the address                                                    |

<a id="example-waiting-list"></a>
### Example waiting list

A domain name offered from a waiting list is not registered, and the response therefore contains only a limited set of fields, with `public_domain_status` set to `W`:

```json
{
    "domain": "waiting-list.dk",
    "domain_encoded": "waiting-list.dk",
    "message": "OK",
    "public_domain_status": "W",
    "status": 200
}
```

<a id="field-values"></a>
### Field Values

The following fields in the domain response return coded values.

**`domain_type`**

| Value | Description |
| ----- | ----------- |
| `N`   | The domain name does not have the VID service |
| `V`   | The domain name has the VID service |

**`dnssec`**

| Value | Description |
| ----- | ----------- |
| `J`   | DNSSEC is active for the domain name |
| `N`   | DNSSEC is not active for the domain name |

**`public_domain_status`**

| Value | Description |
| ----- | ----------- |
| `A`   | The domain name is published to the zone |
| `B`   | The domain name is blocked and not being published to the zone (special status) |
| `H`   | The domain name is being withheld from being published to the zone (general status) |
| `I`   | The domain name is inactive and not being published to the zone (activation required by the registrant) |
| `W`   | The domain name is offered from a waiting list (acceptance required by the designated registrant) |

**`id_status`**

| Value | Description |
| ----- | ----------- |
| `N`   | The registrant has completed identity verification using electronic ID (MitID) |
| `V`   | The registrant has verified their identity by submitting documentation |
| `F`   | The registrant has completed the identity check with their registrar |
| `R`   | The registrant has been assessed by Punktum dk's risk engine as not required to complete the identity check |

<a id="host"></a>
## host

This API returns data on a given host (name server).

<a id="api-1"></a>
### API

Host information is requested using the following endpoint:

```
GET /host/{hostname}
```

Where `{hostname}` is the host name being inquired about. The host name can be provided using any of the encodings listed under [Encoding](#encoding); for IDNA host names, either the Unicode form or the punycode (ACE) form can be used.

<a id="example-1"></a>
### Example

Requesting information for the host `ns1.example.dk`:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/host/ns1.example.dk
```

The response looks as follows:

```json
{
    "glue_spooled": "J",
    "hostname": "ns1.example.dk",
    "hostname_encoded": "ns1.example.dk",
    "message": "OK",
    "nameserver_status": "A",
    "status": 200
}
```

The fields are:

| Field               | Description                                                                       |
| ------------------- | --------------------------------------------------------------------------------- |
| `glue_spooled`      | Indicates whether a glue record is being spooled for the host, see [Field Values](#field-values-1) |
| `hostname`          | The host name inquired about, in Unicode form                                     |
| `hostname_encoded`  | The host name in encoded (punycode/ACE) form, identical to `hostname` for ASCII host names |
| `message`           | Status message accompanying the response, e.g. `OK`                               |
| `nameserver_status` | The status of the host, see [Field Values](#field-values-1)                       |
| `status`            | The HTTP status code of the response                                              |

<a id="field-values-1"></a>
### Field Values

The following fields in the host response return coded values.

**`glue_spooled`**

| Value | Description |
| ----- | ----------- |
| `J`   | A glue record is being spooled for the host |
| `N`   | No glue record is being spooled for the host |

**`nameserver_status`**

| Value | Description |
| ----- | ----------- |
| `A`   | The host is active |
| `G`   | The host is awaiting approval by the registrant of the domain |
| `I`   | The host is awaiting approval by the party responsible for the name server |
| `S`   | The host is marked for deletion |

<a id="query"></a>
## query

This API is a generic entry point that accepts both domain names and host names. It determines the entity type of the inquiry and relays it to the specialized API, returning the same response as a direct inquiry to [domain](#domain) or [host](#host).

This is useful when the entity type of the inquiry is not known in advance.

<a id="api-2"></a>
### API

Information is requested using the following endpoint:

```
GET /query/{name}
```

Where `{name}` is the domain name or host name being inquired about. The name can be provided using any of the encodings listed under [Encoding](#encoding).

<a id="domain-example"></a>
### Domain Example

Querying a domain name returns the same response as the [domain](#domain) API:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/eksempel.dk
```

<a id="host-example"></a>
### Host Example

Querying a host name returns the same response as the [host](#host) API:

```bash
$ curl --header "Accept: application/json" https://whois-api.dk-hostmaster.dk/query/auth01.ns.dk-hostmaster.dk
```

<a id="test-data"></a>
# Test Data

The sandbox environment provides a set of predefined test domains, in addition to any domains you register yourself via EPP or RP. For the list of predefined test domains and details on the sandbox environment, see the [sandbox environment specification](https://github.com/Punktum-dk/sandbox-environment-specification).

One predefined test domain is worth noting here: `waiting-list.dk` simulates the transient `Offered to waiting list` state, which is otherwise difficult to reproduce on demand. It can be queried in the sandbox environment to see the response shown under [the domain example for a name offered to the waiting list](#example-waiting-list).

<a id="references"></a>
# References

Here is a list of documents and references used in this document:

1. [RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol][RFC:5891]
1. [ISO 3166-1: Alpha-2 two-letter country codes][ISO 3166-1]
1. [ISO 8601: International date format][ISO:8601]

<a id="issue-reporting"></a>
# Issue Reporting

For issue reporting related to this specification, the RESTful WHOIS implementation, or the production environment, please contact us at registrar@punktum.dk or via the [registrar contact form][CONTACTFORM].

<a id="appendices"></a>
# Appendices

<a id="http-status-codes"></a>
## HTTP Status Codes

The service uses the following HTTP status codes:

| Status Code | Description                   |
| ----------- | ----------------------------- |
| `200`       | OK                            |
| `400`       | Bad request                   |
| `404`       | Object not found              |
| `415`       | Unsupported media type        |
| `500`       | Internal server error         |
| `503`       | Service temporarily unavailable |

[RFC:5891]: https://www.rfc-editor.org/rfc/rfc5891
[ISO-3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1
[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601
[CONTACTFORM]: https://punktum.dk/en/contact-customer-service?lvl1=Registrars&lvl2=CSRegistrarOther
