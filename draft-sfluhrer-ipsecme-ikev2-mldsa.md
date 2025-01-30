---
title: "IKEv2 Support of ML-DSA"
abbrev: "IKEv2 ML-DSA"
category: info

docname: draft-sfluhrer-ikev2-mldsa-supprrt-latest
submissiontype: IETF
consensus: true
date:
v: 3
area: "sec"
workgroup: "IPsecME"
stand_alone: true
ipr: trust200902
keyword:
 - ML-DSA
coding: UTF-8
venue:
  group: "IP Security Maintenance and Extensions"
  type: "Security Area"
  mail: "cfrg@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ipsecme/"
  github: "sfluhrer/ikev2-mldsa-support"
  latest: "https://sfluhrer.github.io/ikev2-mldsa-support/draft-sfluhrer-ikev2-mldsa-support.html"

author:
 -
    fullname: "sfluhrer"
    organization: Cisco Systems
    email: "sfluhrer@cisco.com"

normative:

  FIPS204:
    target: https://doi.org/10.6028/NIST.FIPS.204
    title: Module-Lattice-Based Digital Signature Standard
    seriesinfo:
      "NIST": "FIPS 204"
    date: August 2024

informative:
  RFC9593:

--- abstract

One IPsec area that would be impacted by Cryptographically Relevant
Quantum Computer (CRQC) is IKEv2 authentication based on traditional
asymmetric cryptograph algorithms: e.g RSA, ECDSA; which are widely
deployed authentication options of IKEv2.
NIST has recently standardized ML-DSA, which is a signature algorithm believed to be secure against Quantum Computers.
This document describes how to use ML-DSA with IKEv2 as an auhentication scheme.

--- middle

# Introduction

   A Cryptographically Relevant Quantum Computer (CRQC) could break
   traditional asymmetric cryptograph algorithms: e.g RSA, ECDSA; which
   are widely deployed authentication options of IKEv2.
   NIST has recently published the postquantum digitial signature algorithm ML-DSA [FIPS204].

   This document describes how to use this algorithm for authentication within IKEv2, as a replacement for the traditional signature algorithms (RSA, ECDSA).

   Each IPsec peer announce the support for ML-DSA authentication via
   SUPPORTED_AUTH_METHODS notification as defined in [RFC9593],
   generates and verifies AUTH payload using ML-DSA.

## Background on ML-DSA

   ML-DSA (as specified in FIPS 204) is a signature algorithm that is believed to be secure against attackers who have a Quantum Computer available to them.
   There are three strengths defined for it (with the parameter sets being known as ML-DSA-44, ML-DSA-65 and ML-DSA-87).
   In addition, for each defined parameter set, there are two versions, the 'pure' version (where ML-DSA directly signs the message) and a 'prehashed' version (where ML-DSA signs a hash that was computed outside of ML-DSA).
   For this protocol, we will always use the pure version.

   In addition, ML-DSA also has a 'context' input, which is a short string that is common to the sender and the recceiver.
   It is intended to allow for domain separation between separate uses of the same public key.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Changes

## Initial Negotiation

Both sides will need to inform the other that they implement ML-KEM signatures.
To do so, they will use the [RFC9593] mechanism to specify support for ML-KEM signatures, using the Multi-octet Announcement, with the following Algorithm Idenfifiers:

* ML-DSA-44 -> `30 0b 06 09 60 86 48 01 65 03 04 03 11`
* ML-DSA-65 -> `30 0b 06 09 60 86 48 01 65 03 04 03 12`
* ML-DSA-87 -> `30 0b 06 09 60 86 48 01 65 03 04 03 13`

TODO: Verify that these are the DER OID values are what NIST has specified

If an implementation supports multiple ML-KEM parameter sets, it will list every parameter set it does support.

If the peer has not specified support for a parameter set in a SUPPORTED_AUTH_METHODS notify, that ML-KEM parameter set MUST NOT be used.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
