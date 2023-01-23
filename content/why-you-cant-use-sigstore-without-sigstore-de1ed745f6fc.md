+++
title = "Why you can’t use Sigstore without Sigstore"
date = "2023-01-06"
tags = ["sigstore","python"]
draft = false
author = "Zachary Newman"
type = "post"
+++

![](/images/sigstore1.jpg)

Photo by [C Dustin](https://unsplash.com/@dianamia?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

I was delighted to see a recent preprint that mentioned Sigstore appear on the IACR’s [Cryptology ePrint Archive](https://eprint.iacr.org/). The reason that we published an academic paper, [Sigstore: Software Signing for Everybody](https://dl.acm.org/doi/10.1145/3548606.3560596), was to encourage the scrutiny of the research community. Progress in the field of computer security only comes from the back-and-forth between proposed defenses and offensive analyses of those techniques, and we welcome third-party analysis of the project.

Called [How to Use Sigstore without Sigstore](https://eprint.iacr.org/2023/003.pdf), the preprint claims that you can achieve the same security properties without any of the constituent components of the system. It’s important to note at this point that a preprint is *not* the same as a published paper, and that works on the ePrint archive have not necessarily been peer-reviewed. Nonetheless, the paper presents a great opportunity to examine why Sigstore is built the way that it is.

In summary:

- The proposal doesn’t *quite* work due to some of the subtleties of real deployments of OpenID Connect.
- Even if it did work, the plan is somewhat circular — it simply shifts responsibility for the features of various Sigstore components to different parties — and loses important security properties.
- Every component of Sigstore is there for a reason: if you think that it doesn’t make sense in your setting, we invite you to chat [in Slack](https://links.sigstore.dev/slack-invite) or [on GitHub](https://github.com/sigstore)!

### How would we eliminate Sigstore?

In this post, we’ll assume familiarity with Sigstore and its components for the “keyless” signing flow; if you need a refresher, the [home page](https://www.sigstore.dev/) contains a nice overview and this post on the [Life of a Sigstore Signature](https://www.chainguard.dev/unchained/life-of-a-sigstore-signature) dives into quite a bit of detail. But to recap, a signer:

1. Authenticates with an OIDC identity provider (think: “Login with Google” flow) to get an OIDC token.
2. Generates an ephemeral signing/verifying key pair.
3. Sends the OIDC token, along with a public key, to [Fulcio](https://github.com/sigstore/fulcio), which issues them an X.509 certificate containing the identity from the OIDC token as its “subject.” Fulcio also publishes this certificate to a [certificate transparency log](https://certificate.transparency.dev/).
4. Signs the artifact (a container image, software package, etc.).
5. Sends the signature (and applicable metadata) to [Rekor](http://github.com/sigstore/rekor), which timestamps the signature and then publishes it to an [artifact transparency log](https://transparency.dev/).

The proposal in the preprint involves two major changes:

1. Embed the public key directly into the OIDC token and disclose the OIDC token itself as the bearer format for the public key.
2. Use a [timestamping authority](https://www.rfc-editor.org/rfc/rfc3161) instead of Rekor.

We address these below, along with miscellaneous minor changes proposed and other criticisms.

### Embedding public keys into OIDC tokens

The preprint proposes a clever hack for embedding public keys into OIDC tokens. Right now, when a signer requests an OIDC token, they ask for a token with the `aud`[ claim](https://www.rfc-editor.org/rfc/rfc7519#section-4.1.3) set to `sigstore`. This prevents Sigstore from receiving a token, then turning around and sending that token to some other service, impersonating the signer.

The hack involves requesting a token with the value of the `aud` claim set to `sigstore?pk=<public key>`, thereby binding the token to a particular key. There are a few issues with this approach:

**It’s not always possible to check OIDC tokens, especially old ones.** The [OIDC spec](https://openid.net/specs/openid-connect-core-1_0.html#RotateSigKeys) (§10.1.1) encourages frequent rotation for the keys used to sign OIDC tokens, and keeping only “recently decommissioned signing keys for a reasonable period of time to facilitate a smooth transition.” This means that older, decommissioned signing keys can be removed, giving users no way to validate the tokens! One solution is to keep a log of the [JWKs](https://openid.net/specs/draft-jones-json-web-key-03.html) (public keys) from the IdPs each time they change — but now we’ve reintroduced a trusted third party, exactly what we were trying to avoid — and a party keeping old keys looks suspiciously like a transparency log (which is one of the roles Fulcio provides).

Even keeping a log of old keys and their validity periods isn’t always sufficient: OAuth allows for tokens that require [interaction with the authorization server](https://datatracker.ietf.org/doc/html/rfc7662) to validate. Having a party like Fulcio allows issuing certificates for such tokens.

**Most providers don’t allow arbitrary** `**aud**` **values.** Many popular identity providers *don’t allow* arbitrary values in the `aud` claim. For instance, [Google OIDC](https://developers.google.com/identity/openid-connect/openid-connect#obtainuserinfo) populates the the value from a register OAuth2.0 application’s client ID and doesn’t allow user-customizable values at all ([Facebook](https://developers.facebook.com/docs/facebook-login/guides/access-tokens/get-long-lived) and [Microsoft](https://learn.microsoft.com/en-us/azure/active-directory/develop/id-tokens#payload-claims) do the same); [in Okta](https://devforum.okta.com/t/access-token-always-contains-default-for-aud/13262/4), the `aud` is configured by the authorization server, not the requester. Even identity providers that *do* support customizable values for this claim often restrict the values to an allowlist.

**OIDC tokens are supposed to remain private.** In theory, if every relying party checks the value of the `aud` claim, you *can* leak a token that’s bound to a particular key pair. Any relying party that accepts a token with that key embedded would know to use their key to validate any associated data. But in practice, this requires that *every* relying party check the audience — if *any* fails to, an attacker can impersonate any signer. This is not just an abstract concern: in [real cases](https://github.com/dgrijalva/jwt-go/issues/422), OAuth libraries don’t check the `aud` claim, and many APIs for these libraries make it quite easy to ignore this claim.

Further, OIDC tokens might include private data. For instance, the `amr`[ claim](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-amr-values-08) indicates whether the user has multi-factor authentication enabled. Publishing this would indicate to an attacker which accounts might be vulnerable to compromise and not protected by MFA! The `address`[ claim](https://openid.net/specs/openid-connect-core-1_0.html#AddressClaim) contains a physical mailing address for a user.

**The hack is limited to JWT-based authentication methods.** OIDC tokens are in [JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519) (JWT) format. However, you might want to use Fulcio to allow signing with identities authenticated using methods *other than* OIDC! For instance, Fulcio has support for [SPIFFE identities](https://spiffe.io/), which often use [X.509 certificates](https://www.itu.int/rec/T-REC-X.509) instead of JWTs for authentication. Signers *could* sign artifacts directly using the key from the X.509 certificate, but these certificates might be longer-lived than is desired.

**This solution lacks transparency.** Fulcio publishes all issued certificates to a certificate transparency log, but an OIDC identity provider does not. The “Transparency” section below examines the benefits of such logging.

### Demonstration of Proof-of-Possession at the Application Layer (DPoP)

Interestingly, this hack resembles a guerilla version of a [proposed enhancement to OAuth 2.0](https://www.ietf.org/archive/id/draft-ietf-oauth-dpop-12.html) called Demonstrating Proof-of-Possession at the Application Layer (DPoP), which binds a public key to an OIDC access token using the `jkt`[claim](https://www.ietf.org/archive/id/draft-ietf-oauth-dpop-12.html#section-6.1). In fact, the Sigstore community [*has* evaluated](https://docs.google.com/document/d/1xLX6SroHMLltAc55DNlVtwQzKTw5uu6WMq9xL_1GTVA/edit) using DPoPs in the past to replace Fulcio, but decided against it for the above reasons. For instance, the [DPoP proposal](https://www.ietf.org/archive/id/draft-ietf-oauth-dpop-12.html#name-objectives) (§2) notes this: “DPoP is not, however, a substitute for a secure transport and MUST always be used in conjunction with HTTPS” (echoing the claim that “OIDC tokens are supposed to remain private.”)

### Rekor

The preprint also proposes replacing Rekor with an [RFC 3161 Time Stamping Authority](https://www.rfc-editor.org/rfc/rfc3161) (TSA). Rekor provides two functions in Sigstore:

1. It timestamps signatures and related metadata.
2. It publishes signatures and related metadata to a transparency log.

If you decide that transparency isn’t important to you, you can indeed replace it with a TSA. In fact, Sigstore allows users to operate their [own TSA](https://github.com/sigstore/timestamp-authority) that can be used in situations where transparency isn’t appropriate — for instance, when using Sigstore to sign proprietary artifacts. See the “Transparency” section below for more information on the benefits of transparency logs in Sigstore.

But replacing Rekor with a TSA doesn’t eliminate a trusted third party; rather, it simply swaps the trusted third party used. In general, it’s better to *supplement* the functionality of Rekor with *additional* timestamps from a TSA (the typical recommended usage of the Sigstore TSA) than to *eliminate* Rekor in favor of a TSA, unless there are compelling privacy concerns.

### Transparency

Even if the above changes to Sigstore were possible to make securely, the modified system would be missing a critical component: [transparency](https://transparency.dev/). The key idea of transparency is to make all behavior in a system public, and therefore auditable and analyzable. For instance, if an OIDC provider started issuing certificates using your account, you could detect this in the Fulcio certificate transparency log and take appropriate action.

Transparency doesn’t always make sense. By definition, it makes metadata public; for highly-sensitive artifact signing, that’s undesirable (though internal-only audit logs can serve a similar role). In such cases, replacing Rekor with a TSA *can be* reasonable.

The preprint makes an interesting claim about transparency: if there’s no revocation list, transparency isn’t useful. This makes two mistakes: first, that Sigstore doesn’t support revocation. Second, that revocation is the only use of transparency.

**Revocation in Sigstore.** A [recent post on this blog](https://blog.sigstore.dev/signatus-ergo-securus-who-can-sign-what-with-tuf-and-sigstore-ea4d3d84b8b6?source=collection_home---6------3-----------------------) notes that signatures alone don’t tell you whether to trust an artifact; for that, you need a verification policy. This verification policy is a much more natural place to handle revocation than the identity layer; see [Don’t Panic](https://blog.sigstore.dev/dont-panic-a-playbook-for-handling-account-compromise-with-sigstore-ee299dca5144) for an example. This allows us to avoid the scalability problems of global revocation lists (see [CRLite](http://www.cs.umd.edu/~dml/papers/crlite_oakland17.pdf) for a discussion of these issues). The mantra here is [revoke artifacts, not keys](https://blog.sigstore.dev/how-to-sign-a-release-of-oss-e96ee94286fc).

**Other uses of transparency logs.** These logs are [invaluable to researchers](https://dl.acm.org/doi/pdf/10.1145/3278532.3278562), who learn about the rate of certificate issuance, the total count of unique domains; they offer a [useful dataset for testing security technologies](https://jameslarisch.com/pdf/hammurabi.pdf). Certificate transparency in web PKI allows researchers to find [phishing domains](https://www.hardenize.com/blog/certificate-transparency-monitoring-phishing-detection) for blocklisting (the analogous use in the software signing world would be to detect [typosquatting](https://snyk.io/blog/typosquatting-attacks/)). In [some cases](https://groups.google.com/a/chromium.org/g/ct-policy/c/PCkKU357M2Q/), an anomaly in the log results in a log turndown, not a certificate revocation. In general, the proper recourse for an anomalous log entry will vary — an appropriate response could be revocation, but it might also be starting to investigate an incident.

### Minor Issues

In addition to the major changes mentioned above, the preprint makes a number of other suggestions and criticisms.

### Dex

Sigstore runs a service called [Dex](https://github.com/dexidp/dex), effectively as a part of Fulcio. It’s used to smooth over incompatibilities between different OIDC identity providers. The preprint suggests eliminating Dex.

While it’s true that without Fulcio there’s no use for Dex, it’s not true that Dex represents an *additional* trusted third party; rather, it should be considered part of the Fulcio trust domain. Further, Dex is a widely-used, battle-tested piece of software and part of the [Cloud Native Computing Foundation](https://www.cncf.io/).

### Policy documentation

Sigstore does not have a [Certificate Policy](https://datatracker.ietf.org/doc/html/rfc3647) or [Certification Practice Statement](https://www.rfc-editor.org/rfc/rfc3647#page-15). These documents describe how a certificate authority operates. They are common in web PKI and required by the CA/Browser forum. The preprint suggests that without these documents, the Sigstore infrastructure cannot be trustworthy.

However, these documents make much more sense in the context of web PKI than in the context of Sigstore. Many of their constituent sections are irrelevant. Sigstore is a community-run project, and all relevant information is widely available; these policy documents are much more important for closed or for-profit organizations. Further, because the certificate infrastructure in Sigstore is automatic, almost all of the information that would typically be found in these documents is specified in the codebase itself.

In general, the Sigstore project prefers to adopt practices from web PKI selectively, rather than wholesale: web PKI has decades of baggage associated with its operation; in contrast, the artifact signing ecosystem is new and can move much more quickly. If there are specific pieces of information from the Certificate Policy or Certification Practice Statement that users of the Sigstore public good instance should be able to find more easily, we would be amenable to publishing that information in a consolidated manner.

### Revocation of Sigstore certificates

The preprint highlights the lack of a Certificate Revocation List (CRL) for Fulcio’s intermediate certificates. However, we [distribute these via The Update Framework](https://dlorenc.medium.com/using-the-update-framework-in-sigstore-dc393cfe6b52) (TUF) which natively supports revocation. TUF is the [result](https://theupdateframework.com/papers/prevention-rollback-attacks-atc2017.pdf?raw=true) [of](https://theupdateframework.com/papers/protect-community-repositories-nsdi2016.pdf?raw=true) [academic](https://theupdateframework.com/papers/package-management-security-tr08-02.pdf?raw=true) [research](https://theupdateframework.com/papers/attacks-on-package-managers-ccs2008.pdf?raw=true) [around](https://theupdateframework.com/papers/survivable-key-compromise-ccs2010.pdf?raw=true) the pitfalls of key management for secure software artifact delivery. In fact, the TUF-distributed certificates are *safer* than certificates distributed using a traditional CA bundle because TUF protects against [freeze attacks](https://theupdateframework.com/security/) or [rollback attacks](https://theupdateframework.com/security/).

### Conclusion

The arguments presented for replacing the components of Sigstore are somewhat circular. If you have the upstream identity provider perform the role of Fulcio, you can indeed do without Fulcio, eliminating one party you must trust. But that’s only true if you can make the upstream identity provider perform the role of Fulcio in a *secure* way (if you can do it at all). If the goal is to eliminate Fulcio, identity providers should run proper Certificate Authorities of their own, instead of trying to cobble one out of odd corners of OIDC. Similarly, if you don’t want to trust Rekor to timestamp signatures, you can have a TSA timestamp signatures instead (or use both as noted above). But that’s just changing *which* party you’re trusting for timestamps, not *removing* trust! It is true that if you stop providing all security guarantees of Sigstore, you can simplify it. But the transparency component of Sigstore represents an important part of its security guarantees.

Like many technologies, a useful system for signing software with ephemeral keys winds up being a little more complicated than what you might design from the ivory tower. Sigstore’s architecture exists in its current form due to often-subtle real-world considerations. Nobody wants to reduce the amount of trust you need to place in Sigstore more than the folks who develop it — that’s what motivated our prior exploration of DPoPs, for instance — but it turns out that some of this complexity is in fact essential.

*Thanks to Yan-Cheng Chang for this preprint! We’re always interested in proposals for improving Sigstore. Thanks also to Hayden Blauzvern, Santiago Torres-Arias, William Woodruff, Trevor Rosen, Trishank Karthik Kuppusamy, Dan Lorenc, and Cody Soyland for helpful discussion.*