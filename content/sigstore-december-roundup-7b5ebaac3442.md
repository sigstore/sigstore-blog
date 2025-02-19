+++
title = "Sigstore December Roundup"
date = "2022-12-23"
tags = ["sigstore","security","softwaresupplychain","supplychainsecurity","npm"]
draft = false
author = "Sigstore"
type = "post"

+++

“*And lo, in the land of software package management, a system was born to bring order and trust. Sigstore was its name, and its mission was to sign packages with short-lived certificates, validated by a powerful OIDC provider. These signed packages were then placed in a transparency database for all to see, like a holy book open for all to read and verify. Sigstore was a beacon of hope in a chaotic world, shining brightly as a protector of software integrity.*”

Thanks, Daniel Feldman for [generating this (and many other)](https://twitter.com/d_feldman/status/1602405219928838144) Sigstore descriptions.

However you describe Sigstore, it’s undeniable that 2022 has been an incredible year for the project and its community. Not only was this the year of the [Sigstore GA](https://blog.sigstore.dev/sigstore-ga-ddd6ba67894d), but the project had an overwhelming amount of contributors:

- 450+ Contributing Individuals
- 70+ Contributing Organizations

### New Scientific Paper: “Sigstore: Software Signing For Everybody”

A peer-reviewed research paper called *Sigstore: Software Signing for Everybody* authored by Zachary Newman, John Speed Meyers, and Santiago Torres-Arias was published at the 2022 ACM Computer and Communications Security (CCS) conference in Los Angeles, CA, an academic computer security conference, featuring publications from research universities around the world and industry labs at organizations like Google, Microsoft, Meta, and Amazon.

📄 [Read the paper](https://blog.sigstore.dev/sigstore-software-signing-for-everybody-has-been-published-in-the-proceedings-of-the-acm-bb7e7d679a73)

When this paper was drafted, 10 months ago, there were 2 million entries in the Rekor log; now there are over 7 million and counting!

### New Case Studies

Many companies are adopting Sigstore and are excited to share their story. We published three new Sigstore case studies from Autodesk, DB Schenker, and Verizon since the last roundup!

[**Using Sigstore to meet FedRAMP Compliance at Autodesk**](https://blog.sigstore.dev/using-sigstore-to-meet-fedramp-compliance-at-autodesk-6f645a920abc) by Jesse Sanford

![https://blog.sigstore.dev/using-sigstore-to-meet-fedramp-compliance-at-autodesk-6f645a920abc](/images/autodesk1.png)

[**Securing Your Software Supply Chain Without Changing Your DevOps WorkflowDB Schenker**](https://blog.sigstore.dev/securing-your-software-supply-chain-without-changing-your-devops-workflow-e23393a5fffa) by Tobias Trabelsi

![https://blog.sigstore.dev/securing-your-software-supply-chain-without-changing-your-devops-workflow-e23393a5fffa](/images/casestudy1.png)

[**Verizon: Security by Default: How Verizon New Business Incubation Uses Sigstore to Demonstrate Provenance and Improve Customer Confidence**](https://blog.sigstore.dev/security-by-default-how-verizon-new-business-incubation-uses-sigstore-to-demonstrate-provenance-7beed5714738) by Aaron Bacchi

![https://blog.sigstore.dev/security-by-default-how-verizon-new-business-incubation-uses-sigstore-to-demonstrate-provenance-7beed5714738](/images/verizon.png)

### Sigstore Landscape

The [Sigstore Landscape](https://blog.sigstore.dev/new-sigstore-landscape-add-your-signed-project-dda0517723b6) is filled up with projects signed by Sigstore. We want to give a shout-out to the latest additions: FluentBit, Istio, Karpenter, Keptn, Knative, Kubewarden, LinkerD, Pulumi, and Shipwright.

Also a special mention for Sigstore adoptions that aren’t yet on the landscape:

**LLVM:** Now signs with Sigstore to make it easier for users to verify that the packages came from llvm and to detect potential malicious signatures. Find it on [apt repo for Debian/Ubuntu](https://apt.llvm.org/) packages.

**Updatecli:** The latest release is now signed with Sigstore. [Read more](https://www.updatecli.io/)

**Kubernetes release:** The recent 1.26 Kubernetes release now signs all software artifacts with Sigstore, not just the container images. [Read more](https://venturebeat.com/data-infrastructure/new-kubernetes-1-26-release-boosts-security-storage-teases-dynamic-resource-allocation)

### New Content

Our community has been busy with new Sigstore Content including:

- [Signatus, ergo securus? Who can sign what with TUF and Sigstore](https://blog.sigstore.dev/signatus-ergo-securus-who-can-sign-what-with-tuf-and-sigstore-ea4d3d84b8b6) by Zack Newman
- [How to become the next Sigstore Evangelist?](https://blog.sigstore.dev/how-to-become-the-next-sigstore-evangelist-9303ed297e54) by Batuhan Apaydin (developer-guy)
- [Sigstore the easy way](https://rewanthtammana.com/sigstore-the-easy-way/index.html) by Rewanth Tammana

### New Community Talks

Don’t miss the newest Sigstore community talks including this keynote: *What does Sigstore get you as a Kubernetes operator?* by Luke Hinds at Kubernetes Community Days UK 2022. View the full [Sigstore Community Talk playlist](https://youtube.com/playlist?list=PLM6mY5TOhY1E1rqcBR93goCd0DaAiyU8f).

{{< youtube xrPzAetGhzY >}}

### Language Client Updates

As many language ecosystems look to adopt Sigstore, work is underway to make it much, much easier. Here’s the latest on language client activity.

#### Java (sigstore-java)

A common sigstore-java library is being actively developed to be integrated into Java ecosystem tools such as Maven and Gradle. If you’d like to get involved in 2023 or keep up with the latest, please join the [Sigstore Java](https://docs.google.com/document/d/1R7mL-IUrc2Z_LuOIvwDWshVuPQS_2VNE_cIQx4Oy5zw/edit) weekly calls.

#### Javascript (sigstore-js)

The npm community recently [accepted the RFC](https://github.com/npm/rfcs/blob/main/accepted/0049-link-packages-to-source-and-build.md) to improve trust of npm packages using Sigstore and trusted build infrastructure. To integrate Sigstore into the npm cli directly, work is underway on a cosign client writing in javascript: sigstore-js. At a recent Sigstore Office Hours Brian de Hamer gave a [demo on the latest with sigstore-js](https://youtu.be/sOSFWWWwVkc?t=968).

#### Python (sigstore-python)

For Python, sigstore-python is working towards its 1.0 release, including work toward stabilizing an importable Python API. sigstore-python 0.9.0 [has just been released](https://twitter.com/8x5clPW2/status/1605996324167356416?s=20&t=bEOmFZgqfWf_1hum4gv-KA) and becomes the first version to use TUF to automagically establish trust in Sigstore’s public good instances. William Woodruff recently gave a demo of sigstore-python at Sigstore Office Hours. The demo included an overview of new policy support, [check out the video](https://www.youtube.com/watch?v=lcCwXgHGV00&feature=youtu.be).

#### Rust (sigstore-rs)

Sigstore-rs is a crate for Rust to interact with Sigstore and [v0.6.0](https://github.com/sigstore/sigstore-rs/releases/tag/v0.6.0) just shipped. This release includes a ton of fixes and support for OCI Image signing. Sigstore and memory safety, all in one, what’s not to love?

### Latest Releases

#### Sigstore

Sigstore is currently on [version 1.5.0](https://github.com/sigstore/sigstore/releases/tag/v1.5.0).

#### Cosign

Cosign is container signing, verification and storage in an OCI registry.

Its latest release is [v2.0.0-rc.0](https://github.com/sigstore/cosign/releases/tag/v2.0.0-rc.0). This is a pre-release for Cosign 2.0! Feel free to try it out, but know there are many breaking changes from 1.0 and the prereleases may continue to change.

#### Fulcio

Fulcio issues code-signing certificates bound to OpenID Connect identities for use within the Sigstore ecosystem. Its most recent release is [v1.0.0](https://github.com/sigstore/fulcio/releases/tag/v1.0.0).

#### Gitsign

Keyless Git signing with Sigstore! Its latest release is [v0.4.1](https://github.com/sigstore/gitsign/releases/tag/v0.4.1). Amongst other things, this release features new sub-commands:

```
gitsign show - Prints out in-toto Statement for the specified commit.
gitsign attest - Stores attestations for a commit / tree in the repository.
```

#### Rekor

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. Its latest release is [v1.0.1](https://github.com/sigstore/rekor/releases/tag/v1.0.1).

### Join the Community

Thank you to all our contributors and users for making 2022 so wonderful!

We take pride in being friendly to everyone, including new folks, and fostering a welcoming and safe environment. There is always room for more people in our community.

Find us on [Slack](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and Twitter [@projectsigstore](https://twitter.com/projectsigstore)

Think about contributing to Sigstore in 2023!

See you all next year ✨