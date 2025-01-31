---
title: "IKEv2 Support of ML-DSA"
abbrev: "IKEv2 ML-DSA"
category: info

docname: draft-sfluhrer-ipsecme-ikev2-mldsa-latest
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
    fullname: "Scott Fluhrer"
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
  RFC7296:
  RFC7427:
  RFC8420:
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

   This document describes how to use this algorithm for authentication within IKEv2 [RFC7296], as a replacement for the traditional signature algorithms (RSA, ECDSA).

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

   FIPS 204 also allows ML-DSA to be run in either determanistic or 'hedged' mode (where randomness is applied to the signature operation).
   We place no requirement on which is used; the implementation should select based on the quality of their random number source.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Changes

## Initial Negotiation

Both sides will need to inform the other that they implement ML-KEM signatures.
To do so, they will use the [RFC9593] mechanism to specify support for ML-KEM signatures, using the Multi-octet Announcement, with the following Algorithm Idenfifiers:

* ML-DSA-44 -> `30 0b 06 09 60 86 48 01 65 03 04 03 11`
* ML-DSA-65 -> `30 0b 06 09 60 86 48 01 65 03 04 03 12`
* ML-DSA-87 -> `30 0b 06 09 60 86 48 01 65 03 04 03 13`

TODO: Verify that these are the DER OID values that NIST has specified

If an implementation supports multiple ML-KEM parameter sets, it will list every parameter set it does support.

If the peer has not specified support for a parameter set in a SUPPORTED_AUTH_METHODS notify, that ML-KEM parameter set MUST NOT be used.

In addition, the SIGNATURE_HASH_ALGORITHMS Notify payload must also be sent (see [RFC7427]).

## Auth Payload Generation

If this implementation has an ML-DSA private key and the corresponding ML-DSA public certificate, and the peer has indicated support for the parameter set, the implementation will generate the AUTH payload as specified in section 3 of [RFC7427], using the ML-DSA algorithm as the signature algorithm, and using the fixed context string "IKEv2 AUTH" (`49 4b 45 76 32 20 41 55 54 48`).

That is, the implementation would take either the InitiatorSignedOctets string or the ResponderSignedOctets string (depending on whether they are the initiator or the responder, see section 2.15 of RFC 7296 for how those strings are constructed), compute the hash of that string (using one of the hashes listed in the peer's SIGNATURE_HASH_ALGORITHMS notify).
Then, the implementation hands that hash to ML-DSA to be signed (in pure mode, using the fixed context string "IKEv2 AUTH".
The resulting signature is the Signature Value.
Note that ML-DSA will hash the message to be signed again; this is expected.

TODO: I've defined the method two different ways - if we keep both, we need to make sure that they are equivalent.

## Auth Payload Validation

If this implementation receives a certificate with an ML-DSA public key, it will process the AUTH payload as implied by RFC7427.
That is, it will recover the hash function from the AlgorithmIdentifier ASN.1 object.
Then, it will take the ResponderSignedOctets or InitiatorSignedOctets string (depending on the whether they are the initiator or the responder), and then hash the string.
Then, it will before an ML-DSA verification, using the hash as the message, the public key from the certificate, the string "IKEv2 AUTH" as the context string, and the signature value from the AUTH payload.
If this signature verification fails, the implementation MUST reject the IKEv2 message.

# Security Considerations

The only security consideration that this adds is that the user must trust the strength of the ML-DSA signature operation.

# Discussion

We made several arbitrary design decisions in this draft.
This section contains the reasoning behind those design decision, and why we did not select the alternative possibilities.
Of course, these decisions are open to change - this is just a first cut.

## ML-DSA and Prehashing

The signature architecture within IKE was designed around RSA (and later extended to ECDSA).
In this architecture, the actual message (the SignedOctets) are first hashed (using a hash that the verifier has indicated support for), and then passed for the remaining part of the signature generation processing.
That is, it is designed for signature algorithms that first apply one of a number of hash functions to the message and then perform processing on that hash.
ML-DSA doesn't fit cleanly into this architecture; internally it adds a prepend to the message to be signed, and then does a fixed SHAKE256 (generating 64 bytes).

We see three ways to address this mismatch

The first is to note that ML-DSA has prehashed parameter sets; that is, ones designed to sign a message that has been hashed by an external source.
At first place, this would appear to be an ideal solution, however it turns out that there are a number of practical issues.
The first is that the prehashed version of ML-DSA would appear to be rarely used, and so it is not unlikely that support for it within crypto libraries may be lacking.
The second is that the public keys for the prehashed versions of ML-DSA parameter sets use different OIDs; this means that the certificates for IKEv2 would necessarily be different than certificates for other protocols (and some CAs might not support issuing certificates for prehashed ML-DSA, again because of the lack of use).
The third is that some users have expressed a desire not to use the prehashed parameter sets of ML-DSA.

The second is to note that, while IKEv2 normally acts this way, it doesn't always.
EdDSA has a similar constraint on not working cleanly with the standard 'hash and then sign' paradigm, and so the existing [RFC8420] provides an alternative method, which ML-DSA would cleanly fit into.
We could certainly adopt this same strategy; our concern would be that it might be more difficult for IKEv2 implementors which do not already have support for EdDSA.

The third way (which this current draft adopts) is what we can refer to as 'fake prehashing'; IKEv2 would generate the hash as current, but instead of running ML-DSA in prehash mode, we have ML-DSA sign it in pure mode as if it was the message.
This is a violation of the spirit, if not the letter of FIPS 204.
However, it is secure (assuming the hash function is strong), and fits in cleanly with both the existing IKEv2 architecture, and what crypto libraries provide.
Because this doesn't have any practical downsides, we opted for this option.

## ML-DSA Context

An additional feature that ML-DSA provides it allows the signer and the verifier to provide a 'context string'.
The signature would verify only if the context strings that are provided by both the signer and the verifier match.
The reason behind this is to ensure that if a public key is used for multiple purposes, a signature for one purpose cannot be used by an adversary in another.
In our case, if the same certificate where used to sign both IKEv2 and TLS exchanges, an adversary could not possibly take the signature from an TLS exchange and try to use it within an IKEv2 exchange.

This really is not a necessary security safe guard; the messages that are actually signed in both cases are distinct enough that an adversary could not actually take advantage of this, even without the protection.

However, given that ML-DSA does provide such a service, and it appears that crypto libraries do support a nonempty context, we cannot see a reason not to use it.

# IANA Considerations

This document has no IANA actions.

The additional OIDs that this uses have been defined by NIST and do not need to be registered by IANA

--- back

# Acknowledgments
{:numbered="false"}

No acknowledgements yet (no one has actually seen this draft until now)
