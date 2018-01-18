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
  I-D.ietf-netconf-subscribed-notifications: yangnote
  I-D.ietf-netconf-yang-push: yangpush
  I-D.ietf-netconf-notification-messages: yanghead
  I-D.ietf-sacm-information-model: sacmim
  I-D.ietf-mile-xmpp-grid: xmppgrid
informative:

--- abstract

This document summarizes a subset of the emerging generic SACM Data Model for
inter-component distribution of SACM Content in and between SACM Domains. The
subset defined in this document is covering every information element that can
be acquired using YANG based protocols, i.e. NETCONF, RESTCONF, COMI or derived
mechanisms that transfer YANG modeled data, such as MUD. As subscriptions to data
origins in a SACM domain are one of the architectural corner-stones of the SACM
architecture, this document recommends the use of YANG Push, YANG subscribed
Notifications and corresponding Notification Headers and Bundles. Analogously, a
mapping of Notification Header content to SACM Metadata is provided in this
document.

<!--
provided by the SACM Information Model. The initial Content Element type transferred are YANG
Subscribed Notification acquired via YANG push. In combination with the Origin Metadata Annotation
defined in draft-ietf-netmod-revised-datastores the data model defined in this document will
ultimately be able to express collected endpoint characteristics, imperative guidance that define and
orchestrate assessment instructions, and also the declarative guidance for endpoint attributes.
-->
--- middle

# Introduction

This document defines an XML encoding of SACM Statements that contain SACM
Content composed of YANG modeled data (i.e. NETCONF messages).
Correspondingly, this documents provides a standardized mapping to derive SACM
Metadata from YANG Subscribed Notifications {{-yangnote}} using Notification
Message Headers and Bundles {{-yanghead}} content.

Every message defined in the generic SACM Data Model is a SACM Statement. The
SACM Statement structure is provided by the SACM Information Model. In
consequence, a SACM Statement is an Information Element not acquired by, but
created by SACM Components for inter-component distribution of SACM Content
(Information Elements on the Data Plane that represent information about Target
Endpoints (TE) or Guidance. Examples include: software identifiers, assessment
guidance/results, ECA Policy rules, or VDD).

YANG modules are a powerful established tool to provide Information Elements
about Target Endpoints with well-defined semantics. YANG Push {{-yangpush}} and
the corresponding YANG Subscribed Notifications {{-yangnote}} drafts make use of
these modules to create streams of notifications (YANG telemetry). Subscriptions
to YANG data stores or YANG streams are Data Sources that provide Information
Elements that can be acquired by SACM Collectors to provide SACM Content on the
Data Plane.

Analogously, filter expressions used in the context of YANG subscriptions
constitute SACM Content that is Imperative Guidance consumed by SACM Components
on the Management Plane in order to create YANG telemetry.

In this document (not including the abstract, of course), terms that are
Capitalized or prefixed with SACM are defined in the SACM Terminology document.

<!--
A SACM Component that collects Target Endpoint Information Elements (TI)
The SACM component illustrated in this draft incorporates a YANG Push client function and an
xmpp-grid publisher function. The output of the YANG Push client function is encapsulated in a SACM
Content Element envelope, which is again encapsulated in a SACM statement envelope. The
corresponding SACM statements are published via the xmpp-grid publisher function into a SACM Domain.
-->

# Requirements notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in RFC
2119, BCP 14 {{RFC2119}}.

# Brokering of YANG Push Telemetry via SACM Statements

Every SACM Content is published into a SACM Domain using a statement
envelope/encapsulation. The general structure of a SACM Statement is based on
the Information Element definition found in {{-sacmim}} and can be summarized as
follows:

* a sacm-statement encapsulates statement-metadata and content-elements
* a content-element encapsulates content-metadata and SACM Content

In the scope of this document, only one type of SACM Content is covered: YANG
modeled data. Correspondingly, the minimal required structure of
statements, statement-metadata, content-elements, and content-metadata are
defined. A complete XML schema definition of this subset of the generic SACM
Data Model can be found in Appendix A.

# Encapsulation of YANG notifications in SACM content-elements

A YANG notification is associated with a set of YANG specific metadata as
defined in {{-yanghead}}. Hence, SACM Content that is derived from a YANG
notification published to a SACM Domain MUST be encapsulated with its
corresponding Metadata in a content-element as defined below.

YANG output that is SACM Content MUST be represented according to the XSD
definition included in the content choice of the content-element.

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

An occurrence of the yang-output element MUST be instantiated in the
content-metadata element, if YANG Push output is to be transferred. Also, the
content-type MUST be set to the enumeration value
"yang-output", respectively.

In general, the list of content-type enumerations is including every subject as
defined in the SACM Information Model. Regarding the definition of the subset
of the generic SACM Data Model provided by this document, the list of potential
content-types is reduced to "yang-output". Please note, that the complete
generic SACM Data Model includes additional content-type enumerations next to
the definition provided by this document.

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

The list of optional elements included in content-metadata will incorporate any
every potential metadata type. For the scope of this document, the list of
elements is also limited to the minimal required set of metadata elements and
the yang-output metadata element to support the encapsulation of NETCONF
encoded subscribed notifications or YANG query result. As defined above, one
occurrence of the yang-output element has to be included in the
content-metadata element.

