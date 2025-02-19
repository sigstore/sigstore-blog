+++

title = "Sigstore Update — August 2022"
date = "2022-08-09"
tags = ["sigstore","keyless","softwaresupplychain","npm","kubecon"]
draft = false
author = "Tracy Miranda"
type = "post"

+++

It has been very busy in the Sigstore community over the past few weeks with lots of activity and initiatives progressing at speed. With so many exciting things happening it’s hard to keep up, but here’s a summary of the highlights from the past month.

### NPM set to adopt Sigstore

GitHub just announced a new request for comments (RFC) for [linking packages to their source and build environment](https://github.blog/2022-08-08-new-request-for-comments-on-improving-npm-security-with-sigstore-is-now-open/) for the npm package manager. As part of the RFC, Sigstore was selected as the solution to signing npm packages as it is the only working solution that met the key requirements:

- Links packages hosted on the npm registry to the source and build they originated from (provenance information).
- Supports signing identities that don’t expose any personally identifiable information (PII) about maintainers, e.g. emails.
- Avoids developer-managed keys (as there’s no good way to offer trust to the community given the challenges of distributing public keys).

As part of the implementation, GitHub has been working on a native JS implementation of the Sigstore signing/verification mechanics called sigstore-js. Github recently donated the library to the Sigstore project — which is very exciting!

The tool supports:

- Signing npm packages using an OpenID Connect identity
- Publishing signatures to a Rekor instance
- Verifying signatures on npm packages

Learn more at: https://github.com/sigstore/sigstore-js

### SigstoreCon

Great news! Sigstore will host its [SigstoreCon](https://blog.sigstore.dev/announcing-sigstorecon-2022-dad650d83802), its first official event, on October 25 in Detroit, in co-location with KubeCon + CloudNativeCon North America.

The Call for Papers is open until August 19 ➜ [Submit a talk](https://events.linuxfoundation.org/sigstorecon-north-america/program/cfp/)

Sponsorship options are open until August 12 ➜ [View prospectus (page 24)](https://events.linuxfoundation.org/sponsor-cncf-events)

Learn more [in this post](https://blog.sigstore.dev/announcing-sigstorecon-2022-dad650d83802)

![](/images/sigstorecon.jpg)

### 3 Million Rekor Entries

Hurray, Rekor reached 3 Million entries! This means that in a little over 6 months, 2 million new entries were logged.

Read the post from January 10, 2021, when we hit [1 million](https://blog.sigstore.dev/celebrating-1-000-000-entries-in-rekor-1950b7c150df).

### Sigstore Expands Technical Steering Committee

With so much activity happening in the community the time was right to expand Sigstore’s Technical Steering Committee from 3 to 5 members. We are thrilled to welcome Trevor Rosen (GitHub) and Santiago Torres Arias (Purdue University) to the committee.

### New: Sigstore Office Hours

We recently launched Sigstore Office Hours, which will take place every other Tuesday. Sigstore Office Hours feature demos from Sigstore users and language integrators as well as follow on discussions. They are open to folks new to the community who are interested in adopting Sigstore. The next one will be on August 16 at 12:30 EST. Follow the [Sigstore Calendar](https://t.co/RSd1uMg3sP) to get all the details.

Watch the recordings of the previous ones:

- [Sigstore Office Hours — Aug 2](https://youtu.be/xxVimHB7nwg)
- [Sigstore Office Hours — July 19](https://youtu.be/LeVS9s66nXA)

### Sigstore Audit

The [Open Source Technology Improvement Fund (OSTIF)](https://ostif.org/) did an audit of Sigstore. They found and fixed a high-risk vulnerability.

Read all about it [here](https://ostif.org/our-audit-of-sigstore-is-complete-high-risk-vulnerability-found-and-fixed/).

### Blog Posts and Talks

In case you missed them, there were a few recent blog posts:

- [Adopting Sigstore Incrementally](https://blog.sigstore.dev/adopting-sigstore-incrementally-1b56a69b8c15) by Hayden Blauzvern
- [Is Sigstore Ready for a Post-Quantum World?](https://medium.com/sigstore/is-sigstore-ready-for-a-post-quantum-world-82c9166985af) by Zachary Newman
- [Scaffolding Sigstore](https://medium.com/sigstore/scaffolding-sigstore-e893eb962f22) by Andrew Block

Priya Wadhwa gave a Sigstore talk called “Demystifying Digital Signatures” at the [OpenSSF Day](https://events.linuxfoundation.org/open-source-summit-north-america/features/openssf-day/) on June 20. [Watch her talk](https://youtu.be/KpyYVLHY8V8) to learn more about Sigstore and check out the full playlist of the [OpenSSF Day recordings](https://www.youtube.com/playlist?list=PLVl2hFL_zAh_T8vXM0gdjvfZppAPMAheA).

### New Releases

#### Sigstore

Sigstore is currently on[ version 1.3.1](https://github.com/sigstore/sigstore/releases/tag/v1.3.1), released at the end of July 2022.

Thank you and welcome to our new contributors:

- [asraa](https://github.com/asraa)
- [Miloslav Trmač](https://github.com/mtrmac)

#### Cosign

Cosign is container signing, verification and storage in an OCI registry. Its latest release is [v1.10.1](https://github.com/sigstore/cosign/releases/tag/v1.10.1), which fixed a security issue.

Thank you and welcome to its most recent contributors:

- [Azeem Shaikh](https://github.com/azeemshaikh38)
- [Ciara Carey](https://github.com/ciaracarey)
- [Frederik Boster](https://github.com/Syquel)
- [Jinhong Brejnholt](https://github.com/JBrejnholt)
- [Masahiro331](https://github.com/masahiro331)
- [saso](https://github.com/otms61)
- [Tobias Trabelsi](https://github.com/Lerentis)
- [William Woodruff](https://github.com/woodruffw)

#### Fulcio

Fulcio is a new kind of root CA for code signing. It’s a work in progress, but it’s coming along nicely. Its most recent release can be viewed here: [Release v0.5.2](https://github.com/sigstore/fulcio/releases/tag/v0.5.2).

Thank you and welcome to our newest contributor: [William Woodruff](https://github.com/woodruffw)

#### Gitsign

Keyless Git signing with Sigstore! Its latest release is [v0.2.0](https://github.com/sigstore/gitsign/releases/tag/v0.2.0).

Release highlights:

- Adds gitsign-credential-cache: an optional socket-based credential cache binary for reusing keys for multiple signing requests without needing to reauth (e.g. rebases).
- Adds support for out-of-band interactive flows to add support for SSH and other sessions where web browsers are not directly present.
- Signing errors will now be output to the user TTY directly if available.
- Fixed Rekor Git SHA generation for tags.

#### Rekor

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. View its latest [release v0.10.0](https://github.com/sigstore/rekor/releases/tag/v0.10.0).

Thank you and welcome Rekor’s newest contributor: [Azeem Shaikh](https://github.com/azeemshaikh38)

### YouTube Channel

Did you know? Sigstore has a [YouTube Channel](https://www.youtube.com/channel/UCWPVc8glVGOODxsA_ep0VVw). Subscribe for the latest videos!

If you recently gave a Sigstore talk, let us know and we’ll add it to our [Community Talks Playlist](https://www.youtube.com/playlist?list=PLM6mY5TOhY1E1rqcBR93goCd0DaAiyU8f).

### Get Involved!

As always, we truly welcome contributors and users to our community. We take pride in being friendly to new folks and fostering a welcome and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks. Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others.

Come and join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and say hello!