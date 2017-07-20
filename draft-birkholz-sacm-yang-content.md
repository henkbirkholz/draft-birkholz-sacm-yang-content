---
title: YANG subscribed notifications via SACM Statements
docname: draft-birkholz-sacm-yang-content-latest
date: 2017-07-19

ipr: trust200902
area: security
wg: SACM Working Group
kw: Internet-Draft
cat: std

coding: us-ascii
pi:
   toc: yes
   sortrefs: yes
   symrefs: yes
   comments: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: N. Cam-Winget
  name: Nancy Cam-Winget
  org: Cisco Systems
  email: ncamwing@cisco.com
  street: 3550 Cisco Way
  code: '95134'
  city: San Jose
  region: CA
  country: USA

normative:
  RFC2119:
#  RFC3635:
#  RFC1573:
#  I-D.greevenbosch-appsawg-cbor-cddl: cddl
#  I-D.ietf-sacm-architecture-13:
  I-D.ietf-netconf-subscribed-notifications: yangnote
  I-D.ietf-netconf-yang-push: yangpush
  I-D.ietf-sacm-information-model: sacmim
  I-D.ietf-mile-xmpp-grid: xmppgrid

informative:
#  RFC7632:
#  I-D.ietf-sacm-requirements: sacm-req

--- abstract

This document summarizes the data model designed at the IETF 99 Hackathon and is intended to grow in
to a definition of general XML SACM statements (and later JSON and CBOR, respectively) for virtually
every kind of Content Element (e.g. software identifiers, assessment guidance/results, ECA Policy
rules, VDD, etc.). The SACM Statement data structure is based on the Information Element (IE) definitions
provided by the SACM Information Model. The initial Content Element type transferred are YANG
Subscribed Notification acquired via YANG push. In combination with the Origin Metadata Annotation
defined in draft-ietf-netmod-revised-datastores the data model defined in this document will
ultimately be able to express collected endpoint characteristics, imperative guidance that define and
orchestrate assessment instructions, and also the declarative guidance for endpoint attributes.

--- middle

# Introduction

YANG modules are a powerful established tool to provide endpoint attributes (IE) with well-defined
semantics. YANG push {{-yangpush}} and the corresponding YANG subscribed notification {{-yangnote}} drafts
make use of these modules to create streams of notifications (telemetry) providing SACM content on
the data plane.
Correspondingly, filter expressions used in the context of YANG subscriptions constitute SACM
content that is imperative guidance consumed by SACM components on the management plane.

The SACM component illustrated in this draft incorporates a YANG Push client function and an
xmpp-grid publisher function. The output of the YANG Push client function is encapsulated in a SACM
Content Element envelope, which is again encapsulated in a SACM statement envelope. The
corresponding SACM statements are published via the xmpp-grid publisher function into a SACM Domain.

# Requirements notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in RFC
2119, BCP 14 {{RFC2119}}.

# Brokering of YANG push telemetry via SACM statements

Every SACM content is published into a SACM domain using a statement envelope/encapsulation. The
general structure of a Statement is based in the Information Element defintion in {{-sacmim}} and can be summarized as follows:

* a statement encapsulates statement-metadata and content-elements
* a content-element encapsulates content-metadata and SACM content

In the scope of this document, only one type of SACM content is covered: YANG output.
Correspondingly, only the minimal required structure of statements, statement-metadata,
content-elements, and content-metadata are defined. A complete XML schema definition of this
minimal statement can be found in Appendix A.

# Encapsulation of YANG notifications in SACM content-elements

A YANG notification is associated with a set of YANG specific metadata. Hence, a YANG notification
published to a SACM Domain MUST be encapsulated with its corresponding metadata in a Content
Element as defined below.

YANG output that is SACM content is represented as an element defintion included in the content
choice of the content-element.

~~~~ XSD
<CODE BEGINS>
<xs:complexType name="content-element">
  <xs:sequence>
    <xs:element name="content-metadata" type="content-metadata" maxOccurs="unbounded"/>
    <xs:choice>
      <xs:element name="yang-output" type="yang-output" />
        <!-- There is only one element here now, but virtually every other content choice
             will go here, i.e. data models, such as OVAL, SCAP, SWID, etc. -->
    </xs:choice>
  </xs:sequence>
</xs:complexType>
<CODE ENDS>
~~~~

## Enumeration definition for content-type

One occurrence of the yang-output element MUST be instantiated in the content-metadata element if
YANG push output is to be transferred. Also, the content-type must be set to the enumeration value
"yang-output", respectively.

In general, the list of content-type enumerations is including every subject as defined in the SACM
Information Model. For the scope of this document, the list of potential content is reduced to
"yang-output" only.

