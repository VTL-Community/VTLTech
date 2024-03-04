# Introduction

## Guidelines version

These technical guidelines address implementation issues for VTL standard version 2.1.

These guidelines are meant to facilitate developers of VTL tools (engines, interpreters, 
translators...) in order to make implementation of such tools easier and speedier.

## Terminology

These guidelines MAY use keywords from RFC 2119. When one of those keywords SHALL be 
interpreted as in the specification, they MUST be written upper-case.

# Serialization of VTL artifacts

Sometimes there may be a need to exchange VTL artifacts when operating in H2M or M2M mode.
This chapter defines a JSON schema that aims to provide an easy implementation of this 
serialization mechanism.

VTL engine implementors MAY not implement this specific scheme and they MAY provide support for
alternative or extended serialization schemes.

## JSON schemes

There are two JSON schemes, one for data and the other for metadata. The parts MAY be instanced
into a single JSON object, in this case each part instance SHOULD be a property named `data` 
or `metadata` respectively, or they MAY be provided as two different JSON files. In any case,
one or both MAY be omitted if the VTL engine can infer them from its running environment.

### JSON scheme for data

```
```

### JSON scheme for metadata

```json
{
    "type": "object",
    "description": "VTL Metadata JSON serialization",
    "properties": {
        "datasets": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    },
                    "description": {
                        "type": "string"
                    },
                    "source": {
                        "type": "string"
                    },
                    "structure": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    }
                },
                "required": [ "name", "structure" ]
            }
        },
        "structures": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    },
                    "description": {
                        "type": "string"
                    },
                    "components": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "variable": {
                                    "type": "string",
                                    "pattern": "[a-zA-Z0-9_]+"
                                },
                                "role": {
                                    "type": "string",
                                    "enum": [ "Identifier", "Measure", "Attribute", "Viral Attribute" ]
                                },
                                "subset": {
                                    "type": "string",
                                    "pattern": "[a-zA-Z0-9_]+( not null)?"
                                }
                            },
                            "required": [ "variable", "role" ]
                        }
                    }
                },
                "required": [ "name", "components" ]
            }
        },
        "variables": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    },
                    "description": {
                        "type": "string"
                    },
                    "domain": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    }
                },
                "required": [ "name", "domain" ]
            }
        },
        "domains": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    },
                    "parent": {
                        "type": "string",
                        "pattern": "[a-zA-Z0-9_]+"
                    },
                    "externalRef": {
                        "type": "string"
                    },
                    "restriction": {
                        "type": "array",
                        "uniqueItems": true,
                        "items": {}
                    },
                    "enumerated": {
                        "type": "array",
                        "uniqueItems": true,
                        "items": {}
                    },
                    "described": {
                        "type": "string"
                    }
                },
                "required": [
                    "name",
                    "parent",
                    "externalRef"
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

A VTL web service implementation SHOULD provide at least these endpoints:

1. `/datapoints`: returns the contents of of one or more datasets;
2. `/structure`: returns the structure(s) of one or more datasets;
3. `/variable`: returns the description(s) of one or more variables;
4. `/domain`: returns the description of one or more valuedomain;
5. `/execution`: returns the status of a submitted VTL program or submits a new VTL program.

The default HTTP `Accept` request header SHOULD BE `Accept: application/json;q=1.0, text/plain;q=0.8, */*;q=0.6`.
Additional content-types not specified in this document MAY BE supported by a specific implementation
and SHALL BE selected through this header.

The returned result and the submitted payload (where applicable) SHOULD be encoded in UTF-8; each endpoint
SHOULD support the HTTP `GET` method, and the `PUT` and `DELETE` methods MAY also be supported in order to
create and delete resources.

Eventual parameters for the `PUT` and `DELETE` methods SHOULD BE encoded as a single JSON object and
passed through the request body.

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
