# Introduction

## Guidelines version

These technical guidelines address implementation issues for VTL standard version 2.1.

These guidelines are meant to facilitate developers of VTL tools (engines, interpreters, 
translators...) in order to make implementation of such tools easier and speedier.

## Terminology

These guidelines MAY use keywords from RFC 2119. When one of those keywords SHALL be 
interpreted as in the specification, they MUST be written upper-case.

# Web API for VTL engines

This chapter describes guidelines for implementors of web service applications providing access
to VTL engines, or VTL engines exposing web services.

## Basic architecture

A VTL engine MAY expose a set of web services, or an application layer MAY support interoperability
with a VTL engine instance located elsewhere. In the latter case, the web service MAY use any 
technology supported by the engine instance, such as JSON-RPC or CORBA, to transfer control between
the engine and the application layer.

A VTL web service implementation SHOULD provide at least these endpoints:

1. `/dataset`: returns the contents of of one or more datasets;
2. `/structure`: returns the structure(s) of one or more datasets;
3. `/variable`: returns the description(s) of one or more variables;
4. `/domain`: returns the description of one or more valuedomain;
5. `/execution`: returns the status of a submitted VTL program or submits a new VTL program.

The arguments to each method call SHOULD be passed in the body section as a JSON object. The default 
HTTP `Accept` header has the value `Accept: application/json;q=1.0, text/plain;q=0.8, */*;q=0.6`.
Additional content-types not specified in this document MAY BE supported by a specific implementation
and SHALL BE selected through this header.

The submitted payload and the returned result SHOULD be encoded in UTF-8; when not otherwise specified,
the HTTP `POST` method SHOULD be used.

### `/dataset` endpoint

This endpoint returns the description and/or contents of one or more datasets specified in the body 
argument with their aliases.

The body argument MUST BE a JSON object, whose keys MUST refer to a VTL alias, known to the web 
service, and whose corresponding values MUST be JSON objects with the following properties:

1. `"data"`: a mandatory string property indicating whether the content of datasets should be
             included in the response. It MAY have the values `"none"`, `"data"`, `"cols"` or
             `"rows"`. If `"none"` is used, data SHOULD NOT be included in the response. If
             `"cols"` is used, data SHOULD be serialized in columnar format, as a JSON object
             with array values. If `"rows"` is used, data SHOULD BE serialized in row format,
             as an array of JSON objects. If `"data"` or another value is used, data SHOULD
             BE present in the response serialized with a format chosen by implementation.

The response MUST BE a JSON object, where each key MUST correspond to exactly one of the aliases
specified in the request body argument. Each value MUST BE a JSON object containing the following
properties:

1. `"structure"`:    a mandatory string property identifying the structure of the VTL dataset;
2. `"description"`:  An optional string property describing the contents of the dataset;
3. `"source"`:       an optional JSON object property containing data source information (i.e. an URL 
                     for data retrieval or specific text or date formats used in the data source);
4. `"presentation"`: an optional JSON object property containing additional information useful when
                     displaying data from this dataset, such as sorting or paging information;
5. `"data"`:         an optional property that MUST be either a JSON array or a JSON object. If the
                     property is a JSON array, each element of the array MUST BE a JSON object
                     representing a single datapoint, with each key corresponding to a component and
                     the value representing the dataset component value. If the property is a JSON
                     object, keys of each property MUST be component names, and the corresponding
                     values MUST be arrays containing as their n-th element the value of that
                     component in the n-th datapoint of the dataset, in encounter order.

The web service implementation MAY choose not to provide data content, even if requested, or to
provide the content in either columnar or row format regardless of the expressed preference; clients
SHOULD NOT expect a particular serialization format.

The `/dataset` endpoint MAY accept the `PUT` HTTP method. The body argument MUST be a JSON object
with the following properties:

1. `"alias"`:        The alias to be used for this VTL dataset in a VTL transformation scheme;
2. `"structure"`:    the same as above;
3. `"description"`:  the same as above;
4. `"source"`:       the same as above;
5. `"data"`:         the same as above.

If the `"source"` property is specified, and data content is available to the web service by using
the reference information provided, the `"data"` property MAY be omitted; otherwise the `"data"` 
property MUST be present.

The response MUST be an empty JSON object in case of success, otherwise an appropriate HTTP error 
response should be used.

The `/dataset` endpoint MAY accept the `DELETE` HTTP method. The body argument MUST be a string,
which refers to a VTL alias, known to the web service.

The response MUST be an empty JSON object in case of success, otherwise an appropriate HTTP error 
response should be used.

### `/structure` endpoint

This endpoint returns a description of one or more structures specified in the body argument with
their identifier. The body argument MUST BE an array of strings, each of which MUST be the identifier 
of a VTL structure, known to the web service.

The response MUST BE a JSON object, in which each key MUST BE the identifier of a VTL structure present
in the body argument and the corresponding value MUST BE a JSON object containing the following 
properties:

1. `"description"`: An optional string property describing the meaning of the structure;
2. `"components"`:  A mandatory JSON object property describing the components in the structure.