A more complete content-metadata element definition is illustrated in the
Appendix A.

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

The composition of metadata that can be associated with a XML NETCONF result
depends on multiple factors:

* acquisition method: query / subscription
* encoding: XML # more content encodings will be supported as indicated by the
  definition
* subscription interval: periodic / on-change
* filter-type: xpath / subtree

Additionally, the actual filter expression (or in future iterations of this
work, a referencing Label, such as a URI, UUID or other composed identifier)
has to be included in the content-metadata.

~~~~ XSD
<CODE BEGINS>
<xs:complexType name="yang-output-metadata">
  <xs:sequence>
    <xs:choice maxOccurs="1">
      <xs:element name="yang-query" type="yang-query-value" />
      <xs:element name="yang-subscribtion" type="yang-subscribtion-type" />
    </xs:choice>
    <xs:element name="encoding" type="yang-encoding" />
    <xs:element name="module-names" type="module-name" minOccurs="0" maxOccurs="unbounded" />
    <xs:element name="filter-expression" type="filter-expression-value" minOccurs="0" maxOccurs="1" />
  </xs:sequence>
</xs:complexType>

<xs:complexType name="yang-subscribtion-type">
  <xs:restriction base="xs:NMTOKEN">
    <xs:enumeration value="periodic" />
    <xs:enumeration value="on-change" />
  </xs:restriction>
  <xs:restriction base="xs:NMTOKEN">
    <xs:enumeration value="xpath" />
    <xs:enumeration value="subtree" />
  </xs:restriction>
<xs:complexType>

<xs:simpleType name="filter-expression-value">
  <xs:restriction base="xs:string" />
</xs:simpleType>

<xs:simpleType name="yang-query-value">
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

# Mapping of YANG Bundled Notifications to SACM Metadata

{{-yanghead}} includes the following definition:

~~~~

       yang-data bundled-message
          +-- bundled-message-header
          |  +-- message-time                yang:date-and-time
          |  +-- message-id?                 uint32
          |  +-- previous-message-id?        uint32
          |  +-- message-generator-id?       string
          |  +-- signature?                  string
          |  +-- notification-count?         uint16
          +-- notifications*
             +-- notification-header
             |  +-- notification-time        yang:date-and-time
             |  +-- subscription-id*         uint32
             |  +-- notification-id?         uint32
             |  +-- module?                  yang-identifier
             |  +-- notification-type?       notification
             |  +-- observation-domain-id?   string
             +-- receiver-record-contents?
~~~~

The corresponding mapping MUST be used when deriving SACM Content Metadata for
content-metadata items from YANG modeled data corresponding to YANG
Notification Message Headers and Bundles:

~~~~
        notification-time -> content-creation-timestamp
        subscription-id + (observation-domain-id OR "SACM Component Label") -> content-element-guid
        module -> module-names
        notification-type -> yang-subscribtion-type
        receiver-record-contents -> content-elements
~~~~

If there are more than one receiver-record-contents instanced included in the
received Notification Message Bundle, multiple content-elements MUST be
instanciated, accordingly.

The following mapping MUST be used when deriving SACM Statement Metadata (see
Appendix A) statement-metadata items representing  NETCONF instances adhering
to the definition of YANG Notification Message Headers and Bundles:

~~~~
        message-id -> statement-guid
        "SACM Component Label" -> data-origin
        message-time -> statement-creation-timestamp
        "SACM Component Publictation Time" -> statement-publish-timestamp
        statement-type -> "Observation"
~~~~

"SACM Component Publicatation Time" can only be inferred by the SACM Component
using its "most trustworthy source of time".

If there is not receiver-record-contents included in the YANG notification, a
SACM Component MUST NOT publish a corresponding SACM Statement to the SACM
Domain.

# SACM Component Composition

A SACM Component able to process YANG subscribed notifications requires at
least two functions:

* a SACM Function supporting YANG Push and YANG Notification Headers and
* Bundles function {{-yangpush}}, {{-yangnote}}, and
* an xmpp-grid provider function {{-xmppgrid}}

Orchestration of functions inside a component, their discovery as capabilities
and the internal distribution of SACM Content inside a SACM Component is out of
scope of this document. # for now

#  IANA considerations

This document includes requests to IANA.

#  Security Considerations

TBD

#  Acknowledgements

Christoph Vigano, Guangying Zheng, Eric Voit, Alexander Clemm

#  Change Log

First version -00

Second version -02
* generalized the content of the document, detaching it from the implementation
  created at the Hackaton of IETF 99
* included a mapping of the -03 version of the YANG Notification Headers and
  Bundles draft to this draft

# Contributors

Eric Voit

--- back

# Minimal SACM Statement Definition for YANG Output

The definitions of statements, statement-metadata, content-element, and
content-metadata are provided by the SACM Information Model {{-sacmim}}.

Due to the stripping down of content-elements to YANG output, the enumerations
still included in the relationship-type are not able to point to other types of
content in the scope of this document, but are able to reference other
content-types in the scope of the generic SACM Data Model.

~~~~ XSD
<CODE BEGINS>
{::include partial-poc-sacm-data-model.xsd}
<CODE ENDS>
~~~~
