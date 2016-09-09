---
docname: draft-groves-core-senml-bto-latest
title: SenML Base Time Offset Attribute
abbrev: SenML BTO
date: 2016-09-09
category: proposed standard

ipr: trust200902
area: ART
workgroup: Core Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: C. Groves
    name: Christian Groves
    organization: Huawei
    email: Christian.Groves@nteczone.com

normative:
  RFC2119:
  I-D.ietf-core-senml:

informative:




--- abstract

SenML {{I-D.ietf-core-senml}} defines a base time attribute and time value 
which is used to determine the time when a value is recorded. In 
some applications a SenML record will contain a series of values 
related to a constant sample time interval, e.g. once every 60 seconds. 
This means that the time attribute will be required for each sample. 
This document defines a new “time offset” base attribute that allows a 
sender to include the time for the sample interval between values. If the 
“time offset” base attribute is used the sender will not send the time 
attribute for each value, minimising message and storage size.

--- middle

Introduction        {#introduction}
============

SenML currently defines the base time “bt” and time “t” attributes 
to indacte the time that each value in a SenML record is recorded. 
This means that for each record (value “v”) that a “t” attribute is 
required. The example from section 5.1.2/[RFC senML] is copied below 
to illustrate a SenML package with multiple records.

~~~~~~~~~~

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

~~~~~~~~~~
{: #figops title="SenML Package with multiple records"}

As can be seen in the example above there is a fixed offset between the 
data points of 10 seconds. 

The document proposes a base time offset “bto” attribute that is used to 
indicate the offset between datapoints when a fixed offset is used. This 
negates the need to include the “t” attribute for each data point. The 
example below takes the above example and applies the base time offset to 
it.

~~~~~~~~~~
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
~~~~~~~~~~
{: #figops title="SenML Package with multiple records using base time offset"}

It can be seen that it results in a record size reduction. The receiver of the 
record would use the base time “bt” for the initial data point, e.g. “v”: 21.2 
would have a timestamp  of 1320067464. Each subsequent data point would have a 
time stamp equal to the previous data point plus the base time offset “bto”, e.g. 
“v”: 21.3 would have a timestamp of 1320067474, “v”: 21.4 would have a timestamp 
of 1320067484, and so on.


Terminology     {#terminology}
============

SenML Structure and Semantics        {#structure}
=============================

Base Attributes
---------------

Regular Attributes
------------------

Considerations
--------------

JSON Representation (application/senml+json)
============================================

CBOR Representation (application/senml+cbor)
============================================

XML representation (application/senml+xml)
==========================================

EXI Representation (application/senml-exi)
==========================================

Security Considerations
=======================

IANA Considerations
===================

Acknowledgements
================





--- back



