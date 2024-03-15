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

The taxonomy of categories is as follows:

1. Syntax errors (related to grammar inconsistencies in the VTL program)
2. Semantic errors (related to metadata verification)
3. Operator errors (related to specific operators)
4. Definition errors (related to <code>define</code> statements)
5. Data errors (related to storage or retrieval of actual data)<ol>

## Error message examples

This section provides a list of template examples for specific error messages. The templates
can be parameterized in order to include information about an actual error. The list is not
intented to be exhaustive, and additional categories MAY be added at a later time.

VTL engine implementations MAY choose to override any template definition and the textual
representation of any eventual parameter in each template.

Abbreviations are used in the template to signify specific VTL language elements, as follows:

| Placeholder | Meaning                                                                              |
|-------------|--------------------------------------------------------------------------------------|
| `%c`        | A component (represented with name, role and valuedomain)                            |
| `%r`        | The role of a component (component, identifier, measure, attribute, viral attribute) |
| `%d`        | A valuedomain or valuedomain subset                                                  |
| `%s`        | A data structure or set of unique components                                         |
| `%n`        | An alias to a dataset, component, ruleset or operator                                |
| `%o`        | A VTL operator's name or operator group's name                                       |

### Syntax errors

### Semantic errors

<table>
    <tr></tr>
    <tr>
        <th>Group</th>
        <th>Description</th>
        <th>Template examples</th>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>Expected exactly one component of a given role and/or domain</td>
        <td><pre>Required&nbsp;exactly&nbsp;one&nbsp;%r&nbsp;of&nbsp;domain&nbsp;%d,&nbsp;but&nbsp;found:&nbsp;%s
Required&nbsp;exactly&nbsp;one&nbsp;%r,&nbsp;but&nbsp;found:&nbsp;%s</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>Components of a given role and/or domain are not present in a structure</td>
        <td><pre>Required&nbsp;at&nbsp;least&nbsp;one&nbsp;%r&nbsp;of&nbsp;domain&nbsp;%d,&nbsp;but&nbsp;found:&nbsp;%s
Required&nbsp;at&nbsp;least&nbsp;one&nbsp;%r,&nbsp;but&nbsp;found:&nbsp;%s</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>One or more components were requested but are not present</td>
        <td><pre>Component&nbsp;%n&nbsp;not&nbsp;found&nbsp;in&nbsp;%s
Components&nbsp;%n,&nbsp;%n,&nbsp;%n...&nbsp;not&nbsp;found&nbsp;in&nbsp;%s</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>Identifiers must be invariant</td>
        <td><pre>%o&nbsp;cannot&nbsp;change&nbsp;identifiers&nbsp;%s
%o&nbsp;cannot&nbsp;change&nbsp;identifier&nbsp;%n&nbsp;to&nbsp;a&nbsp;%r
%o&nbsp;cannot&nbsp;change&nbsp;%c&nbsp;to&nbsp;an&nbsp;identifier
%o&nbsp;cannot&nbsp;change&nbsp;identifiers&nbsp;from&nbsp;%s&nbsp;to&nbsp;%s</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>Named components are not present in a structure</td>
        <td><pre>Components&nbsp;named&nbsp;%n,&nbsp;%n,&nbsp;...&nbsp;not&nbsp;found&nbsp;in&nbsp;%s
Component&nbsp;named&nbsp;%n&nbsp;not&nbsp;found&nbsp;in&nbsp;%s
Components&nbsp;%s&nbsp;not&nbsp;found&nbsp;in&nbsp;%s
Component&nbsp;%c&nbsp;not&nbsp;found&nbsp;in&nbsp;%s</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>An undefined alias is used in an expression</td>
        <td><pre>Alias&nbsp;%n&nbsp;is&nbsp;not&nbsp;bound&nbsp;to&nbsp;this&nbsp;transformation&nbsp;scheme
In&nbsp;%o,&nbsp;alias&nbsp;%n&nbsp;is&nbsp;not&nbsp;defined</pre></td>
    </tr>
    <tr></tr>
    <tr>
        <td></td>
        <td>Operator parameters or results have mismatched domains</td>
        <td><pre>Incompatible&nbsp;types&nbsp;in&nbsp;%o:&nbsp;%c&nbsp;and&nbsp;%c
Incompatible&nbsp;types&nbsp;in&nbsp;%o:&nbsp;%c&nbsp;and&nbsp;%d
Incompatible&nbsp;types&nbsp;in&nbsp;%o:&nbsp;%d&nbsp;and&nbsp;%c
Incompatible&nbsp;types&nbsp;in&nbsp;%o:&nbsp;%d&nbsp;and&nbsp;%d</pre></td>
    </tr>
</table>

### Operator errors

### Definition errors

### Data errors

# Serialization of VTL artifacts

Sometimes there may be a need to exchange VTL artifacts when operating in H2M or M2M mode.
This section defines a JSON schema that aims to provide an easy implementation of this 
serialization mechanism.

The schema encompasses all VTL artifacts included in the VTL User Manual section "Generic Model
for Variables and Value domains". VTL engines MAY not implement this specific scheme and/or
they MAY provide support for alternative or extended serialization schemes. All of the four
sections of the JSON schema are optional if the implementation does not require them.

## JSON scheme for VTL metadata

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
