+++
title = "sigstore.dev and Rekor evolution"
date = "2026-06-28"
tags = ["sigstore","rekor"]
draft = false
author = "Jussi Kukkonen, Google"
type = "post"
+++

If you have been following the Sigstore project over the last year, you know we’ve been rolling out [version 2 of Rekor](https://github.com/sigstore/rekor-tiles/), Sigstore's transparency log. The intent was to enable signing with Rekor v2 by default, but there's been a change of plans: **the sigstore.dev public good instance will continue using Rekor v1 as the default log for the foreseeable future.**

Why are we staying with Rekor v1? While Rekor v2 provides greater infrastructure resiliency, it introduces breaking changes in client behavior. The industry-wide transition to Post-Quantum Cryptography (PQC) will also result in breaking changes to clients. Keeping the public good instance at Rekor v1 may allow us to minimize disruption until absolutely necessary—stay tuned for a separate post about the PQC plans.

### Rekor v2 is Available (and Optimized)

Even though we're not enabling signing by default, **Rekor v2 is running and available in production today**. It is also supported out-of-the-box by modern verifying clients. Ecosystems that manage their own signing processes can choose to opt into Rekor v2. These signing systems should use the up-to-date signing configuration from [root-signing](https://github.com/sigstore/root-signing/blob/main/targets/signing_config_rekor_v2.v0.2.json) (e.g. using a TUF client) instead of hard coding Rekor v2 URLs as the URLs will rotate more frequently.

Rekor v2 has many infrastructure advantages, but recently a client advantage was added that may be interesting to some ecosystems: Rekor v2 is compatible with large DSSE envelopes, as the whole envelope is no longer sent over to the network to be processed by the log. Instead,
the `hashedrekord` entry type is used for higher performance. For more technical details on this change, see the [Client Specification](https://github.com/sigstore/architecture-docs/blob/main/client-spec.md#44-transparency-log-entry) and the [Rekor v2 Specification](https://github.com/sigstore/architecture-docs/blob/main/rekor-v2-spec.md#43-types).

### Client compatibility with Rekor v2

This is an overview of the situation at the time of writing. Please refer to the [Sigstore Conformance](https://github.com/sigstore/sigstore-conformance/) Conformance Report for up-to-date test results and individual client project documentation for details.

**Compatible clients**

* [sigstore/cosign](https://github.com/sigstore/cosign)
* [sigstore/sigstore-go](https://github.com/sigstore/sigstore-go)
* [sigstore/sigstore-java](https://github.com/sigstore/sigstore-java)
* [sigstore/sigstore-js](https://github.com/sigstore/sigstore-js)
* [sigstore/sigstore-python](https://github.com/sigstore/sigstore-python)
* [sigstore/sigstore-rust](https://github.com/sigstore/sigstore-rust)

*All of these clients also support the DSSE-as-hashedrekord feature but may not have released it yet -- please check your client changelog*

**Clients known to not have Rekor v2 support at time of writing**

* [sigstore/sigstore-rs](https://github.com/sigstore/sigstore-rs)
* [sigstore/sigstore-ruby](https://github.com/sigstore/sigstore-ruby)
