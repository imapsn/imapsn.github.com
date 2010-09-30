---
layout: post
title: "Magic Signatures"
---

Info for PKCS n00bs.

Magic Envelope
: a data structure that wraps a body of data with information that
indicates the data's provenance. See
<http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-00.html>.

application/json
: A data structure serialization format. Defined in RFC4627 <http://tools.ietf.org/html/rfc4627>.

base64 encoding
: A mapping of n-bit groups from a byte stream to an alphabet. Defined
in RFC4648 <http://tools.ietf.org/html/rfc4648>

PKCS#1
: the first family of public key cryptography standards for the RSA
algorithm published by RSA Laboratories. For list of standards see
<http://en.wikipedia.org/wiki/PKCS>.

modulus
: n = pq, the product of two distinct large prime numbers

RSA public key
: a modulus and the public exponent (n,e)

RSA private key
: a modulus and the private exponent (n, d)

cryptographic scheme
: higher level algorithms that use cryptographic primitives to achieve
particular security goals

RSASSA-PKCS1-v1_5
: a cryptographic scheme for dealing with signatures. Defined in
<http://tools.ietf.org/html/rfc3447#section-8.2>

EMSA-PKCS1-v1_5
: an encoding method for signatures.  Defined in
<http://tools.ietf.org/html/rfc3447#section-9.2>.

Java Signature Algorithms
: see <http://download.oracle.com/javase/6/docs/technotes/guides/security/StandardNames.html#Signature>
SHA256withRSA is the algorithm used for magic envelopes.

Java KeyStore types
: see <http://download.oracle.com/javase/6/docs/technotes/guides/security/StandardNames.html#KeyStore>

