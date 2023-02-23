+++

title = "Sigstore Update — September 2022"
date = "2022-09-08"
tags = ["sigstore","javascript","softwaresecurity","softwaresupplychain","kubecon"]
draft = false
author = "Tracy Miranda"
type = "post"

+++

### SigstoreCon

The SigstoreCon call-for-papers closed last month and the program committee has been busy ranking the 23 great submissions received. Many thanks to all who submitted talks. And thanks to our program committee members: Priya Wadhwa, Lily Sturman, Appu Goundan, Jacques Chester, and Batuhan Apaydin.

The program will be announced on September 13. We hope to see you at [SigstoreCon](https://events.linuxfoundation.org/sigstorecon-north-america/) our first official event, on October 25 in Detroit, in co-location with KubeCon + CloudNativeCon North America. [Register here.](https://events.linuxfoundation.org/sigstorecon-north-america/register/)

### Sigstore Awards

For the first time, we will be hosting Sigstore Awards! The awards are to recognize the wonderful work that you all put into this community to make Sigstore the standard for signing, verifying and protecting software.

We will be giving out three awards and the winners will be nominated by the community and winner selected by the Sigstore Technical Steering Committee.

Nominations are now open and will close on September 20.

Please nominate folks here 👇

- [Most Valuable Sigstore Contributor](https://github.com/sigstore/community/issues/123)
- [Best Sigstore Evangelist](https://github.com/sigstore/community/issues/124)
- [Best Sigstore User Adopter](https://github.com/sigstore/community/issues/125)

The Award Ceremony will take place at SigstoreCon in Detroit on October 25.

![](/images/sigstore.jpg)

### Sigstore Case Study

In case you missed it we had a recent Sigstore case study, check it out if you are interested in why organizations are looking to adopt Sigstore:

- [Signing and Securing Confidential Kubernetes Clusters in the Cloud with Sigstore](https://blog.sigstore.dev/signing-and-securing-confidential-kubernetes-clusters-in-the-cloud-with-sigstore-aceac3034e70) by Fabian Kammel of Edgeless Systems

![](/images/fabian.png)

### NPM + Sigstore: A first look at sigstore-js

Last month we shared Github’s plans to [use Sigstore for the npm package manager](https://github.blog/2022-08-08-new-request-for-comments-on-improving-npm-security-with-sigstore-is-now-open/) and the recent donation of sigstore-js to the community. This month, Brian DeHamer of Github joined us at Sigstore Office Hours to give a first look at the new library that can be used for signing and verifying signatures with Javascript/Typescript. Check out the demo here:

{{< youtube z9QWOQm1tsI >}}

### Big new release of sigstore-rs for Rust

This month saw a big release of the sigstore-rs library for rustlang. The [0.4.0 release](https://github.com/sigstore/sigstore-rs/releases/tag/v0.4.0) included some major new features including:

- Full rekor OpenAPI client code
- Crypto key interface

Many thanks to all contributors, including our new contributors:

- [Tony Arcieri](https://github.com/tarcieri)
- [Jyotsna](https://github.com/jyotsna-penumaka)
- [Xynnn_](https://github.com/Xynnn007)

### Other New Releases

#### Sigstore

Sigstore is currently on [version 1.4.0](https://github.com/sigstore/sigstore)!

Thank you and welcome to our new contributors:

- [Ricardo Katz](https://github.com/rikatz)
- [Ville Aikas](https://github.com/vaikas)

#### Cosign

Cosign is container signing, verification and storage in an OCI registry. Its latest release is [v1.11.1](https://github.com/sigstore/cosign/releases).

Thank you and welcome to its most recent contributors:

- [David Bendory](https://github.com/bendory)
- [Kazuma Watanabe](https://github.com/wata727)
- [Noah Kreiger](https://github.com/nkreiger)
- [Samsondeen](https://github.com/dsa0x)

#### Fulcio

Fulcio issues code-signing certificates bound to OpenID Connect identities for use within the Sigstore ecosystem. Its most recent release is [v0.5.3](https://github.com/sigstore/fulcio/releases/tag/v0.5.3) from August 23.

Thank you and welcome to our newest contributor:

- [Azeem Shaikh](https://github.com/azeemshaikh38)
- [Paul Thomson](https://github.com/pauldthomson)

#### Gitsign

Keyless Git signing with Sigstore! Its latest release is [v0.3.0](https://github.com/sigstore/gitsign/releases/tag/v0.3.0) which features .gitconfig support as well as experimental support for [Git based attestations](https://github.com/sigstore/gitsign/tree/main/cmd/gitsign-attest) — store attestations about your code directly in your repository! (note: This is not yet included in the main `gitsign` binary and is not available as a downloadable release artifact - please install from source).

Check out this recent office hours where Billy Lynch demos the new [gitsign attest functionality.](https://youtu.be/z9QWOQm1tsI?t=668)

#### Rekor

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. Its latest release [v0.11.0](https://github.com/sigstore/rekor/releases/tag/v0.11.0) was on August 19.

Thank you and welcome Rekor’s newest contributor: [Samsondeen](https://github.com/dsa0x).

### Get Involved & Good First Issues

As always, we truly welcome contributors and users to our community. We take pride in being friendly to new folks and fostering a welcome and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks. Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others. We recently highlighted some ‘good first issues’ for those looking for a good place to get started:

- [Cosign good first issues](https://github.com/sigstore/cosign/issues?q=is%3Aissue+is%3Aopen+label%3A"good+first+issue")
- [Infra good first issue](https://github.com/sigstore/community/issues/121)

Come and join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and say hello!