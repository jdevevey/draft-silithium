---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Silithium - A Compact, Efficient and Non-separable Hybrid Signature"
abbrev: "Silithium"
category: std

docname: draft-devevey-cfrg-silithium-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: Security
workgroup: CFRG
keyword:
 - post-quantum
 - cryptography
 - hybridization
 - signature
venue:
  group: Cryptography Forum
  type: Research Group
  mail: cfrg@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/cfrg/
  github: jdevevey/draft-silithium

author:
 -
    fullname: Julien Devevey
    organization: ANSSI
    email: julien.devevey@ssi.gouv.fr
 -
    fullname: Maxime Roméas
    organization: ANSSI
    email: maxime.romeas@ssi.gouv.fr
 -
    fullname: Morgane Guerreau
    organization: PQShield
    email: morgane.guerreau@pqshield.com

normative:
  RFC6090:
  RFC5480:
  SEC1:
    title: "SEC 1: Elliptic Curve Cryptography"
    date: May 21, 2009
    author:
      - org: "Certicom Research"
    target: https://www.secg.org/sec1-v2.pdf
  SEC2:
    title: "SEC 2: Recommended Elliptic Curve Domain Parameters"
    date: January 27, 2010
    author:
      - org: "Certicom Research"
    target: https://www.secg.org/sec2-v2.pdf
  FIPS.204:
    title: "Module-Lattice-Based Digital Signature Standard"
    date: August 13, 2024
    author:
      - org: "National Institute of Standards and Technology (NIST)"
    target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.204.pdf
    seriesinfo:
      "FIPS PUB": "204"

informative:
  RFC9794:
  DGR26: DOI.10.1007/978-3-032-22698-3_5
  I-D.ietf-lamps-pq-composite-sigs:
  RFC9955:
  ANSSI2024:
    title: "Position Paper on Quantum Key Distribution"
    target: https://cyber.gouv.fr/sites/default/files/document/Quantum_Key_Distribution_Position_Paper.pdf
    author:
      - org: "French Cybersecurity Agency (ANSSI)"
      - org: "Federal Office for Information Security (BSI)"
      - org: "Netherlands National Communications Security Agency (NLNCSA)"
      - org: "Swedish National Communications Security Authority, Swedish Armed Forces"
  DLOGsig:
    title: "IT Security techniques — Digital signatures with appendix — Part 3: Discrete logarithm based mechanisms"
    date: November 2018
    author:
      - org: ITU-T
    seriesinfo:
      ISO/IEC: 14888-3:2018
...

--- abstract


This document defines Silithium, an augmentation of US NIST Module-Lattice-based Digital Signing Algorithm (ML-DSA) [FIPS.204] with traditional elliptic-curve operations, that uses ML-DSA in a black-box manner.
This results in a digital signature scheme with hybrid security, requiring solving hard lattice problems as well as discrete logarithm in order to forge a signature.
This augmentation is designed to satisfy regulatory guidelines in certain regions.
Silithium is strongly unforgeable as long as ML-DSA is.
Morevoer, Silithium is safe for key-reuse both with its traditional and post-quantum component.


--- middle

# Introduction

Quantum computing poses new challenges to cryptography, as traditional signature algorithms, such as RSA or ECDSA, are vulnerable to quantum attacks.
Fortunately, new cryptographic standards, such as ML-DSA, recently became available and are assumed to be resistant to such attacks.

This leaves the community in a situation where some standards are widely deployed, remain secure for now, but will be broken some time in the future,
while on the other hand other standards are only at the beginning of their deployment.
For the latter, even if the mathematical problems underlying their security are starting to be well understood, many other security issues may arise due to the lack of time to fully harden their implementation, or even find all the bugs they may contain.

## Hybrid cryptography

Due to the recent nature of post-quantum cryptography and especially post-quantum standards, several European cybersecurity agencies [ANSSI2024] are recommending to hybridize these new standards with traditional cryptography, resulting in PQ/T hybrids.
These hybrids must be resistant to both classical and quantum attacks, ensuring security as high as the maximum security between post-quantum and traditional cryptography.

