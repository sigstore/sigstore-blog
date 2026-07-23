+++
title = "Rekor Public-Good Instance Hardening"
date = "2026-07-24"
tags = ["sigstore","rekor"]
draft = false
author = "Hayden Blauzvern (Google)"
type = "post"
+++

# **Hardening Sigstore Public Infrastructure**

To harden our public-good infrastructure, reduce attack surfaces, and ensure seamless interoperability
across the developer ecosystem, **we are reducing the number of supported entry types on the Sigstore public deployment transparency log.**

This change will have **no impact** for users of Cosign and Sigstore SDKs, as well as integrators using these client libraries.

This decision is driven by a lack of client support and adoption for several older or experimental entry types.
By narrowing our focus to a core set of highly adopted, well-supported schemas, we can deliver a more secure,
performant, and reliable transparency log for everyone.

**Why Streamline Entry Types?**

By deprecating under-utilized entry types, we:

* **Harden the Infrastructure:** Reducing the API surface directly reduces the risk of parsing vulnerabilities.
* **Standardize & Eliminate Client Fragmentation:** It ensures the public deployment matches the capabilities of our officially supported Sigstore SDKs, such that any Sigstore client can seamlessly verify signatures generated across the entire ecosystem.
* **Optimize Performance:** The community can focus testing, auditing, and optimization efforts on the core entry types that power the vast majority of the ecosystem.

**Supported Entry Types Moving Forward**

The Sigstore public deployment will exclusively support the following entry types:

1. `hashedrekord`: The standard format representing a hashed artifact and its signature.
2. `dsse`: Used for in-toto attestation signing.
3. `intoto` (versions v0.0.1 and v0.0.2): A previous revision of the `dsse` type, still supported by older clients.  
4. `helm`: Supporting [Helm chart provenance](https://github.com/sigstore/helm-sigstore). Please reach out if you're relying on this plugin or open to contributing changes to make it conformant with other clients.

## **SDK Recommendation for Developers**

If you are developing or maintaining a Sigstore client or integration, **your implementation should exclusively generate** `hashedrekord` **entries**.
This type represents the well-supported path for the vast majority of signing use cases, including container signing, blob signing, and general attestation envelopes.
Optionally, when integrating with Rekor v1, your implementation may generate `dsse` entries for server-side payload generation, although large attestations are unsupported.

For newer integrations, we recommend [leveraging Rekor v2](https://blog.sigstore.dev/rekor-evolution/), which is well-optimized for all client payloads, including large attestations.

**What Does This Mean for You?**

* **For Most Users:** If you use official Sigstore tools (like Cosign) or official language SDKs (Go, Python, JS, Rust, Ruby), **no action is required.** These tools already default to the supported types.
* **For Private Deployments:** These additional types continue to be supported in the codebase, though we recommend migrating to use `hashedrekord`.
* **For Custom Integrations:** If you have built custom client integrations that rely on unsupported Rekor entry types, you will need to migrate your client payload generation to use `hashedrekord` or `dsse`.

If you have any questions, feel free to reach out on the Sigstore Slack or open an issue on the [Rekor GitHub repository](https://github.com/sigstore/rekor/issues).
