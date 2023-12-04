+++

title = "Sigstore November Roundup"
date = "2023-11-30"
tags = ["sigstore","securesupplychain"]
draft = false
author = "Luke Hinds (TSC Chair)"
type = "post"

+++

Welcome to the November edition of the Sigstore Roundup! This is a regular summary of Sigstore news, events, releases and other happenings.

### Sigstore Google Season of Docs 2023 Case Study
 
A very comprehisive case study has been published on the [Sigstore docs wiki](https://github.com/sigstore/docs/wiki/Sigstore-Google-Season-of-Docs-2023-Case-Study) about the Sigstore project's participation in the 2023 program.

Thank you Lisa Tagliaferri for all your hard work on this and making it a success!

### Latest Releases

#### Rekor v1.3.3

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain.

Check out the last major release [v1.3.1](https://github.com/sigstore/rekor/releases/tag/v1.3.1), made since we last updated you.

- Enable GCP cloud profiling on rekor-server
- Move index storage into interface
- Add type of ed25519 key for TUF
- Allow parsing base64-encoded TUF metadata and root content 

#### Cosign v2.2.1

Cosign is container signing, verification and storage in an OCI registry. Its latest release is [v2.2.1](https://github.com/sigstore/cosign/releases/tag/v2.2.1).

Some of the main new features include:

- Support basic auth and bearer auth login to registry
- COSIGN_PKCS11_IGNORE_CERTIFICATE environment variable to skip loading certificates into a PKCS11 key when set to "1".
- Cosign triangulate now supports image digest retrieval from OCI registries
- Attach rekor bundle to a container image
- Add support outputting rekor response on signing

#### Gitsign v0.8.0

Keyless Git signing with Sigstore! Its latest release is [v0.8.0](https://github.com/sigstore/gitsign/releases/tag/v0.8.0).

What’s changed:

- Add options for Rekor client, make public key fetcher configurable.
- Add gitsign initialize. (#321)
- Fix offline verification marshalling, add e2e tests.

### Sigstore Libraries

Le'ts take a look at the latest releases / updates of the Sigstore libraries.

#### sigstore-rs v0.7.3

A Rust library for interacting with Sigstore. Its latest release is [v0.7.3](https://github.com/sigstore/sigstore-rs/releases/tag/v0.7.3).

What’s changed:

- sigstore-rs now supports use of a TUF trustroot. This allows for the use of a TUF repository as a trust root for verifying signatures.

#### sigstore-java v0.5.0

Lots of new changes in the sigstore java library. Its latest release is [v0.5.0](https://github.com/sigstore/sigstore-java/releases/tag/v0.5.0)

What’s changed:

- BYOB-based SLSA-generator
- pkix der encoded key parsing
- Add accessors to trustroot 

#### sigstore-go

While not at its first release, development on the Go library is still ongoing!

Do check it out where there a lots of examples on using the library [see here](https://github.com/sigstore/sigstore-go#examples)

### sigstore-python

The Python library is also still under active development. Check out the last major release [v2.0.0](https://github.com/sigstore/sigstore-python/releases/tag/v2.0.0), made since we last updated you.

- CLI: sigstore sign and sigstore get-identity-token now support the
--oauth-force-oob option; which has the same behavior as the
preexisting `SIGSTORE_OAUTH_FORCE_OOB` environment variable
- Version 0.2 of the Sigstore bundle format is now supported
- API addition: VerificationMaterials.to_bundle() is a new public API for
producing a standard Sigstore bundle from sigstore-python's internal
representation 
- API addition: New method sign.SigningResult.to_bundle() allows signing
applications to serialize to the bundle format that is already usable in
verification with verify.VerificationMaterials.from_bundle()

### sigstore-js

The JavaScript library (used to power NPM provenannce) is also still under 
active development. Check out the last major release [2.1.0](https://github.com/sigstore/sigstore-js/releases/tag/sigstore%402.1.0)

sigstore-js is one of the our more mature libraries now, so changes are mostly
bug fixes and minor improvements.

### In the News

- JPMorgan’s Global CISO urges use of Sigstore, Alpha-Omega in open source security drive [read more](https://www.thestack.technology/jpmorgans-global-ciso-use-sigstore-alpha-omega/)
- Sigstore: Simplifying Code Signing for Open Source Ecosystems [read more](https://openssf.org/blog/2023/11/21/sigstore-simplifying-code-signing-for-open-source-ecosystems/)
- Stacklok Builds on Sigstore to Identify Safe Open Source Libraries [read more](https://thenewstack.io/stacklok-builds-on-sigstore-to-identify-safe-open-source-libraries/)
- Wind River Further Expands VxWorks RTOS Containers Leadership with Cosign Support [read more](https://www.busistacklnesswire.com/news/home/20231101614010/en/)

### Join the Community!

New contributors and users are always welcome into our community. We take pride in being friendly to new folks and fostering a welcoming and safe environment. Being a large open source project, there is always so much to do, not all of them being complex coding tasks.

Valued contributions include: helping with documentation, general testing, and sharing your love of Sigstore with others.

Join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ) and come say hello! 👋