Hybridization for signature scheme must be tackled carefully. The role played by signatures in modern protocols and their induced complexity is reinforced by the intrinsic complexity of hybridization. One particular notion that is not covered by the standard unforgeability (EU-CMA) or strong unfogeability (sEU-CMA) security notions for signatures is the reuse of keys between the hybrid signature scheme and its components. In this setting, which should be avoided in general but could also be necessary in some applications, it must not be possible to break down the hybrid signature scheme into its traditional and post-quantum components, and vice-versa.
Previous works have covered this topic and defined guidelines and desirable notions for hybrid signature schemes {{RFC9955}}.

## Silithium

Silithium is a particular instantiation of the hybrid signature framework described in [DGR26].
This paper introduces a new security notion called (s)H-EU-CMA which captures the well known notions of (strong) unforgeability and apply them to PQ/T hybrid schemes. In particular, this grants strong non-separability to the resulting scheme, and this removes the security issues that may arise when private key material of one component is reused outside the PQ/T scheme. See {{sec-considerations}} for more details on the security properties.

As described in [DGR26], Silithium is a PQ/T hybrid construction building on ML-DSA and EC-Schnorr (also called EC-SDSA).
While we assume that the developer will have access to an ML-DSA implementation, we do not make such assumption for EC-Schnorr, as this scheme is less widely used. However, we expect that a general purpose elliptic curve library is likely to be available. For this reason, in {{scheme-description}} we describe directly the elliptic curve operations rather than relying on the high-level description of EC-Schnorr like in [DGR26].

Silithium is designed with the goal of lessening the implementation burden. Assuming elliptic curve addition and multiplication as well as ML-DSA are available, then it can quickly be implemented: signing requires in essence sampling a curve point, signing with ML-DSA and computing one multiplication and one addition modulo the order of the curve.

Silithium offers existential unforgeability (EU-CMA) under hybrid assumptions, i.e. either elliptic curve discrete logarithm or lattice problems are hard, as well as strong existential unforgeability (sEU-CMA) under the assumption that lattice problems are hard, making it fit for most applications.
It also offers various notions of non-separability (and other beyond unforgeability features), making it resilient to uses (and sometimes misuses) where hybrid keys are split and reused inside their component signature schemes, which could be desirable in some migration scenarios.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification is consistent with the terminology defined in {{RFC9794}}.
Some relevant definitions from {{RFC9794}} are copied here for easier reading.
In addition, the following terminology is used throughout this specification:

**ALGORITHM**:
  The usage of the term "algorithm" within this
  specification generally refers to any function which
  has a registered Object Identifier (OID) for
  use within an ASN.1 AlgorithmIdentifier.

**POST-QUANTUM TRADITIONAL (PQ/T) HYBRID SCHEME**:
  {{RFC9794}} defines a PQ/T Hybrid Scheme as:
  A multi-algorithm scheme where at least one component algorithm
  is a post-quantum algorithm and at least one is a traditional algorithm.

**SIGNATURE**:
          A digital cryptographic signature, making no assumptions
            about which algorithm.

The following new terminology is also introduced:

**KEY-REUSE SAFETY**:
    A PQ/T scheme is said to be safe for PQ key-reuse if its PQ (resp. traditional) key component can be separately reused in the corresponding PQ (resp. traditional) scheme.

## Notations

The algorithm descriptions use python-like syntax. The following symbols deserve special mention:

 * `||` represents concatenation of two byte-arrays.

 * `[:]` represents byte array slicing.

 * `(a, b)` represents a pair of values `a` and `b`. Typically, this indicates that a function returns multiple values; the exact conveyance mechanism -- tuple, struct, output parameters, etc. -- is left to the implementer.

 * `(a, _)`: represents a pair of values where one -- the second one in this case -- is ignored.

 * `func(a) -> b`: represents a function named `func` that takes `a` as input and produces `b`.

 * `Func<TYPE>()`: represents a function that is parameterized by `<TYPE>` meaning that the function's implementation will have minor differences depending on the underlying TYPE. Typically this means that a function will need to look up different constants or use different underlying cryptographic primitives depending on which composite algorithm it is implementing.

