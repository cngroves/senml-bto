---
title: SenML Base Time Offset Attribute
abbrev: SenML BTO
docname: draft-groves-core-senml-bto-latest
date: 2016-10-17
category: std

ipr: trust200902
area: art
workgroup: CoRE Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
- ins: C. Groves
  name: Christian Groves
  organization: Huawei
  street: '' 
  city: ''
  code: ''
  country: Australia
  email: Christian.Groves@nteczone.com
- ins: W. Yang
  name: Weiwei Yang
  organization: Huawei
  street: '' 
  city: ''
  code: ''
  country: P.R.China
  email: tommy@huawei.com

normative:
  RFC2119:
  I-D.ietf-core-senml:
  
informative:
  


--- abstract

SenML {{I-D.ietf-core-senml}} defines a base time attribute and time value 
which is used to determine the time when a value is recorded. In 
some applications a SenML package will contain a series of records 
related to a constant sample time interval, e.g. once every 60 seconds. 
This means that the time attribute will be required for each record. 
This document defines a new “time offset” base attribute that allows a 
sender to include the time for the sample interval between records. If the “time offset” base attribute is used the sender will not send the time attribute for each record, minimising message and storage size.

--- middle

Introduction        {#introduction}
============

SenML currently defines the base time “bt” and time “t” attributes to indicate the time that each value in a SenML record is recorded. This means that for each record (value “v”) that a “t” attribute is required. The example from section 5.1.2/{{I-D.ietf-core-senml}} is copied below to illustrate a SenML package with multiple records.

~~~~

[ {"bn": "urn:dev:ow:10e2073a01080063",
      "bt": 1320067464,
      "bu": "%RH",
      "v": 21.2, "t": 0 },
      { "v": 21.3, "t": 10 },
      { "v": 21.4, "t": 20 },
      { "v": 21.4, "t": 30 },
      { "v": 21.5, "t": 40 },
      { "v": 21.5, "t": 50 },
      { "v": 21.5, "t": 60 },
      { "v": 21.6, "t": 70 },
      { "v": 21.7, "t": 80 },
      { "v": 21.5, "t": 90 },
   ... 

~~~~
{: #figmul title="SenML Package with multiple records"}

As can be seen in the example above there is a fixed offset between the data points of 10 seconds. 

This document proposes a base time offset “bto” attribute that is used to indicate the offset between datapoints when a fixed offset is used. This negates the need to include the “t” attribute for each data point. The example below takes the above example and applies the base time offset to it.

~~~~
[ {"bn": "urn:dev:ow:10e2073a01080063",
      "bt": 1320067464,
      “bto”: 10,
      "bu": "%RH",
      "v": 21.2},
      { "v": 21.3},
      { "v": 21.4},
      { "v": 21.4},
      { "v": 21.5},
      { "v": 21.5},
      { "v": 21.5},
      { "v": 21.6},
      { "v": 21.7},
      { "v": 21.5},
   ... 
~~~~
{: #figmulbto title="SenML Package with multiple records using base time offset"}

It can be seen that it results in a record and overall pack size reduction. The receiver of the record would use the base time “bt” for the initial data point, e.g. “v”: 21.2 would have a timestamp  of 1320067464. Each subsequent record would have a time stamp equal to the previous record plus the base time offset “bto”, e.g. “v”: 21.3 would have a timestamp of 1320067474, “v”: 21.4 would have a timestamp of 1320067484, and so on.


Terminology     {#terminology}
============
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

See {{I-D.ietf-core-senml}} for further definitions.

SenML Structure and Semantics        {#structure}
=============================

Base Attributes
---------------
This document adds an additional “base time offset” attribute.

Base Time Offset: The base time offset represents a fixed time offset between each record in a SenML pack. The first record represents time zero (or the base time if provided) and the offset is then applied to each subsequent record. Either a positive or negative base time offset is allowed.

Regular Attributes
------------------
This document modifies the behaviour of the time attribute.

Time: If the base time offset attribute is used in a SenML pack then the time attribute shall not be used in the individual records.

*Editor’s note: One theoretical use case may be to include the time attribute if the entry’s time deviates from the regular offset. It is not seen that such a corner case warrants the extra complexity.*

Considerations
--------------
To simplify implementation, the use of base time offset is limited to:

- Fixed time increasing or decreasing records from the time stamp associated with the first record in the SenML pack. This means that usages such as described by the 2nd example in 5.1.2/{{I-D.ietf-core-senml}} where negative time offset from time zero being provided by the last record in a SenML pack are not possible.

*Editor’s note: It could be possible to facilitate this use case by allowing a t=0 to be set on the last record with the use of a negative base time offset value. However again it’s not seen that such a corner case warrants the extra complexity in having to store values and calculate offsets at the end of transmission/reception.*

- It is not possible to for two records in the SenML pack to have the same time. For example the inclusion of a record for a voltage measurement followed by a current measurement would result in the records having times with a difference of the time offset.


JSON Representation (application/senml+json)  {#senmljson}
============================================
This document defines an additional SenML label (JSON object member name) as shown in {{jsonlabels}} below.

|             Name | label | Type    |
| Base Time Offset | bto   | Numer   |
{: #jsonlabels title="JSON SenML Labels"}

CBOR Representation (application/senml+cbor)   {#senmlcbor}
============================================
As per section 6/{{I-D.ietf-core-senml}}, for CBOR the string map key “bto” shall be used to indicate the use of the base time offset.

XML representation (application/senml+xml)   {#senmlxml}
==========================================
This document defines an addition XML attribute as shown in {{xmllabels}} below.

|            Name | XML  | Type    |
| Base Time Offset| bto  | double  |
{: #xmllabels title="XML SenML Labels"}

The RelaxNG schema for the XML is:

~~~~
  senml = element senml {
     attribute bn { xsd:string }?,
     attribute bt { xsd:double }?,
     attribute bv { xsd:double }?,
     attribute bs { xsd:double }?,
     attribute bu { xsd:string }?,
     attribute bver { xsd:int }?,
     attribute bto { xsd:double }?,

     attribute l { xsd:string }?,

     attribute n { xsd:string }?,
     attribute s { xsd:double }?,
     attribute t { xsd:double }?,
     attribute u { xsd:string }?,
     attribute ut { xsd:double }?,

     attribute v { xsd:double }?,
     attribute vb { xsd:boolean }?,
     attribute vs { xsd:string }?,
     attribute vd { xsd:string }?
   }

   sensml =
     element sensml {
       senml+
   }

   start = sensml
~~~~


EXI Representation (application/senml-exi)    {#senmlexi}
==========================================
As per clause 8/{{I-D.ietf-core-senml}} extensions are indicated through the use of an EXI schemaID options. The EXI schemaID options MUST be set to the value of "b" indicating the scheme provided in this specification.

The following is the XSD Schema to be used for strict schema guided EXI processing.

~~~~
<?xml version="1.0" encoding="utf-8"?>
   <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
   elementFormDefault="qualified"
   targetNamespace="urn:ietf:params:xml:ns:senml"
   xmlns:ns1="urn:ietf:params:xml:ns:senml">
     <xs:element name="senml">
       <xs:complexType>
         <xs:attribute name="bn" type="xs:string" />
         <xs:attribute name="bt" type="xs:double" />
         <xs:attribute name="bv" type="xs:double" />
         <xs:attribute name="bu" type="xs:string" />
         <xs:attribute name="bver" type="xs:int" />
         <xs:attribute name="bto" type="xs:double" />
         <xs:attribute name="l" type="xs:string" />
         <xs:attribute name="n" type="xs:string" />
         <xs:attribute name="s" type="xs:double" />
         <xs:attribute name="t" type="xs:double" />
         <xs:attribute name="u" type="xs:string" />
         <xs:attribute name="ut" type="xs:double" />
         <xs:attribute name="v" type="xs:double" />
         <xs:attribute name="vb" type="xs:boolean" />
         <xs:attribute name="vs" type="xs:string" />
         <xs:attribute name="vd" type="xs:string" />
       </xs:complexType>
     </xs:element>
     <xs:element name="sensml">
       <xs:complexType>
         <xs:sequence>
           <xs:element maxOccurs="unbounded" ref="ns1:senml" />
         </xs:sequence>
       </xs:complexType>
     </xs:element>
   </xs:schema>
~~~~ 

Security Considerations
=======================
No extra security issues are seen beyond those described in {{I-D.ietf-core-senml}}. 

IANA Considerations
===================
This document proposes one new entry in the IANA SenML label reegistry.

Label Name: Base Time Offset

Label: bto

CBOR: -

XML Type: Double

ID: b

Reference: This specification.

Acknowledgements
================
TBD



