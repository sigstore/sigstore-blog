+++

title = "Sigstore January Roundup"
date = "2024-01-30"
tags = ["sigstore","securesupplychain"]
draft = false
author = "Luke Hinds (TSC Chair)"
type = "post"

+++

Welcome to the January edition of the Sigstore Roundup! This is a regular summary of Sigstore news, events, releases and other happenings.

### Events

#### KubeCon Europe 2024

The next KubeCon Europe will be held on **19th – 22nd March**.

There are serveral Sigstore related talks and events planned for KubeCon Europe, including:

- [Securing the Supply Chain with Sigstore Artifacts Signatures at Scale - Dmitry Savintsev & Yonghe Zhao, Yahoo](https://kccnceu2024.sched.com/event/1YeNE)
- [Navigating the Software Supply Chain Defense Landscape - Marina Moore & Aditya Sirish A Yelgundhalli, New York University](https://kccnceu2024.sched.com/event/1YeO3/navigating-the-software-supply-chain-defense-landscape-marina-moore-aditya-sirish-a-yelgundhalli-new-york-university)
- [Contribfest: Enable Additional Signing Mechanisms for TUF and in-toto: No Cryptography Skills Required](https://kccnceu2024.sched.com/event/1YkRE/contribfest-enable-additional-signing-mechanisms-for-tuf-and-in-toto-no-cryptography-skills-required)

#### FOSDEM 2024

The next FOSDEM will be held in **3rd & 4th February 2024** along with a talk
on Sigstore and SLSA by John Viega.

- [Making it easy to get to SLSA level 2, John Viega](https://fosdem.org/2024/schedule/event/fosdem-2024-2877-making-it-easy-to-get-to-slsa-level-2/)

#### Sigstore Community Meeting

The last Sigstore Community Meeting was be held on **January 23rd**.

You can watch a recording of the meeting [here](https://www.youtube.com/watch?v=r4WQYk4KWqs)

The next Sigstore Community Meeting will be held on **February 6th**.

To join the meeting, please see the [meeting details](https://calendar.google.com/calendar/u/0/embed?src=fq4kgom2ce43hncnbcfja2ck20@group.calendar.google.com&ctz=America/New_York)

### Interesting Discussions and Developments

#### Cryptographic Agility

Some very interesting work is happening in the Sigstore community around our
signature algorithm agility work and the integration of post-quantum signatures
schemes.

Community member William Woodruff had this to say about the work:

> Over the past month, we've revisited the Configurable Crypto Algorithms proposal and have begun to work towards signature algorithm agility on various core Sigstore codebases (Fulcio, Rekor, cosign, sigstore-go, and the protobuf-specs). We've already made significant progress towards enabling agility within a safe set of signature suites, and are quickly moving towards a state where the public Sigstore instances will be able to handle Ed25519 keypairs and signatures. Longer term, we'll be using that agility to investigate integrations of post-quantum signatures schemes, including hash-based signature schemes like LMS and LM-OTS (the latter being a perfect fit for Sigstore's "keyless" model).

The original proposal can be found [here](https://docs.google.com/document/d/18vTKFvTQdRt3OGz6Qd1xf04o-hugRYSup-1EAOWn7MQ/edit#heading=h.op2lvfrgiugr)

#### sigstore-go project

A new implementation of the signing and verification logic in being developed
within the sigstore-go library and we welcome all contributions or code review.

Check out the issue tracker (https://github.com/sigstore/sigstore-go/issues) for
starter issues

#### sigstore-python: DSSE support has landed!

Support for DSSE has been added to the sigstore-python library. This is a
significant milestone for the project. The DSSE support is currently in a
pre-release state and we welcome all feedback and contributions.

For those unfamiliar with DSSE (Dead Simple Signing Envelope), it is a protocol
for signing and verifying software artifacts using an embedded payload that is
passed over with a signature. This is in contrast to the traditional detached
signature approach.

You can view the pull request [here](https://github.com/sigstore/sigstore-python/commit/bb9b1a0fc689f074a10e3f314dbf8a7486705f78)

A release will be made shortly.

### Latest Releases

This month has not seen any major releases, but there have been a number of
minor releases across the various Sigstore projects.

#### sigstore v1.8.1

- Added kms support for AWS China region by @maaiika

#### Timestamp v1.2.1

- v1.2.1 includes a minor bug fix to set the SignedData version value
in a timestamp response as per the RFC.

### User adoption

- The Bandit Project is now using sigstore to sign their releases. [read more](https://github.com/PyCQA/bandit?tab=readme-ov-file#container-images) 

### In the News

- [Yahoo using Sigstore-based signing with their own PKI infrastructure](https://www.yahooinc.com/paranoids/scaling-up-supply-chain-security-implementing-sigstore-for-seamless-container-image-signing)
 - [Eclipse added as a trusted identity provider, enabling Sigstore with Eclipse's Jenkins instances](https://blogs.eclipse.org/post/mika%C3%ABl-barbero/eclipse-foundation-embraces-sigstore)

### Join the Community!

New contributors and users are always welcome into our community. We take pride in being friendly to new folks and fostering a welcoming and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks.

Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others.

Join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and come say hello! 👋