For the purpose of describing the elliptic curve operations, the following notation is used throughout the document:

~~~
q           Order of the group

G           Generator of the group

[n]P        P added to itself n times
~~~

# Scheme Description {#scheme-description}

This section describes the Silithium functions needed to instantiate the public API of a digital signature scheme.

## Key Generation

The security properties guaranteed by Silithium do not forbid key material generated for Silithium to be reused outside of Silithium (and conversely). See {{sec-considerations}} for further discussion on security properties. In this document, we describe the generation of fresh key material.

To generate a new key pair for Silithium, the `KeyGen() -> (pk, sk)` function is used.  The `KeyGen()` function calls independently the key generation algorithm of ML-DSA, and the `RandomPoint` function that outputs `(d, P)` such that `P = [d]G`. Multi-threaded, multi-process, or multi-module applications might choose to execute the key generation functions in parallel for better key generation performance or architectural modularity.

The following describes how to instantiate a `KeyGen()` function for a given variant of Silithium represented by `<OID>`.

~~~
Silithium-<OID>.KeyGen() -> (pk, sk)

Explicit inputs:

  None

Implicit inputs mapped from <OID>:

  ML-DSA  The underlying ML-DSA algorithm and parameter set,
          for instance "ML-DSA-65".

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  (pk, sk)    A Silithium keypair.

Key generation process:

  1. Generate component keys

    mldsaSeed = Random(32)
    (mldsaPK, mldsaSK) = ML-DSA.KeyGen_internal(mldsaSeed)
    (d, P) = Curve.RandomPoint()

  2. Check for component key generation failure

    if NOT (mldsaPK, mldsaSK) or NOT (d, P):
      output "Key generation error"

  3. Output the Silithium public and private keys

    pk = SerializePublicKey(mldsaPK, P)
    sk = SerializePrivateKey(mldsaSeed, d)
~~~

This keygen process makes use of the seed-based `ML-DSA.KeyGen_internal(𝜉)`, which is defined in Algorithm 6 of [FIPS.204].
If private key interoperability is not required, it is possible to deviate from {{serialize-privkey}} and store the ML-DSA private key as an expanded private key, allowing the use of `ML-DSA.KeyGen()` (Algorithm 1 of [FIPS.204]).

## Sign

The `Sign()` algorithm of Silithium opens up the signature procedure of EC-Schnorr to integrate a call to `ML-DSA.Sign()`. The rough idea is that the challenge of EC-Schnorr is no longer computed with a hash function, but is rather extracted from an ML-DSA signature. To further bind the two components together, the random nonce `R` (a point on the Curve) is passed to ML-DSA as a context string, along with the EC-Schnorr public key.

~~~
Silithium-<OID>.Sign(sk, M) -> s

Explicit inputs:

  sk      Silithium private key consisting of signing private keys
          for each component.

  M       The message to be signed, an octet string.

Implicit inputs mapped from <OID>:

  ML-DSA  The underlying ML-DSA algorithm and parameter set,
          for instance "ML-DSA-65".

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  s    The Silithium signature value.

