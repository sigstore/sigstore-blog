+++
title = "Sigstore & Post-Quantum Cryptography (2025)"
author = "Zach Steindler (GitHub)"
date = "2025-05-15"
tags = ["sigstore", "nist", "postquantumcryptography"]
type = "post"
draft = false
+++

In the coming years, systems will transition to post-quantum cryptographic algorithms (PQCA). There is some inherent tension in these transitions as we learn things through adoption, but making decisions too soon can saddle you with tech debt. The quick summary is that the Sigstore project wants to enable people to sign content with PQCA keys as soon as possible, and adopt PQCA in the Sigstore services (like Fulcio, Rekor, and a timestamp authority) when reliable and vetted PQCA is available in the Go ecosystem.

![klystron gallery at Standford linear accelerator](/images/slac.jpg)

This document will describe what's happening with PQCA outside of Sigstore, give an expected timeline for PQCA adoption in Sigstore, and talk about open questions raised by the transition.

There's a [discussion in the sigstore/community repository](https://github.com/sigstore/community/discussions/591) where you can respond to the content in this post.

### Definitions

Because this work is in progress, there are synonyms for the same concept coming from the academic community or a US government standard.

ML-DSA (Module-Lattice-Based Digital Signature Algorithm) is used to sign and verify digital signatures; NIST standardized it in FIPS 204. It has a similar function to HMAC with a traditional asymmetric cryptographic algorithm, like RSA or elliptic curves. While RSA has public keys and signatures that are 100s of bytes and elliptic curves' are 10s of bytes, ML-DSA's are in the kilobyte range, making them much larger.

SLH-DSA (Stateless Hash-Based Digital Signature Algorithm) is another way to sign and verify digital signatures; NIST standardized it in FIPS 205. It's an alternative to ML-DSA with smaller keys (10s of bytes) but a larger signature (10 kilobyte range).

### Context

Here's a timeline of things that have already happened with respect to the transition to PQCA:

| Date | Description |
| --- | --- |
| Oct 2020 | **Superseded and no longer recommended:** NIST publishes [SP 800-208](https://csrc.nist.gov/pubs/sp/800/208/final) recommending PQCA's LMS and XMSS for digital signatures. These use stateful hash-based signature schemes which makes safe implementation very difficult. |
| Sep 2022 | **Superseded and no longer recommended:** NSA releases CNSA 2.0 (v1.0) recommending LMS / XMSS adoption by the end of 2025. |
| Aug 2024 | NIST publishes [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final), specifying ML-DSA for digital signatures, and [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final), specifying SLH-DSA for stateless hash-based digital signatures. |
| Nov 2024 | NIST publishes an initial public draft of [IR 8547](https://csrc.nist.gov/pubs/ir/8547/ipd), deprecating existing algorithms by 2030 and disallowing them by 2035. |
| Dec 2024 | NSA releases CNSA 2.0 (v2.1) recommending ML-DSA for digital signatures (FIPS 204) and setting CNSSP 15 as the adoption timeline. The [FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF) is still up, but the [guidance](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF) currently returns a HTTP 404. [CNSSP 15](https://www.cnss.gov/CNSS/openDoc.cfm?a=%2F3d3sI7VEM5svJe45UgUCg%3D%3D&b=C8B7B57CA694A14AF5C7B79438F7A465B5D7B8C7117957E9ED2F691A0A89A1FE48C8193F6B53A8398540602962045C7C) states "Beginning 1 January 2027... CNSA 2.0 algorithms will be required in all new products and services that provide cryptographic protection for users or for updates... CNSA 2.0 algorithms are mandated for all protocol use by 31 December 2031". |

And here are things that are planned but have not yet happened, or are works in progress:

| Date | Description |
| --- | --- |
| Jan 2025 | Trail of Bits is [working on general cryptographic agility across Sigstore services](https://github.com/sigstore/sig-clients/issues/16). |
| TBD | Upcoming FIPS publication by NIST will specify details for FALCON, another [digital signature finalist from the third round of the NIST competition](https://csrc.nist.gov/Projects/post-quantum-cryptography/post-quantum-cryptography-standardization/round-3-submissions).  |
| Aug 2025 | Planned release of Go 1.25. This is the [earliest release that would support SLH-DSA / FIPS 205](https://github.com/golang/go/issues/64537#issuecomment-2445056004). |
| Sep 2025 | Per [IR 8528](https://csrc.nist.gov/pubs/ir/8528/final), there is another NIST PQC Standardization Conference planned to evaluate any remaining of the 14 digital signature schemes from the [Round 2 Additional Signatures](https://csrc.nist.gov/Projects/pqc-dig-sig/round-2-additional-signatures) finalists. |
| Feb 2026 | Planned release of Go 1.26. It's possible SLH-DSA won't be available before this release. This is the [earliest release that would support ML-DSA / FIPS 204](https://github.com/golang/go/issues/64537#issuecomment-2877714729). |

### Sigstore's Approach

With that context in mind, how worried should we be about not using PQCA? As a general internet user, not very worried. Some adversaries are collecting encrypted content today with the hope of being able to decrypt them in the future, but this attack [won't work for digital signatures that include PQCA](https://bughunters.google.com/blog/5108747984306176/google-s-threat-model-for-post-quantum-cryptography#software-signatures). Since it takes some time to move to new cryptographic algorithms, it's better to start this process sooner than later. Additionally, some Sigstore users may be working with government or regulated industries that require PQCA in the near future.

Then what should Sigstore's approach to adopting PQCA be?

First, we want to unblock experimentation for clients that want to use PQCA keys as soon as possible. This will help people get hands-on experience with these algorithms and uncover potential issues and understand the tradeoffs. But to do so we have to choose which algorithm to support and add it to the protocol buffer specification. There's a [recently landed pull request](https://github.com/sigstore/protobuf-specs/pull/616) that adds the ML-DSA algorithm, chosen over SLH-DSA for its smaller signature size. With this change, clients will be able to provide their own implementation to things like [sigstore-go's signing Keypair interface](https://github.com/sigstore/sigstore-go/blob/48df3a9d13bf9e18e84af432290b5742c26437ab/pkg/sign/keys.go#L33-L39) and experiment with using ML-DSA ephemeral keys in Sigstore bundles. We can also easily add support to [Sigstore's Key Management Services (KMS) library](https://github.com/sigstore/sigstore/tree/main/pkg/signature/kms) to help with signing, as cloud providers include these algorithms in their offering.

Of course, there are also the Sigstore services, Fulcio, Rekor, and a timestamp authority that make use of cryptography. Trail of Bits is continuing their work on cryptographic agility so that in the future these services can specify which algorithms they are using. Once that work is done, people operating private Sigstore instances can bring their own PQCA implementation and experiment with end-to-end PQCA Sigstore instance. The public good Sigstore instance will wait for reliable and vetted PQCA to be part of Go's `crypto` package.

### Open Questions

However it's not enough for reliable and vetted PQCA implementations to be available. Like the rest of the internet, Sigstore will need a plan for how to transition to PQCA, and how to transition off of traditional cryptographic algorithms. Here are some of the questions we will need to answer in the future:

- What changes will need to take place to verification material? Both in terms of how the verification material is signed (as part of TUF) as well as if the existing trusted root format can accommodate both traditional and PQCA verification material?

- Will there be separate service endpoints that use PQCA? Or will services return both traditional signed content alongside PQCA (and if so, how will we avoid signature stripping attacks)?

- Will both traditional and PQCA signatures be stored in a single bundle, or will there be two separate bundles?

- How will end-users specify via policy what content they will accept for verification?

We will learn from other internet services as they handle not just onboarding to PQCA but also offboarding from traditional cryptographic algorithms.
