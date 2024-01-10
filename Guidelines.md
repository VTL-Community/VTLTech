# Introduction

## Guidelines version

These technical guidelines address implementation issues for VTL standard version 2.1.

These guidelines are meant to facilitate developers of VTL tools (engines, interpreters, 
translators...) in order to make implementation of such tools easier and speedier.

## Terminology

These guidelines MAY use keywords from RFC 2119. When one of those keywords SHALL be 
interpreted as in the specification, they MUST be written upper-case.

## Glossary

Glossary of terms used throughout this document:

* **VTL program**: A collection of VTL statements that forms a complete VTL Transformation scheme;

# Web API for VTL engines

This chapter describes guidelines for implementors of web service applications providing access
to VTL engine or VTL technology stack implementations.

## Basic architecture

A VTL web service implementation SHOULD provide at least two basic HTTP POST endpoints:

1. `/metadata`: returns the structure(s) of one or more datasets;
2. `/compute`: returns the result of a submitted VTL program.

The arguments to each method call SHOULD be passed in the body section as a JSON object. The default 
HTTP `Accept` header has the value `Accept: application/json;q=1.0, text/plain;q=0.8, */*;q=0.6`.
Additional content-types not specified in this document MAY BE supported by a specific implementation and
SHALL BE selected through this header.

The submitted payload and the returned result SHOULD BE encoded in UTF-8.

The web application MAY contain an embedded VTL engine, or MAY submit the user request to a reachable
VTL engine instance located elsewhere. In the latter case, the web service MAY use any supported 
technology, such as JSON-RPC or CORBA, to transfer the request and retrieve the result.

## Internationalization

The results of the methods MAY contain locale-sensitive content, such as number or data format,
artifact descriptions, and so on. When appropriate, the implementation MAY support additional languages.

In this case the user MAY provide a priority list of languages with the HTTP `Accept-Language` header.
If the requested artifact is not available in one of the preferred languages, the English language SHOULD
BE used instead.

## Storage

Web services implementations MAY choose to rely on external storage technology in order to provide data 
to the submitted request. Web services implementations MAY additionally support user storage of VTL 
artifacts. In this case any details regarding how to achieve this, and the list of required HTTP endpoints, 
are left to each implementation and MUST be documented. 

Web services implementations SHOULD BE able to map VTL aliases used in the submitted program to specific
data sources, either internal or external, without the manual intervention of the user. When an alias is
mapped to a data source that requires authorization, the implementation MUST BE able to retrieve an user
context.

Web services implementations SHOULD BE able to retrieve any metadata for artifacts used in the submitted
program, without the manual intervention of the user. These metadata MAY BE retrieved from an external 
provider, if configured, and MAY require access to storage.

## Security

Security MUST BE implemented when the web service implementation require an user context in order to 
process the submitted VTL program. In this case, the web application container MUST BE configured to
forward the authentication token to the web service, or the application MUST support an authentication
and authorization mechanism, such as SPNEGO.

Confidentiality SHOULD BE enforced by deployting the implementation behind a reverse proxy or on an 
application server that is configured to provide a X509 certificate with the https:// protocol.

## `/metadata` entry point

This endpoint retrieves the metadata of one or more VTL datasets specified in the body argument with 
their aliases. The body argument MUST BE an array of strings, each of which MUST refer to a VTL alias,
known to the web service, that can be mapped to a dataset.

If an alias is unknown, or the request body is not a JSON array of strings, a HTTP 404 error MUST BE
returned to the client with an appropriate message description.

The response MUST BE a JSON file with exactly four dictionaries named `"datasets"`, `"structures"`,
`"variables"`, and `"domains"` respectively. Each of those dictionaries are described below.

### `"datasets"` dictionary

This dictionary describes each dataset. Each key MUST correspond to exactly one of the aliases specified 
in the request body argument. Each value MUST BE a JSON object containing the following properties:

1. `"structure"`:    a mandatory string property identifying the structure of the VTL dataset;
2. `"description"`:  An optional string property describing the contents of the dataset;
3. `"source"`:       an optional JSON object property containing data source information (i.e. an URL 
                     for data retrieval or specific text or date formats used in the data source);
4. `"presentation"`: an optional JSON object property containing additional information useful when
                     displaying data from this dataset, such as columns order and sorting information.

### `"structures"` dictionary

This dictionary describes all the structures mentioned in the previous dictionary. Each key MUST be an
identifier mentioned as the value of the `"structure"` property of the `"datasets"` dictionary.

Each value MUST BE a JSON object containing the following properties:

1. `"description"`: An optional string property describing the meaning of the structure;
2. `"components"`:  A mandatory JSON object property describing the components in the structure.

Each property of the `"components"` JSON object has the component name as its key and a JSON object 
as its value with the following properties:

1. `"role"`:     a mandatory string property specifying the role of the component, MUST be one of
                 `"identifier"`, `"measure"` and `"attribute"`;
3. `"subset"`:   an optional string property specifying if this component can only takes values from
                 a specified subset of the VTL represented variable domain;
4. `"nullable"`: an optional boolean property specifying that the component can take the null value;
                 this property is true by default for all component roles except for `"identifier"`,
                 in which case the property, if set, MUST have the value `false`.

### `"variables"` dictionary

This dictionary describes all the variables mentioned in the previous dictionary. Each key MUST be a
variable identifier mentioned as a key in each of the `"components"` JSON objects in the `"structures"`
dictionary.

Each value is a JSON object describing a VTL represented variable with the following properties:

1. `"domain"`:      a mandatory string property specifying the valuedomain of the VTL represented variable;
2. `"description"`: an optional string property describing the meaning of the VTL represented variable;

### `"domains"` dictionary

This dictionary describes all the non-basic valuedomains mentioned in both the `"components"` and the 
`"variables"` dictionaries. Each key MUST be a valuedomain identifier mentioned as the value of either
the `"subset"` or `"domain"` properties respectively.

Each value MUST BE a JSON object containing the following properties:

1. `"parent"`:      a mandatory string property specifying the parent valuedomain of this valuedomain;
2. `"description"`: an optional string property describing the meaning of the VTL valuedomain;

The JSON object MUST also have exactly one among these properties:

* `"enumeration"`: a JSON array of values of a type compatible with the parent valuedomain, representing
                   all the admissible values for the VTL valuedomain or subset;
* `"condition"`:   a string containing a VTL expression used to restrict the admissible values in the
                   valuedomain, the `value` alias SHOULD be used as a placeholder for the actual value;
* `"externalref"`: a string containing an opaque description of a VTL valuedomain stored externally, in case
                   of SDMX codelists this SHOULD be a SDMX 3.0 URI.
