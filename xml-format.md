# XML Event Format for CloudEvents

## Abstract

This format specification for CloudEvents defines how events are expressed
in XML documents.

## Status of this document

This document is a working draft.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Attributes](#2-attributes)
3. [Data](#3-data)
4. [Envelope](#4-envelope)
5. [XML Batch Format](#5-xml-batch-format)
6. [References](#6-references)

## 1. Introduction

[CloudEvents][ce] is a standardized and protocol-agnostic definition of the
structure and metadata description of events. This specification defines how
elements defined in the CloudEvents specification are to be represented using
[Extensible Markup Language (XML)](xml-spec) documents.

* The [associated schema][xml-schema] formalizes the structure of the XML
document.

* The [Attributes](#2-attributes) section describes the representation and
data type mappings for CloudEvents context attributes.

* The [Data](#3-data) section defines the container for the data portion of a
CloudEvent.

* The [Envelope](#4-envelope) section defines an XML container for CloudEvents
attributes and an associated media type.

* The [Batch](#5-xml-batch-format) section describes how multiple CloudEvents
may be packaged into a single XML document.

### 1.1. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119][rfc2119].

### 1.2 Approach

This specification promotes an attribute heavy XML document style; such a model
presents advantages in terms of parsing/processing along with a more compact
XML document representation.

Attribute and Element names are deliberately terse for performance reasons.

Preservation of attribute value type information is supported allowing
custom extensions to be communicated without value type loss.

The model is formalized by this [XML Schema][xml-spec].

## 2. Attributes

The CloudEvents type system is mapped to XML schema types as follows :

| CloudEvents   | XML | Type Tag |
| ------------- | ---------------------------------------------------------------------- | -- |
| Boolean       | [xs:boolean][xml-primitives] | B |
| Integer       | [xs:int][xml-primitives] | I |
| String        | [xs:string][xml-primitives] | S |
| Binary        | [xs:base64Binary][xml-primitives] | BI |
| URI           | [xs:anyURI][xml-primitives] following [RFC 3986][rfc3986]| U |
| URI-reference | [xs:anyURI][xml-primitives] following [RFC 3986][rfc3986] | UR |
| Timestamp     | [xs:dateTime][xml-primitives] following [RFC 3339][rfc3339] | T |

Each concrete CloudEvent context attribute is represented using an XML element with the following
shape:

``` xml
<Attr n="id" v="XXX-YYY-ZZZ-0000" t="S">
```

where :

* _n_ is the _name_ of the context attribute
* _v_ is the _value_ of the context attribute
* _t_ is the _type_ of the attribute value acording to the _Tag_ defined in the table above.

See section below on special handling for `specversion`

## 2.1 Specification Version

To better support streaming style processing the specification version
(`specversion` context attribute) is communicated as an attribute of the enclosing CloudEvent
enveloping element.

``` xml
<CloudEvent specver="1.0">
</CloudEvent>
```

## 3. Data

The `Data` portion of a CloudEvent is contained is a `<Data></Data>` element and supports
Binary, Text, and XML Data. Each represention is wrapped in a discriminating element:

### Binary Data

Carried in an element with an expected type of `xs:base64Binary`

``` xml
<Data>
    <Bin>... base64 encoded string ...</Bin>
</Data>
```

### Text Data

Carried in an element with an expected type of `xs:string`

``` xml
<Data>
    <Txt>Hello World</Txt>
</Data>
```

### XML Data

Carried in an element with an expected type of `xs:any`

``` xml
<Data>
    <Xml>
        <SomeElement>
            ....
        </SomeElement>
    </Xml>
</Data>
```

## 4. Envelope

Each CloudEvent is wholly represented as an XML element, the root xml
document element is `<CloudEvent ver="n.n">` with the `ver` attribute
used to define the specification version being followed.

Such a representation MUST use the media type `application/cloudevents+xml`.

The enveloping element contains:

* A set of context attributes.
* An optional data element.

eg:

``` xml
<CloudEvent ver="1.0" >
    <Attr n="id" v="000-1111-2222" t="S">
    <Attr n="time" v="2020-03-19T12:54:00-07:00" t="T">
    <Attr n="type" v="SOME.EVENT.TYPE" t="S"/>
    ....
    <Data>
        <Txt>Now is the winter of our discount tents...</Txt>
    </Data>
</CloudEvent>
```

## 5. XML Batch Format

In the _XML Batch Format_ several CloudEvents are batched into a single XML
document. The document comprises a list of elements in the XML Format.

Although the _XML Batch Format_ builds on top of the _XML Format_, it is
considered as a separate format: a valid implementation of the _XML Format_
doesn't need to support it. The _XML Batch Format_ MUST NOT be used when only
support for the _XML Format_ is indicated.

An XML Batch of CloudEvents MUST use the media type
`application/cloudevents-batch+xml`.

Example:

``` xml
<CloudEventBatch>
    <CloudEvent ver="1.0">
       ....
    </CloudEvent>
    <CloudEvent ver="1.0">
       ....
    </CloudEvent>
    ....
</CloudEventBatch>
```

An example of an empty batch of CloudEvents (typically used in a response, but also valid in a request):

```xml
<CloudEventBatch />
```

## 6. References

[CloudEvents XML Schema][xml-schema]

[ce]: ./spec.md
[ce-types]: ./spec.md#type-system
[xml-schema]: ./spec.xsd
[xml-format]: ./xml-format.md
[xml-spec]: https://www.w3.org/TR/2008/REC-xml-20081126/
[xml-primitives]: https://www.w3.org/TR/xmlschema-2/#built-in-primitive-datatypes
[rfc2119]: https://tools.ietf.org/html/rfc2119
[rfc3339]: https://www.ietf.org/rfc/rfc3339.txt
[rfc3986]: https://tools.ietf.org/html/rfc3986
