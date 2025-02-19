+++

title = "Sigstore October Roundup"
date = "2022-10-12"
tags = ["sigstore","hacktoberfest","sigstorecon","releases","securesupplychain"]
draft = false
author = "Sigstore"
type = "post"

+++

### Technical Steering Committee: New Member

Thank you, Dan Lorenc, for your time on the Technical Steering Committee (TSC)! Sigstore is where it is today thanks to your help.

We also want to give a big welcome to Priya Wadhwa who is replacing Dan on the TSC! We know you’ll also be great at moving Sigstore in the right direction.

### SigstoreCon

The SigstoreCon [program has been announced](https://events.linuxfoundation.org/sigstorecon-north-america/program/schedule/)!

We are thrilled to have representatives from 14 different companies speaking at the event, namely: Autodesk, Chainguard, Cycode, Datadog, Edgeless Systems, GitHub, Google, IBM Research, InfluxData, Nirmata, Red Hat, Trail of Bits, Upgrade, and VMware.

![](/images/sigstorecon.jpg)

Thank you to our Program Committee for putting together such a great program.

- Appu Goundan
- Batuhan Apaydin
- Jacques Chester
- Lily Sturman
- Priya Wadhwa

We look forward to seeing you all in Detroit! [Register here](https://events.linuxfoundation.org/sigstorecon-north-america/register/).

### SigstoreCon Twitter Space

Join the upcoming [Twitter Space](https://twitter.com/i/spaces/1lDxLnYZVVQGm) on Thursday, October 13 at 9 am PT to learn how to join, how to talk, and how to find new friends at SigstoreCon.

![](/images/twitterspace.jpg)

### Hacktoberfest

Sigstore is participating in Hacktoberfest (a month-long celebration of open source) for the first time. [Get all the details and start contributing](https://blog.sigstore.dev/contribute-to-sigstore-during-hacktoberfest-2022-42315a41da5f)!

![](/images/hacktober.png)

### New Look for Sigstore

You might have noticed that Sigstore has a new colour palette and a new logo. But there’s more! [Read the blog post](https://blog.sigstore.dev/a-new-look-for-sigstore-9dd38877e308) and where you’ll find the brand kit and logo downloads.

![](/images/newlook.png)

### Latest Releases

#### Sigstore

Sigstore is currently on [version 1.4.4](https://github.com/sigstore/sigstore/releases/tag/v1.4.4)!

What’s changed:

- Fixed TUF root initialization with GCS bucket. This affects anyone who uses their own TUF root hosted on GCS, and specifies the GCS bucket only by name and not by HTTP path
- Fix remoteFromMirror with GCS bucket

#### Cosign

Cosign is container signing, verification and storage in an OCI registry. Its latest release is [v1.13.0](https://github.com/sigstore/cosign/releases/tag/v1.13.0).

Highlight change: For users who have deployed a private instance of Fulcio release v0.6.x and issue certificates with the Username identity, you will need to upgrade to use this version.

#### Fulcio

Fulcio issues code-signing certificates bound to OpenID Connect identities for use within the Sigstore ecosystem. Its most recent release is [v1.0.0-rc.0](https://github.com/sigstore/fulcio/releases/tag/v1.0.0-rc.0).

What’s changed:

- Update previous releases and add notes for v0.6.0
- Use same way to output version and expose build info to prometheus
- Update swagger doc version for Fulcio 1.0
- Update CHANGELOG for v1.0.0-rc.0

#### Gitsign

Keyless Git signing with Sigstore! Its latest release is [v0.3.2](https://github.com/sigstore/gitsign/releases/tag/v0.3.2).

What’s changed:

- Config: Fork out to git binary for config data
- Add tests for gitsign-attest

#### Rekor

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. Its latest release is [v1.0.0-rc.1](https://github.com/sigstore/rekor/releases/tag/v1.0.0-rc.1).

What’s changed:

- Add retry command line flag on rekor-cli
- Add some info and debug logging to commonly used funcs
- Add CHANGELOG.md for v1.0.0-rc.1

### Join the Community!

This month is [Hacktoberfest](https://blog.sigstore.dev/contribute-to-sigstore-during-hacktoberfest-2022-42315a41da5f)! Get started with Sigstore there :)

Otherwise, new contributors and users are always welcome into our community. We take pride in being friendly to new folks and fostering a welcoming and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks.

Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others.

Join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and come say hello! 👋