Each key of the `"components"` JSON object MUST have the component name as its key, and the
corresponding value MUST be a JSON object with the following properties:

1. `"role"`:     a mandatory string property specifying the role of the component, MUST be one of
                 `"identifier"`, `"measure"` and `"attribute"`;
2. `"subset"`:   an optional string property specifying if this component can only takes values from
                 a specified subset of the VTL represented variable domain;
3. `"nullable"`: an optional boolean property specifying that the component can take the null value;
                 this property is true by default for all component roles except for `"identifier"`,
                 in which case the property, if set, MUST have the value `false`.

The `/structure` endpoint MAY accept the `PUT` HTTP method. The body argument MUST be a JSON object
with the following properties:

1. `"identifier"`:  a mandatory string property specifying the identifier of the VTL structure;
2. `"description"`: the same as above;
3. `"components"`:  the same as above.

The response MUST be an empty JSON object in case of success, otherwise an appropriate HTTP error 
response should be used.

The `/structure` endpoint MAY accept the `DELETE` HTTP method. The body argument MUST be a string,
which refers to an indentifier of a VTL structure, known to the web service.

The response MUST be an empty JSON object in case of success, otherwise an appropriate HTTP error 
response should be used.

### `/variable` endpoint

This endpoint returns a description of one or more represented variables specified in the body argument
with their identifier. The body argument MUST BE an array of strings, each of which MUST be the 
identifier of a VTL represented variable, known to the web service.

The response MUST BE a JSON object, in which each key MUST BE the identifier of a represented
variable present in the body argument and the corresponding value MUST BE a JSON object describing a 
VTL represented variable with the following properties:

1. `"domain"`:      a mandatory string property specifying the valuedomain of the VTL represented variable;
2. `"description"`: an optional string property describing the meaning of the VTL represented variable.

### `/domain` endpoint

This endpoint returns a description of one or more value domain subsets, specified in the body argument
with their identifier. The body argument MUST BE an array of strings, each of which MUST be the identifier 
of a VTL value domain subset, known to the web service.

The response MUST BE a JSON object, in which each key MUST BE the identifier of a value domain present
in the body argument and the corresponding value MUST BE a JSON object containing the following properties:

1. `"parent"`:      a mandatory string property specifying the parent value domain subset;
2. `"description"`: an optional string property describing the meaning of the VTL valuedomain;

The JSON object MUST also have exactly one among these properties that describes the value domain:

* `"enumeration"`: a JSON array of values of a type compatible with the parent valuedomain, representing
                   all the admissible values for the VTL valuedomain or subset;
* `"condition"`:   a string containing a VTL expression used to restrict the admissible values in the value
                   domain, the `value` alias SHOULD be used as a placeholder for the actual value;
* `"externalref"`: a string containing an opaque description of a VTL value domain subset stored externally,
                   in case of SDMX codelists this SHOULD be a SDMX URN.

### `/execution` endpoint

This endpoint submits a VTL program to the web service for execution. The body argument MUST be a JSON
object, containing the following properties:

1. `"executionId"`: a mandatory integer property pointing to an identifier of a previously submitted VTL
                    program;
2. `"date"`:        a mandatory string property containing the date of the VTL program submission in the
                    ISO 8601 format;
3. `"hash"`:        a mandatory string property with the hexadecimal representation of a hash obtained by
                    applying a hash algorithm to a message composed by concatenating the `"executionId"`
                    and `"date"` properties, with the current datetime with a precision of milliseconds
                    expressed in the ISO 8601 format; the hash algorithm SHOULD BE SHA-3.

The web service implementation MAY reject requests that are too frequent or have an invalid hash. The
rejection MAY be silent, in which case the request is simply ignored, or MAY cause a HTTP 429 error.

The response MUST BE a JSON object containing the following properties:

1. `"executionId"`: a mandatory integer property matching the execution identifier of a previously
                    submitted VTL program;
2. `"date"`:        a mandatory string property containing the date of the VTL program submission in the
                    ISO 8601 format;
3. `"status"`:      a mandatory string property specifying the state of the job execution;

The `/execution` endpoint MUST accept the `PUT` HTTP method. The body argument MUST be a JSON array of
strings, each representing one or more lines of a complete VTL program that has at least one persistent
assignment statement.

The response MUST BE a JSON object containing the following properties:

1. `"executionId"`: a mandatory integer property representing an execution identifier for the submitted
                    submitted VTL program;
3. `"date"`:        a mandatory string property containing the date of the VTL program submission in
                    the ISO 8601 format.

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

Security MUST BE implemented when the web service implementation requires an user context in order to 
process the submitted VTL program. In this case, the web application container MUST BE configured to
forward the authentication token to the web service, or the application MUST support an authentication
and authorization mechanism, such as SPNEGO. HTTP headers SHOULD be used to provide those tokens.

Confidentiality SHOULD BE enforced by deploying the implementation behind a reverse proxy or on an 
application server that is configured to provide a X509 certificate with the https:// protocol.
