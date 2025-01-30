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

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