~~~~ XSD
<CODE BEGINS>
<xs:simpleType name="content-type">
  <xs:restriction base="xs:string">
    <xs:enumeration value="yang-output" />
       <!-- There is only one type here now, but virtually every other content-type
            will go here, i.e. data models, such as OVAL, SCAP, SWID, etc. -->
  </xs:restriction>
</xs:simpleType>
<CODE ENDS>
~~~~

## Element definition for content-metadata

The list of optional elements included in content-metadata will incorporate any every potential
metadata type. For the scope of this document, the list of elements is also limited to the minimal
required set of metadata elements and the yang-output metadata element to support the encapsulation
of NETCONF subscribed notifications and YANG query result. As defined above, one occurrence of the
yang-output element has to be included in the content-metadata element.

The general content-metadata elements are illustrated in the Appendix A.

~~~~ XSD
<CODE BEGINS>
<xs:complexType name="content-metadata">
  <xs:sequence>
    <xs:element name="content-element-guid" type="content-element-guid"/>
    <xs:element name="content-creation-timestamp" type="content-creation-timestamp"/>
    <xs:element name="content-topic" type="content-topic"/>
    <xs:element name="content-type" type="content-type"/>
    <xs:element name="data-source" type="data-source" minOccurs="0"/>
    <xs:element name="data-origin" type="data-origin" minOccurs="0"/>
    <xs:element name="relationship" type="relationship" minOccurs="0" maxOccurs="unbounded"/>
    <xs:element name="yang-output-metadata" type="yang-output-metadata" minOccurs="0"/>
  </xs:sequence>
</xs:complexType>
<CODE ENDS>
~~~~

## Definition of the yang-output-metadata element included in content-metadata

The composition of metadata that can be associated with a XML NETCONF result depends on multiple
factors:

* acquisition method: query / subscription
* encoding: XML / JSON / CBOR
* subscription interval: periodic / on-change
* filter-type: xpath / subtree

Additionally, the actual filter expression (or in future iterations of this work a referencing label, such as a URI, UUID or other composed identifier) has to be included in the content-metadata.

~~~~ XSD
<CODE BEGINS>
<xs:complexType name="yang-output-metadata">
  <xs:sequence>
    <xs:choice maxOccurs="1">
      <xs:element name="yang-query" type="yang-query" />
      <xs:element name="yang-subscribe" type="yang-subscribe" />
    </xs:choice>
    <xs:element name="encoding" type="yang-encoding" />
    <xs:element name="module-names" type="module-name" minOccurs="0" maxOccurs="unbounded" />
  </xs:sequence>
</xs:complexType>

<xs:complexType name="yang-subscribe">
  <xs:restriction base="xs:NMTOKEN">
    <xs:enumeration value="periodic" />
    <xs:enumeration value="on-change" />
  </xs:restriction>
  <xs:restriction base="xs:NMTOKEN">
    <xs:enumeration value="xpath" />
    <xs:enumeration value="subtree" />
  </xs:restriction>
<xs:complexType>

<xs:simpleType name="filter-expression">
  <xs:restriction base="xs:string" />
</xs:simpleType>

<xs:simpleType name="yang-query">
  <xs:restriction base="xs:string" />
</xs:simpleType>

<xs:simpleType name="yang-encoding">
  <xs:restriction base="xs:NMTOKEN">
    <xs:enumeration value="netconf" />
    <xs:enumeration value="restconf" />
    <xs:enumeration value="comi" />
  </xs:restriction>
</xs:simpleType>

<xs:simpleType name="module-name">
  <xs:restriction base="xs:string" />
</xs:simpleType>
<CODE ENDS>
~~~~

# SACM Component Composition

A SACM Component able to process YANG subscribed notifications requires at least two functions:

* a YANG push client function {{-yangpush}}, {{-yangnote}}
* an xmpp-grid provider function {{-xmppgrid}}

Orchestattion of functions inside a component, their discovery as capabiliites and the internal
communication of SACM content inside a SACM component is out of scope of this document for now.

#  IANA considerations

This document includes requests to IANA.

#  Security Considerations

TBD

#  Acknowledgements

Christoph Vigano, Guangying Zheng, Eric Voit, Alexander Clemm

#  Change Log

First version -00

# Contributors

--- back

# Minimal SACM Statement Definition for YANG Output

The definitions of statements, statement-metadata, content-element, and content-metadata are provided by the SACM Information Model {{-sacmim}}.

Due to the stripping down of content-elements to YANG output, the enumerations still included in the
relationship type are not able to point to other content actually.

~~~~ XSD
<CODE BEGINS>
{::include partial-poc-sacm-data-model.xsd}
<CODE ENDS>
~~~~