Signature Generation Process:

  1. Deserialize the private key and recompute the public point

    (mldsaSeed, d) = DeserializePrivateKey(sk)
    (_, mldsaSK) = ML-DSA.KeyGen_internal(mldsaSeed)
    P = [d]G

  2. Generate a random scalar k in the range ]0, q[

    k = RandScalar(0, q)

  3. Compute the random nonce

    R = [k]G

  4. Serialize R and the EC-Schnorr public key as an octet string

    ctx = SerializeCommitment(R, P)

  5. Sign the message with ML-DSA and extract the challenge component

    mldsaSig = ML-DSA.Sign(mldsaSK, M, ctx)
    c = mldsaSig[:c_len]

  6. Complete the EC-Schnorr signature procedure

    x = k + ecSK * c mod q

  7. Serialize and output the Silithium signature

    s = SerializeSignatureValue(mldsaSig, x)
~~~

Note that in step 4, both `R` and the EC-Schnorr public key are at most 66 bytes (in compressed form), hence their serialization fits under the maximum size of 255 bytes for ML-DSA context string. Silithium does not take any context string as an input and does not allow user-defined context string to be transmitted to ML-DSA. See {{user-context}} for a discussion on user-defined context string.

## Verify

The `Verify()` algorithm recomputes the context string from the EC-Schnorr components `(x, c)` and the EC-Schnorr public key. Validating the ML-DSA signature with such context string implicitly validates the EC-Schnorr signature. It is not possible to abort early from the signature verification, ensuring the Simultaneous Verification property (further discussion in {{sec-considerations}}).

~~~
Silithium-<OID>.Verify(pk, M, s) -> true or false

Explicit inputs:

  pk      Silithium public key consisting of verification public keys
          for each component.

  M       Message whose signature is to be verified, an octet string.

  s       A Silithium signature value to be verified.

Implicit inputs mapped from <OID>:

  ML-DSA  The underlying ML-DSA algorithm and parameter set,
          for instance "ML-DSA-65".

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  Validity (bool)   True if the Silithium signature is valid, false
                    otherwise.

Signature Verification Process:

  1. Deserialize the public key

    (mldsaPK, P) = DeserializePublicKey(pk)

  2. Deserialize the signature and extract components. Both c and x
    are interpreted as scalars.

    (c, x, mldsaSig) = DeserializeSignatureValue(s)

  3. Ensure that x is within the bounds

    if (x < 0 or x >= q)
      return error

  4. Recompute the nonce R

    R = [x]G - [c]P

  5. Serialize R and the EC-Schnorr public key as an octet string

    ctx = SerializeCommitment(R, P)

  6. Output the result of ML-DSA verification

    return ML-DSA.Verify(mldsaPK, M, mldsaSig, ctx)
~~~

# Serialization

## Curve Point encoding {#point_encode}

Elliptic curve points MUST be encoded in compressed form, so that the commitment can fit inside the ML-DSA context string (of maximum length 255 bytes).
To unify the implementation, the same point encoding is used at every step of the algorithm, even if no size constraints apply.

Point compression consists in storing only the x-coordinate of a point, with an additional byte at the first position of the encoding set to `0x02` (resp. `0x03`) if its y-coordinate is positive (resp. negative).
We refer to {{SEC1}} (sections 2.3.3 and 2.3.4) for a formal description of point compression.

Scalar (also called field element) are encoded to bytes following sections 2.3.5 to 2.3.8 of {{SEC1}}.

## SerializePublicKey and DeserializePublicKey

The serialization routine for public keys performs encoding of the EC-Schnorr public point, and then concatenates the ML-DSA public key with the encoded point.

~~~
Silithium<OID>.SerializePublicKey(mldsaPK, P) -> bytes

Explicit inputs:

  mldsaPK The ML-DSA public key, which is bytes.

  P       The EC-Schnorr public key, a point on the Curve.

Implicit inputs mapped from <OID>:

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  bytes   The encoded Silithium public key.

Serialization Process:

  1. Encode P using compressed encoding.

    encP = Curve.Compress(P)

  2. Combine and output the encoded public key

    output mldsaPK || encP
~~~

Deserialization reverses this process.
This function depends on the underlying variant of ML-DSA, as the length of `mldsaPK` varies across parameter sets.
Values for `ML-DSA.PKLen` are listed in {{tab-mldsa-sizes}}.

~~~
Silithium<OID>.DeserializePublicKey(bytes) -> (mldsaPK, P)

Explicit inputs:

  bytes    An encoded Silithium public key.

Implicit inputs mapped from <OID>:

  ML-DSA  The underlying ML-DSA algorithm and
          parameter set to use, for example "ML-DSA-65".

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  mldsaPK The ML-DSA public key, which is bytes.

  P  The EC-Schnorr public key, a point on the Curve.

Deserialization Process:

  1. Split the Silithium public key into both components
    mldsaPK = bytes[:ML-DSA.PKLen]
    encP = bytes[ML-DSA.PKLen:]

  2. Decode P using compressed encoding.
    P = Curve.Decompress(encP)

  3. Output the decoded public key

    output (mldsaPK, P)
~~~

## SerializePrivateKey and DeserializePrivateKey {#serialize-privkey}

The serialization routine for private keys performs encoding of the private scalar, and then concatenates the ML-DSA private seed with the encoded scalar.

~~~
Silithium.SerializePrivateKey(mldsaSeed, d) -> bytes

Explicit inputs:

  mldsaSeed The ML-DSA private key, which is the bytes of the seed.

  d         The secret scalar.

Implicit inputs:

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  bytes   The encoded Silithium private key.

Serialization Process:

  1. Encode d with fixed-length encoding defined by the Curve.

    encd = ScalarEncode(d)

  2. Combine and output the encoded private key

    output mldsaSeed || encd
~~~

Deserialization reverses this process. This function does not depend on the ML-DSA variant, because the length of `mldsaSeed` is fixed across all parameter sets.

~~~
Silithium.DeserializePrivateKey(bytes) -> (mldsaSeed, d)

Explicit inputs:

  bytes      An encoded Silithium private key.

Implicit inputs:

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  mldsaSeed The ML-DSA private key, which is the bytes of the seed.

  d         The private scalar.

Deserialization Process:

  1. Split the Silithium private key into both components

    mldsaSeed = bytes[:32]
    encd = bytes[32:]

  2. Decode d as a scalar

    d = ScalarDecode(encd)

  3. Output the decoded private key

    output (mldsaSeed, d)
~~~

## SerializeSignatureValue and DeserializeSignatureValue

The serialization routine encodes the `x` component of the EC-Schnorr signature (which is a scalar), and concatenates its encoding with the ML-DSA signature.

~~~
Silithium.SerializeSignatureValue(mldsaSig, x) -> bytes

Explicit inputs:

  mldsaSig The ML-DSA signature value, which is bytes.

  x        The scalar component of the EC-Schnorr signature.

Implicit inputs:

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  bytes   The encoded Silithium signature value.

Serialization Process:

  1. Encode x with fixed-length encoding defined by the Curve.

    encx = ScalarEncode(x)

  2. Combine and output the encoded private key

    output mldsaSig || encx
~~~

Deserialization reverses this process. Furthermore, it extracts the component `c` of the ML-DSA signature to allow its independent reuse.
This function depends on the underlying variant of ML-DSA, as the length of `mldsaSig` varies across parameter sets.
Values for `ML-DSA.SigLen` and `ML-DSA.cLen` are listed in {{tab-mldsa-sizes}}.

~~~
Silithium<OID>.DeserializeSignatureValue(bytes) -> (mldsaSig, x)

Explicit inputs:

  bytes    An encoded Silithium signature value.

Implicit inputs mapped from <OID>:

  ML-DSA  The underlying ML-DSA algorithm and
          parameter set to use, for example "ML-DSA-65".

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  c         The challenge component of the EC-Schnorr signature.

  x         The scalar component of the EC-Schnorr signature.

  mldsaSig  The ML-DSA signature value, which is bytes.

Deserialization Process:

  1. Split the Silithium signature into all components.
    mldsaSig = bytes[:ML-DSA.SigLen]
    encx = bytes[ML-DSA.SigLen:]
    encc = mldsaSig[:ML-DSA.cLen]

  2. Decode x and c as a scalar.
    x = Curve.ScalarDecode(encx)
    c = Curve.ScalarDecode(encc)

  3. Output the decoded signature.

    output (c, x, mldsaSig)
~~~

## SerializeCommitment

The EC-Schnorr nonce `R` (a point on the Curve) is serialized together with the EC-Schnorr public key `P` (a point on the Curve) as a context string, later passed on to the ML-DSA signature function. As discussed in {{point_encode}}, compressed encoding is used, such that the resulting bytes string fits under the maximum length of 255 bytes for all parameter sets.

~~~
Silithium.SerializeCommitment(R, P) -> bytes

Explicit inputs:

  R   The EC-Schnorr nonce, a point on the Curve.

  P   The EC-Schnorr public key, a point on the Curve.

Implicit inputs:

  Curve   The underlying elliptic curve, for instance "P256" or "P-384".

Output:

  bytes   The encoded commitment.

Serialization Process:

  1. Encode R and P using compressed encoding.

    encR = Curve.Compress(R)
    encP = Curve.Compress(P)

  2. Combine and output the encoded commitment.

    output encR || encP
~~~

There is no need to reverse this process.

# Algorithm Identifiers and Parameters

This section lists the algorithm identifiers and parameters for all Silithium variants.

Only one parameter set per security level is defined to keep the overall number of combinations within a reasonable limit.

* id-Silithium44-P-256
  - OID: **TODO**
  - ML-DSA variant: ML-DSA-44
  - Curve: P-256

* id-Silithium65-P-384
  - OID: **TODO**
  - ML-DSA variant: ML-DSA-65
  - Curve: P-384

* id-Silithium87-P-521
  - OID: **TODO**
  - ML-DSA variant: ML-DSA-87
  - Curve: P-521

# Security Considerations {#sec-considerations}

This section brushes over the security properties of Silithium.
More detailed proofs and reductions can be found in {{DGR26}}.
This section also explains the differences in security introduced by the variant considered in this specification.

## Changes with respect to the published version of Silithium {#dgr-changes}

The original scheme from {{DGR26}}, which we dub Silithium-DGR to avoid ambiguity, is similar to the one presented here, except that it replaces `tr = H(vkPQ)` with `tr' = H(vkT, vkPQ)`, which is then used to derive `mu = H(tr('), R, M)`. Contrary to this variant, no specific context string is necessary, but could be introduced for the sake of domain separation.
To avoid reimplementing the whole scheme, Silithium-DGR relies on either an implementation of ML-DSA providing the external-mu variant or by modifying its secret key.
Silithium-DGR posess key-reuse safety for both ECDSA as the traditional component and ML-DSA as the post-quantum component.

As neither external mu nor updating the signing key may be available, Silithium puts the updated data inside the context string of ML-DSA instead of its signing key.
The resulting scheme can be implemented using any ML-DSA implementation and has the following security properties, detailed in the following sections.

 - Unforgeability: breaking Silithium is equivalent to breaking ML-DSA *and* the discrete logarithm problem.

 - Strong unforgeability: finding a new signature for an already signed message is equivalent to doing the same for ML-DSA.

 - Traditional key-reuse safety: Silithium key can be securely reused with ECDSA, in the sense that unforgeability for the two schemes is still guaranteed.

 - PQ key-reuse safety: Silithium key can be securely reused with ML-DSA, with the restriction that the application MUST NOT allow the user to set arbitrary context strings.

The following sections give rationale about the aforementioned security claims.

## Unforgeability

Silithium satisfies the standard unforgeability security notion as long as either ML-DSA does so *or* discrete logarithm is a hard problem.
Indeed, to forge a signature, an adversary must simultaneously forge a ML-DSA signature as well as respond correctly to a Schnorr challenge.
As long as one of the two is a difficult problem, then forging a Silithium signature is also hard.

This also means that the nature of the unforgeability of Silithium changes if one of the two is broken:

 - If discrete logarithm is broken, e.g. with a quantum computer, then Silithium is as secure as ML-DSA.

 - If ML-DSA is broken, then Silithium remains secure against classical adversaries, as long as the discrete logarithm problem is.

## Strong Unforgeability

Theorem 2 in {{DGR26}} shows that signing a message for which a signature is already known for Silithium-DGR is as hard to do as for ML-DSA.

This result remains true for Silithium. In order to forge another signature for an already signed message, an adversary could:

 - Use another traditional nonce `R'`. In that case, the (message, context) signed by ML-DSA changes, and the adversary must break ML-DSA to complete its forgery.

 - Find another `x' != x` such that `[x]G - [c]P = [x']G - [c]P`. This is actually impossible to do, as Silithium checks that `x` lies in a domain where no such `x'` exists.

 - Find another ML-DSA signature for the same (message, context) pair. Assuming that this new signature contains the same challenge `c` as the genuine signature,
this allows to recycle the traditional component that is part of the Silithium signature without changes,
while still obtaining a fresh signature, as the other part of the ML-DSA signature is fresh.

Out of the three attack paths, the second one is impossible, and the first and third ones require breaking the (strong) unforgeability of ML-DSA.
As such, with respect to strong unforgeability, Silithium is at least as hard to break as ML-DSA.


## Key-reuse safety with ECDSA

As a starting point, it was proven in {{DGR26}} that Silithium-DGR can posesses both traditional and post-quantum key reuse safety with ECDSA and ML-DSA respectively.

Does the knowledge of multiple (message, signature) pairs for both Silithium and ECDSA help forge signatures for either of them, when ECDSA uses a keypair that is a subset of a Silithium keypair?

 - Each of these two signature schemes is unforgeable when used alone, the remaining question being whether concurrent usage creates undesirable interactions.

 - The two signature schemes are relying on different hash functions and different paradigms.

The only attack path to leverage this additional information is a key-recovery attack on one of the schemes in order to use the recovered key to attack the second one.
However, key-recovery on either of the two schemes is a difficult problem, harder than directly forging, making this attack path impractical.

As such, Silithium is safe for key-reuse with ECDSA.

## Key-reuse safety with ML-DSA {#pq-key-reuse}

The changes introduced in Silithium with respect to Silithium-DGR have an impact here.
Indeed, Silithium integrates an ML-DSA signature as part of its signature.
An adversary can extract this signature, giving a valid ML-DSA signature that was not produced by the ML-DSA signer.

Conversely, the ML-DSA signer can be used to produce Silithium signatures. The adversary still has to break the discrete logarithm problem to complete the signing algorithm of Silithium, which is possible for a quantum adversary. Hence, under these hypotheses:

 - the standalone ML-DSA instance is not secure, as an adversary is able to use Silithium signatures to produce valid ML-DSA signatures,

 - Silithium has degraded security against quantum adversaries.

Noting that Silithium uses a specific context string when calling ML-DSA, the applicative layer can:

 - disallow ML-DSA verification on user-provided context string,

 - disallow ML-DSA signing on user-provided context string.

Each of the above points answers its corresponding security concern.
In the case of key-reuse for standalone ML-DSA where the context string cannot be arbitrarily set by an attacker, these attacks do no longer work.
Under these two hypotheses on the applicative layer, a security level similar to Silithium-DGR is attained: Silithium keys can securely be reused with ML-DSA.

# Potential Changes

## Append R to the message

In this proposal, the commitment `R` is passed to ML-DSA as part of its context.
It could be instead passed to ML-DSA as part of the message to sign, by appending it at the end of `M` using the external-mu feature of [FIPS.204].

In that case, the PQ key-reuse safety is improved. On one hand, the message is now appended with a suffix that is impredictible for the adversary, which could make the signed message useless for the adversary. On the other hand, the context string is fixed once the public key is known, which makes it easier to identify potential ML-DSA signatures extracted from a Silithium signature.

This option is currently left out due to implementations considerations. As ML-DSA is not a streaming signature algorithm, appending the commitment `R` to the message `M` before signing would require a new memory allocation whose size is not predictable, and this would not be suitable for devices with memory constraints.

## User-defined context string {#user-context}

As outlined in precedent sections, Silithium uses at most 2 * 66 = 132 bytes of ML-DSA context string to incorporates the EC-Schnorr public key and nonce. This leaves 123 bytes of available space, as the maximum length of ML-DSA context string is 255 bytes. It could hence be possible to define a Silithium variant that takes as input a user-defined context string, with a maximum length of 123 bytes.
If a longer context string is desirable, note that the previous modification frees 66 bytes of context.

## Silithium-DGR

As discussed in {{dgr-changes}} this proposal made the choice to introduce a variant of the Silithium-DGR scheme in order to be implementable in any context and not require the external-mu variant of ML-DSA.
This comes at the cost of additional assumptions made on the applicative layer in order to ensure PQ key-reuse safety, as discussed in {{pq-key-reuse}}.
If the other trade-off is more desirable, this proposal could be changed back to the original Silithium-DGR scheme.

# IANA Considerations

This document has no IANA actions.

# Implementation considerations

Silithium uses ML-DSA (a FIPS approved algorithm) as a black-box, enabling the reuse of existing FIPS certified implementations.

The `Curve.RandomPoint()`, which outputs `(d, P)` such that `P = [d]G`, is not specified in this document. Its output is exactly the same as what would be produced by a key generation algorithm for most schemes based on elliptic curves, as for example ECDSA. Even though Silithium is not built on ECDSA, it is entirely possible to reuse the key generation algorithm of ECDSA as `Curve.RandomPoint()`, if such function is available.

--- back

# Silithium and Components Sizes

| Algorithm | SKLen | PKLen | SigLen |
| --------- | ----- | ----- | ------ |
| id-Silithium44-P-256 | 64 | 1345 | 2452 |
| id-Silithium65-P-384 | 80 | 2001 | 3357 |
| id-Silithium87-P-521 | 98 | 2659 | 4693 |
{: #tab-silithium-sizes title="Sizes (in bytes) of keys and signatures of Silithium"}

| Algorithm | PKLen | SigLen | challLen |
| --------- | ----- | ------ | ---- |
| id-ML-DSA-44 | 1312 | 2420 | 32 |
| id-ML-DSA-65 | 1952 | 3309 | 48 |
| id-ML-DSA-87 | 2592 | 4627 | 64 |
{: #tab-mldsa-sizes title="Sizes (in bytes) of ML-DSA signature components"}

# Component Algorithm Reference {#appdx_components}

| Component Signature Algorithm ID | OID | Specification |
| ----------- | ----------- | ----------- |
| id-ML-DSA-44 | 2.16.840.1.101.3.4.3.17 | [FIPS.204] |
| id-ML-DSA-65 | 2.16.840.1.101.3.4.3.18 | [FIPS.204] |
| id-ML-DSA-87 | 2.16.840.1.101.3.4.3.19 | [FIPS.204] |
{: #tab-component-sig-algs title="ML-DSA variants used in Silithium"}

| Elliptic CurveID | OID | Specification |
| ----------- | ----------- | ----------- |
| secp256r1 | 1.2.840.10045.3.1.7 | [RFC6090], [SEC2] |
| secp384r1 | 1.3.132.0.34 | [RFC5480], [RFC6090], [SEC2] |
| secp521r1 | 1.3.132.0.35 | [RFC5480], [RFC6090], [SEC2] |
{: #tab-component-curve-algs title="Elliptic Curves used in Silithium"}

EC-Schnorr is not used as a black-box algorithm.
As an informative reference, the EC-SDSA scheme is defined in [DLOGsig].

# Acknowledgments
{:numbered="false"}

We would like to thank Scott Fluhrer (Cisco), John Gray (Entrust) and Thom Wiggers (PQShield) for the useful discussions about Silithium.

The structure and content of this document are heavily inspired by {{I-D.ietf-lamps-pq-composite-sigs}}.
