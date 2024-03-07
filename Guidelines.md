# Introduction

## Guidelines version

These technical guidelines address implementation issues for VTL standard version 2.1.

These guidelines are meant to facilitate developers of VTL tools (engines, interpreters, 
translators...) in order to make implementation of such tools easier and speedier.

## Terminology

These guidelines MAY use keywords from RFC 2119. When one of those keywords SHALL be 
interpreted as in the specification, they MUST be written upper-case.

# Error messages

VTL error messages are categorized by a series of at least two numbers, separated by dashes.
The first number assigns the error message to one of the general categories. The second
number assigns the error message to a group inside each category. The meaning of additional
numbers beyond the second is implementation dependent.

The taxonomy of categories and groups is as follows:

<ol>
<li>Syntax errors (related to grammar inconsistencies in the VTL program)<ol>
    <li>Error group A</li>
    <li>Error group B</li>
</ol></li><li>Semantic errors (related to metadata verification)<ol>
    <li>Error group C</li>
    <li>Error group D</li>
</ol></li><li>Operator errors (related to specific operators)<ol>
    <li>Error group E</li>
    <li>Error group F</li>
</ol></li><li>Definition errors (related to <code>define</code> statements)<ol>
    <li>Error group G</li>
    <li>Error group S</li>
</ol></li><li>Data errors (related to storage or retrieval of actual data)<ol>
    <li>Error group G</li>
    <li>Error group S</li>
</ol></li>
</ol>

## Error message examples

This section provides a list of template examples for specific error messages. The templates
can be parameterized in order to include information about an actual error. The list is not
intented to be exhaustive, and additional categories MAY be added at a later time.

VTL engine implementations MAY choose to override any template definition and the textual
representation of any eventual parameter in each template.

### Syntax errors

### Semantic errors

### Operator errors

### Definition errors

### Data errors

# Serialization of VTL artifacts

Sometimes there may be a need to exchange VTL artifacts when operating in H2M or M2M mode.
This chapter defines a JSON schema that aims to provide an easy implementation of this 
serialization mechanism.

VTL engine implementors MAY not implement this specific scheme and they MAY provide support
for alternative or extended serialization schemes.

## JSON schemes

There are two JSON schemes, one for data and the other for metadata. The parts MAY be instanced
into a single JSON object, in this case each part instance SHOULD be a property named `data` 
or `metadata` respectively, or they MAY be provided as two different JSON files. In any case,
one or both MAY be omitted if the VTL engine can infer them from its running environment.

### JSON scheme for data

```json
{
}
```

### JSON scheme for metadata

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "description": "VTL Metadata JSON serialization",
    "$defs": {
        "vtl-id": {
            "type": "string",
            "pattern": "^[a-zA-Z][a-zA-Z0-9_]*$|^'.*'$"
        },
        "set-type": {
            "type": "array",
            "uniqueItems": true,
            "oneOf": [
                { "items": { "oneOf": [ { "type": "string" }, { "type": "null" } ] } },
                { "items": { "oneOf": [ { "type": "number" }, { "type": "null" } ] } }
            ]
        },
        "identifiable": {
            "type": "object",
            "properties": {
                "name": { "$ref": "#/$defs/vtl-id" },
                "description": { "type": "string" }
            },
            "required": [ "name" ]
        }
    },
    "type": "object",
    "properties": {
        "datasets": {
            "type": "array",
            "items": {
                "allOf": [ { "$ref": "#/$defs/identifiable" } ],
                "properties": {
                    "source": { "type": "string" },
                    "structure": { "$ref": "#/$defs/vtl-id" }
                },
                "required": [ "structure" ]
            }
        },
        "structures": {
            "type": "array",
            "items": {
                "allOf": [ { "$ref": "#/$defs/identifiable" } ],
                "properties": {
                    "components": {
                        "type": "array",
                        "items": {
                            "allOf": [ { "$ref": "#/$defs/identifiable" } ],
                            "properties": {
                                "role": {
                                    "type": "string",
                                    "enum": [ "Identifier", "Measure", "Attribute", "Viral Attribute" ]
                                },
                                "subset": { "$ref": "#/$defs/vtl-id" },
                                "nullable": { "type": "boolean" }
                            },
                            "required": [ "role" ]
                        }
                    }
                },
                "required": [ "components" ]
            }
        },
        "variables": {
            "type": "array",
            "items": {
                "allOf": [ { "$ref": "#/$defs/identifiable" } ],
                "properties": {
                    "domain": { "$ref": "#/$defs/vtl-id" }
                },
                "required": [ "domain" ]
            }
        },
        "domains": {
            "type": "array",
            "items": {
                "allOf": [ { "$ref": "#/$defs/identifiable" } ],
                "unevaluatedProperties": false,
                "oneOf": [
                    {
                        "properties": {
                            "externalRef": { "type": "string" }
                        },
                        "required": [ "externalRef" ]
                    }, {
                        "properties": {
                            "parent": { "$ref": "#/$defs/vtl-id" }
                        },
                        "required": [ "parent" ],
                        "oneOf": [{
                                "properties": {
                                    "restriction": { "$ref": "#/$defs/set-type" }
                                },
                                "required": [ "restriction" ]
                            }, {
                                "properties": {
                                    "enumerated": { "$ref": "#/$defs/set-type" }
                                },
                                "required": [ "enumerated" ]
                            }, {
                                "properties": {
                                    "described": { "type": "string" }
                                },
                                "required": [ "described" ]
                            }
                        ]
                    }
                ]
            }
        }
    }
}
```

# VTL web services

This chapter describes guidelines for implementors of web service applications providing access
to VTL engines, or VTL engines exposing web services.

## Architecture

A VTL engine MAY expose a set of web services, or an application layer MAY support interoperability
with a VTL engine instance located elsewhere. In the latter case, the web service MAY use any 
technology supported by the engine instance, such as JSON-RPC or CORBA, to transfer control between
the engine and the application layer.

A VTL web service implementation SHOULD provide ways to load VTL data, perform execution of a scheme 
and retrieve the results.

The payload and the results of calls to a VTL web service (where applicable) SHOULD be encoded in UTF-8.

## Internationalization

VTL web service implementations MAY provide ways to work with locale-sensitive content. When applicable,
the implementation SHOULD provide a way to select the locale of the data.

## Storage

Web services implementations MAY choose to rely on external storage technology in order to provide
data to the submitted request. Web services implementations MAY additionally support user storage of 
VTL artifacts. In this case any details regarding how to achieve this, and the list of required HTTP 
endpoints, are left to each implementation and MUST be documented. 

Web services implementations SHOULD BE able to map VTL aliases used in the submitted program to specific
data sources, either internal or external.

## Security

Security MUST BE implemented when the web service implementation requires an user context in order to 
process the submitted VTL program. In this case, the web application container MUST BE configured to
forward the authentication token to the web service, or the application MUST support an authentication
and authorization mechanism, such as Basic or Negotiate. HTTP headers SHOULD be used to provide those
tokens.

Encryption of communication between the user and the web service SHOULD BE enforced by deploying 
the implementation behind a reverse proxy or on an application server that is configured to 
provide a X509 certificate with the https:// protocol.
