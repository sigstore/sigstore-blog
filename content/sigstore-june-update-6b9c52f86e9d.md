+++
title = "Sigstore June Update!"
date = "2021-06-30"
tags = ["sigstore","infosec","security","kubernetes"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

Another month, another set of exciting updates! The [Sigstore](http://sigstore.dev/) community has been working at a ferocious pace to harden our platforms and tools, while working on the larger picture of supply-chain security. The pieces are coming together, and the bigger vision of OSS supply chain transparency is getting a little less blurry.

![](/images/jun21.jpg)

This means the Sigstore community is starting to engage deeper in our peer communities as we integrate and share knowledge in both directions. We want to thank them all for their help! In particular, the [TektonCD](http://tekton.dev/), [In-Toto](http://in-toto.io/), [TUF](https://theupdateframework.io/), [SPIFFE/SPIRE](http://spiffe.io/) and [CNCF TAG Security](https://github.com/cncf/tag-security) groups have been immensely helpful. You can see some recordings from these meetings [here](https://www.youtube.com/watch?v=TVNyL2_ljtg).

Let’s start with the numbers, and then jump into project updates! This month, we jumped up to 675 commits from 14 organizations. Activity is still going up and to the right, which is awesome to see!

![](/images/jun211.png)

### Trust Roots!

Before project updates, we also need to cover the Root Key Ceremony! The Sigstore community welcomed our first set of five Key Holders on June 18th, where we signed our initial [TUF Trust Root](http://github.com/sigstore/root-signing). We designed this process to be transparent, secure, and most importantly — fun, and thanks to the amazing help from [DanPOP](https://twitter.com/danpopnyc) and the entire team behind [CloudNative.tv,](http://cloudnative.tv/) the event lived up to expectations! We had over 80 viewers throughout the entire process, and dozens of community members verified each step of the process.

This root will be used to protect all of the keys we use to sign things across the Sigstore project, from our transparency log to our root CA, all the way down to the individual binaries we release. These delegations will be managed via The Update Framework. Up next, we’ll begin a continuous process to rotate our five Key Holders across the open source community every four months. This process got quite a bit of [media coverage](https://www.wired.com/story/sigstore-open-source-supply-chain-code-signing/), and we want to thank everyone again!

![](/images/jun212.png)

### Cosign

Cosign is rapidly approaching our first 1.0 release! The current plan is to release v0.6.0 with our final round of large changes, then target a 1.0 after a short public comment period. We’re comfortable with the codebase now and are already using it in production, but we want to get one final round of feedback from users that might be waiting until things are more stable before taking a close look. Watch out for this call for feedback soon!

In other highlights, we recently added support for:

- Improved [hardware key](https://github.com/sigstore/cosign/pull/369) signing
- Direct secrets management for Kubernetes environments
- A bunch of UX around managing, cleaning up, and copying signatures and other attached artifacts (like SBOMs) across registries
- OIDC Identity Token based authentication, for keyless automated builds

We’re also seeing the [Cosign specification](https://github.com/sigstore/cosign/blob/main/SPEC.md) supported across other projects! So far we’ve seen:

- Integration with TektonCD Chains!
- [Buildpack](https://github.com/buildpacks/pack/issues/268)/[kpack](https://github.com/pivotal/kpack) support
- [Kyverno](https://github.com/kyverno/kyverno/tree/feature/cosign) for Kubernetes policy
- and more!

OCI isn’t just for containers, so we’re also starting to tackle the general problem of attaching and storing other forms of data in an OCI registry. Software Bill of Materials files, [WASM modules](https://github.com/solo-io/wasm-image-spec), [Tekton bundles](https://tekton.dev/docs/pipelines/tekton-bundle-contracts/) and more can all be uploaded and stored just like container images, but there’s no great tooling to manage or sign these today. We’re hoping cosign can support all of the well-known artifact types over time.

We’re also getting close to completing the `cosign tuf` implementation! [Asra Ali](https://twitter.com/AsraEntr0py) has been working on a distributed key-generation and root signing process, based on her work designing the [Root Key Ceremony](https://www.youtube.com/watch?v=GEuFsc8Zm9U)! You can see the demo [here](https://www.youtube.com/watch?v=3d-hhZKh3sw&t=102s) if you’re interested! This [TUF integration](https://github.com/sigstore/cosign/pull/366) will hopefully address many of the problems that the Notary v1 project faced, by solving “Trust on First Use” with the use of our transparency log. The Sigstore projects are designed to work separately, but are even better together!

### Fulcio

[Fulcio](http://github.com/sigstore/fulcio) (our free Root CA) has also picked up the pace recently! We finally have a federation design in place that will allow us to support arbitrary domains, beyond the large email-based OIDC providers. Rather than reinventing the wheel, we’ve decided to base this off of the work of another great OSS project — [SPIFFE and SPIRE](http://spiffe.io/).

The most exciting part of this plan is that these SPIFFE identifiers can be generated and managed internally, even in offline or hybrid environments. These hybrid identifiers can then be exchanged between organizations and validated using the standard OIDC protocol. This plan is taking place across a few projects, and we’re referring to this overall architecture as “Zero Trust Supply Chains”. Read some more here in our [whitepaper](https://github.com/sigstore/community/blob/main/docs/zero-trust-supply-chains.pdf).

We’re also working on making it easier to run Fulcio in disconnected environments, with the ability to retrieve a subordinate cert from the official Sigstore Trust Root. This process will be manual at first, but we hope to eventually automate it. If you’re interested in using either federation model (subordinate or direct) for your OIDC endpoints, please reach [out here](https://github.com/sigstore/fulcio/issues/122)!

### Rekor

Last but not least, we have [Rekor](http://github.com/sigstore/rekor) — our Binary Transparency log. Rekor has also had a busy month, with a few new features landing and some even more exciting features planned! We cut and published our [v0.2.0 release](https://github.com/sigstore/rekor/releases/tag/v0.2.0), which included support for new types, including [In-Toto Attestations](https://github.com/in-toto/attestation) and [RFC3161 Timestamps](https://www.ietf.org/rfc/rfc3161.txt).

These new types also bring new transparency features. In-Toto Attestations include rich metadata around exactly how artifacts are produced. Instead of throwing this valuable data away and only storing a proof of existence, we now have support for storing and querying this full structured data. This allows for full *Attestation Transparency*, a step beyond just plain *Binary Transparency.* This work is new, but exciting! The goal is to actually allow us to verifiably trace an artifact back to its sources and build steps, recursively, across a distributed supply chain!

The new [RFC3161](https://www.ietf.org/rfc/rfc3161.txt) Timestamp type unlocks “Time Transparency”. This means Rekor can act as a Timestamp Authority, without requiring end users to trust Rekor! The timestamps generated and signed can be logged directly into our transparency log, allowing anyone to audit and verify these for accuracy later. This is very similar to the [Roughtime](https://blog.cloudflare.com/roughtime/) work, but packaged up slightly differently to optimize for supply-chain use-cases rather than network or service authentication. This timestamp format is unfortunately a bit dated, so we’re also working to [design simpler formats](https://github.com/sigstore/rekor/issues/315) that are easier to use and understand for authenticated timestamps.

### Wrapping Up

This post contains only a small portion of the features that have landed in Sigstore over the last month and the work that’s ongoing! We’d like to thank the community again — we wouldn’t be here without your help and support. If you’re looking to get involved in any of these efforts, please reach out! Our [slack channel](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ), [email list](http://sigstore-dev.googlegroups.com/) or [GitHub organization](http://github.com/sigstore) all work to find us!
