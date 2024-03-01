+++

title = "Sigstore February Roundup"
date = "2024-03-01"
tags = ["sigstore","securesupplychain"]
draft = false
author = "Luke Hinds (TSC Chair)"
type = "post"

+++

Welcome to the February edition of the Sigstore Roundup! This is a regular summary of Sigstore news, events, releases and other happenings.

### Events

#### KubeCon Europe 2024

The next KubeCon Europe will be held on **19th – 22nd March**.

There are serveral Sigstore related talks and events planned for KubeCon Europe, including:

- [Securing the Supply Chain with Sigstore Artifacts Signatures at Scale - Dmitry Savintsev & Yonghe Zhao, Yahoo](https://kccnceu2024.sched.com/event/1YeNE)
- [Navigating the Software Supply Chain Defense Landscape - Marina Moore & Aditya Sirish A Yelgundhalli, New York University](https://kccnceu2024.sched.com/event/1YeO3/navigating-the-software-supply-chain-defense-landscape-marina-moore-aditya-sirish-a-yelgundhalli-new-york-university)
- [Contribfest: Enable Additional Signing Mechanisms for TUF and in-toto: No Cryptography Skills Required](https://kccnceu2024.sched.com/event/1YkRE/contribfest-enable-additional-signing-mechanisms-for-tuf-and-in-toto-no-cryptography-skills-required)

#### FOSDEM 2024

The next FOSDEM will be held in Brussels, Belgium on the **3rd & 4th February 2024**
along with a talk on Sigstore and SLSA by John Viega.

- [Making it easy to get to SLSA level 2, John Viega](https://fosdem.org/2024/schedule/event/fosdem-2024-2877-making-it-easy-to-get-to-slsa-level-2/)

#### Sigstore Community Meeting

The last Sigstore Community Meeting was held on the **February 20th**.

You can watch a recording of the meeting [here](https://www.youtube.com/watch?v=YJeKKPNpXf0)

The next Sigstore Community Meeting will be held on **February 5th**.

To join the meeting, please see the [meeting details](https://calendar.google.com/calendar/u/0/embed?src=fq4kgom2ce43hncnbcfja2ck20@group.calendar.google.com&ctz=America/New_York)

Please do come along, all are welcome!

### Interesting Discussions and Developments

#### Sigstore Graduation Review

Sigstore will go forward for Graduation Review in the OpenSSF TAC meeting on
March 5th. This is a significant milestone for the project and we are excited to
see the outcome. The pull request for the graduation review can be found
[here](https://github.com/ossf/tac/pull/273)

### Latest Releases

This month has not seen any major releases, but there have been a number of
minor releases across the various Sigstore projects.

### Fulcio v1.4.4

The Fulcio library has been updated to v1.4.4. This release includes the addition
of a production OIDC provider for Eclipse, and some minor bug fixes and changes
such as changing the `parseExtension` function to be public, exposing the
metrics port to be overridden, and the addition of a configurable idle timeout.

Read the release notes [here](https://github.com/sigstore/fulcio/releases/tag/v1.4.4)

### Rekor v1.3.5

Logs timestamps now have nanosecond precision, support was added for sha384/sha512
hash algorithms in hashedrekords, additional DB unique index correction

Read the release notes [here](https://github.com/sigstore/rekor/releases/tag/v1.3.5)

#### sigstore-go v0.1.0

v0.2.0 of sigstore-go includes an updated TUF client. This also updates
verification to require specifying both the certificate issuer and SAN.

Read the release notes [here](https://github.com/sigstore/sigstore-go/releases/tag/v0.2.0)

#### sigstore v1.8.1

The sigstore library has been updated to v1.8.2

Support was added for an Ed25519ph Signer/Verifier and autoclosing the oauth
flow window. 

Client credentials are now supported as an OIDC Auth Flow Provider.

Read the release notes [here](https://github.com/sigstore/sigstore/releases/tag/v1.8.2)

#### Timestamp v1.2.2

The timestamp-authority library has been updated to v1.2.2. Just a minor release
for a bug fix around a Go checksum database error on installation due to
deleting a tag

Read the release notes [here](https://github.com/sigstore/timestamp-authority/releases/tag/v1.2.2)

### In the News / Community

- Caleb Woodbine wrote a blog titled "Sign, Verify and Trust with Cosign" [read more](https://blog.calebwoodbine.com/sign-verify-and-trust-with-cosign/)

- The Opensource Minder project on how they are using sigstore to verify 
cryptographic provenance. [read more](https://stacklok.com/blog/4-ways-to-secure-your-software-artifacts-in-minder)

- A stream was hosted by Viktor Farcic and Whitney Lee on "Signing Artifacts - Feat. Notary, Sigstore, and Open Policy Containers" [watch here](https://www.youtube.com/watch?v=p4M-ZdBsA7o)

### Join the Community!

New contributors and users are always welcome into our community. We take pride in being friendly to new folks and fostering a welcoming and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks.

Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others.

Join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and come say hello! 👋