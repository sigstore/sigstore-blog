+++
title = "Cosign 1.0!"
date = "2021-07-28"
tags = ["sigstore","docker","kubernetes","security"]
draft = false
author = "Dan Lorenc"
type = "post"
+++

The [cosign](http://github.com/sigstore/cosign) project started in [February 2021](https://github.com/sigstore/cosign/commit/587bb41f822e54d235c6d464330f8c2faab7aba1) with a goal of making it easy to sign and verify containers on any OCI registry today. The community support has been incredible! We’ve added 7 maintainers from 5 organizations, and have merged [394 commits](https://github.com/sigstore/cosign/commits/main) from 32 contributors across 10 organizations. Cosign has been tested on [13 OCI registries](https://github.com/sigstore/cosign#registry-support) and is now packaged in five different package managers. We’ve cut [seven releases](https://github.com/sigstore/cosign) over six months and are now thrilled to declare our first general availability release, [cosign 1.0](https://github.com/sigstore/cosign/releases/tag/v1.0.0), which is **ready for production use**!

![](/images/cosign1.jpg)

Photo by [Kirill Sh](https://unsplash.com/@kirill2020?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

This means that the core feature set of signing and verifying artifacts sorted in OCI registries, as well as underlying specifications for storing and discovering signatures are considered stable and ready for production use. We’ve been using this [tooling ourselves](https://security.googleblog.com/2021/05/making-internet-more-secure-one-signed.html) for three months now to sign and verify releases of the [distroless](http://github.com/GoogleContainerTools/distroless) container images, and are now ready to help the rest of the industry secure their supply chains!

To get started, you can install cosign via a few different package managers and start signing! Give it a try [here](http://github.com/sigstore/cosign) with just a few commands, or integrate it into your build and deployment flows automatically.

### What’s next?

As cliche as it sounds, this really is just the beginning! We’re working on better key management stories, rich attestation support, SBOM integrations, end to end policy system support, and stabilizing the rest of the “experimental” features that make cosign work well with the rest of the sigstore projects. Let’s jump into the specifics!

We have quite a few improvements planned for key management and distribution. Up next, we’re working to add support for [The Update Framework](https://theupdateframework.io/), which will provide a battle-tested approach for resilience and compromise recovery. An early (but working) prototype can be seen in [action here](https://github.com/sigstore/cosign/pull/366). This integration will support all of the existing key management options (hardware keys, cloud KMS support, Kubernetes secrets, etc.) to the broader TUF community. In addition, we’re also working on several enhancements to TUF itself (through TUF Augmentation Proposals), including support for our “keyless” OIDC-based keys.

Signatures are only one piece of the overall supply chain integrity story, so we’re also working on rich attestation support using the [In-toto](https://github.com/in-toto/in-toto) attestation model. This will provide a flexible system for storing verifiable metadata, including things like SBOMs and vulnerability scans. See here for some more working demos and early plans!

### Beyond Cosign!

We believe in specifications driven by working code, and see cosign as just one implementation of the overall signature format and ecosystem. This means we need tight integration with build systems and policy engines to make signatures automatic and invisible. We plan on taking the Tekton Chains project to beta later this summer, and GA by the end of the year.

We’ll also keep working on integrations with other build systems like kpack, Jenkins X, Jenkins, Google Cloud Build, Github Actions, Gitlab Runner and more! Help here is needed and welcomed! We’re still at the start of a long road to automatic supply chain security, and now is the best time to get involved.

### Across Sigstore

Across the rest of [Sigstore](http://sigstore.dev/), we’ve also seen rapid contributor and ecosystem growth! We’re now up to [703 commits from 50 different contributors, across 16 organizations!](https://insights.lfx.linuxfoundation.org/projects/sigstore/dashboard;subTab=technical)

![](/images/cosign2.jpg)

### Rekor

The Rekor transparency log has seen a ton of activity lately too, with support for several new types and features. Since our last release, we’ve added support for [Helm provenance](https://github.com/sigstore/rekor/pull/354), [Alpine Packages](https://github.com/sigstore/rekor/pull/337) and [In-Toto Attestations](https://github.com/sigstore/rekor/pull/364). The In-Toto support builds on our rich attestation storage and [Attestation Transparency](https://docs.google.com/document/d/12JKXypgnDNQ2NBJYR73zLktSXj2pfBJcQWl675n_VdE/edit#heading=h.ramqb693vq3c) designs, allowing us to store and index full build provenance metadata, instead of just the signature and public key information. This indexing will allow users to store and query the full build history for any artifact, without needing to worry about attaching the attestations directly to artifacts.

We’re also finishing up some exciting trusted timestamp work, which will allow Rekor to act as a free timestamp authority for any other projects interested in improving their key revocation and management stories. Look out for some more blog posts and demos on this later in August!

Rekor (and transparency in general) is the cornerstone to the rest of the projects in Sigstore, so we’ll begin to focus on operationalizing and stabilizing the Rekor infrastructure next. This means we’ll finally be able to remove the scary “EXPERIMENTAL” and “DANGER” warnings from everything that’s stored in Rekor. We’re hoping to finish this up by early September.

### Fulcio

Fulcio is our free code signing CA, built to make short-lived x509 certificates available to anyone. We originally designed fulcio to run as a centralized, public-good instance backed up by our other transparency logs, but we’ve seen significant interest from large organizations to run this CA internally, with various forms of federation back up to our main trust root. To help out here, we’ve started working on making it easier to support these delegation models, and to deploy and run your own disconnected Fulcio instance.

We’ve recently added support for configurable backing CA services, so you can now use Fulcio with anything that speaks the standard PKCS11 protocol. This builds on our existing support for the Google Cloud Certificate Authority Service, which we use in production. See [here](https://github.com/sigstore/fulcio#fuclioca-pkcs11) for some docs on how to set this up with SoftHSM, or other HSMs you might be running in production.

The federation process is also coming together, and we’ve started allowing OIDC-based systems that use the industry-standard SPIFFE identity protocol to opt-in to direct federation via a pull-request on GitHub! See the documentation for how to try this out yourself [here](https://github.com/sigstore/fulcio/tree/main/federation#oidc-federation-configs).

### Other Updates

In other updates, [Andrew Block](https://twitter.com/sabre1041) just released the first version of the [Sigstore Helm plugin](https://github.com/sigstore/helm-sigstore), to make signing and verification of Helm charts using the Rekor transparency log easy! He also built and documented a Helm chart based approach to deploying the entire sigstore stack [here](https://github.com/sigstore/).

In the policy space, things are moving quickly too! [Batuhan Apaydın](https://twitter.com/developerguyba) and [Furkan Türkal](https://twitter.com/furkanturkai) have built a prototype integration for verifying cosign signatures using OPA/Gatekeeper, and documented it with some amazing diagrams over [here](https://github.com/developer-guy/container-image-sign-and-verify-with-cosign-and-opa) and Jim Bugwadia merged an [initial version](https://github.com/kyverno/kyverno/pull/2078) of cosign support into [Kyverno](https://www.google.com/search?q=kyverno&oq=kyverno&aqs=chrome.0.69i59j0l4j69i60l2j69i61.805j1j7&sourceid=chrome&ie=UTF-8), which should be going out in the next release. Last but not least, [Connaisseur 2.0](https://sse-secure-systems.github.io/connaisseur/v2.0.0/) has also just been released, which provides support for the cosign and Notary v1 signature specifications.

Finally, these updates are getting longer and longer to write because so much activity is going on! We’d like to again thank the community for their support! We’re probably going to move to more frequent updates at the project level to help scale this process without losing this important communication mechanism. If you’re using sigstore in any way, feel free to reach out over Slack or [Twitter](http://twitter.com/lorenc_dan).