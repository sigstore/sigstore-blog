+++

title = "Privacy in Sigstore"
date = "2022-05-28"
tags = ["sigstore","privacy","softwaresupplychain","supplychainsecurity"]
draft = false
author = "Zachary Newman"
type = "post"

+++

![](/images/privacy.png)

Photo by [Tim Mossholder](https://unsplash.com/@timmossholder?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/data-privacy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

By default, the[ keyless signing flow](https://github.com/sigstore/cosign/blob/main/KEYLESS.md) for Sigstore exposes a user’s email:

```
$ rekor-cli search --email zack@example.com \ # not my real email!
    | wc -l
Found matching entries (listed by UUID):
112
```

Specifically, a user logs in to Fulcio with[ OIDC](https://openid.net/connect/).[ Fulcio issues a short-lived certificate](https://github.com/sigstore/fulcio/blob/main/docs/how-certificate-issuing-works.md) with the[ SAN](https://en.wikipedia.org/wiki/Subject_Alternative_Name) set to your email address as reported by the OIDC identity provider, *even if your email is not typically exposed on that service itself* (for instance, your GitHub email will be exposed, even though it’s not generally public). That certificate goes into Fulcio’s[ transparency log](https://transparency.dev/), which is public.

This raises substantial privacy concerns: many folks don’t want to share their email/identity at all, and even if they’re okay with sharing their email/identity, they might not want it publicly linked to every artifact they sign. Further, even when an individual is comfortable sharing their identity at a moment in time, they may not be in another year (“your name being linked to something isn’t in your personal threat model until it’s too late”).

In this post, we enumerate a few solutions for signer privacy in Sigstore, including things you can do as a user today, along with solutions to look forward to.

### Background and Context

Traditionally, the main use of Sigstore’s keyless signing flow is to link artifact signatures to public identities. Why might you use Sigstore if you didn’t want your identity public?

- *Transparency*: Putting all signatures for, say, releases of numpy on a transparency log can help with incident detection and after-the-fact analysis.
- *Continuity*: Maybe you want to prove that the person who signed package X today is the same as the person who signed it last week, but *without* managing a key pair yourself. In many cases, this is sufficient (for instance, I use plenty of software written by people whom I only know by pseudonym, and only in the context of one specific application).
- *Selective disclosure*: Maybe you don’t want to share your exact identity, but you’re comfortable disclosing certain aspects of it (membership in a list of known maintainers, or the domain of your email). This can be convincing to verifiers.

One challenge is balancing transparency vs. privacy. Traditionally, the benefits of a transparency log come from, well, the *transparency*: making public *everything* that we consider valid (every valid signature, in the case of Sigstore). This means that we can do things like “get notified every time there’s a release of numpy” or “see if user@example.com ever signs something without me knowing about it.” However, this can leak sensitive information (as in[ Certificate Transparency](https://dl.acm.org/doi/abs/10.1145/3338498.3358655)). There’s prior art for protecting the[ privacy of users reporting misbehavior](https://arxiv.org/abs/1703.02209), but limited support for protecting the privacy of the log contents. Hiding the contents of the log can reduce the utility of the transparency feature.

## Current

A privacy-concerned user can do all of the following today.

### Keyed signing

One way to sidestep the problem is to use a traditional, key-based signing flow. Public keys are totally random and opaque, and can be generated per-use case to avoid correlation. Signing multiple artifacts with the same key gives the verifier confidence in the continuity of the signing identity (barring key compromise).

This loses some of the benefits of keyless signing: the signer must manage a signing key, and they must distribute the verification key so that clients know how to verify the artifacts. However, this still gets the advantage of sticking the artifact in a transparency log (e.g., so we can easily see *all* artifacts signed by a given key).

### HashedRekord

Many of the entries in Rekor are[ HashedRekord](https://github.com/sigstore/rekor/blob/main/pkg/types/hashedrekord/v0.0.1/hashedrekord_v0_0_1_schema.json) objects, which contain the hash of the object, a signature, and a public key. Notably, this *doesn’t* include any metadata. So, if the artifacts themselves aren’t public, you can sign them to your heart’s content and all anybody watching the log can learn is that you’re doing a bunch of signing.

There are a few caveats:

- If the artifacts ever become public, they become linkable back to the signer’s identity
- If the artifacts are easy to enumerate (e.g., you know it’s one of 100 artifacts associated with a given software package), someone can enumerate possible values and see if any hashes match. This publicly links the identity of the signer to the signed object, which may not always be desirable. This can be mitigated by[ salting](https://en.wikipedia.org/wiki/Salt_(cryptography)) the hash, though there’s no built-in support for that.
- Metadata is still metadata: if you’re signing when you’re supposed to be asleep, you might get in trouble with your parents. There are other (serious) risks from the metadata too: you’re still exposing your email address, and you might be able to correlate the artifacts with subsequent releases elsewhere on the internet to break the privacy.

### Pseudonyms

Users can register a Google account under a pseudonym, and use that to publish artifacts.

This may violate the terms-of-service of your OIDC identity provider. Also, you have no privacy from the identity provider themselves, who can link you to the artifacts.

### Run your own

If you’re concerned about your email appearing on the public Fulcio/Rekor instances, you can run your own and lock down access to them.

[Sigstore the Hard Way](https://github.com/lukehinds/sigstore-the-hard-way) (may be out of date) is a set of instructions for getting Sigstore running in GCP. If you’re interested in doing this, the Sigstore Slack instance will be very helpful.

Cons:

- *Lots* of work.
- You lose economies of scale in trust (e.g., attention from monitors).
- Extra configuration for clients.

### Near-term

In this section, we explore things that we pretty-much know how to do already, and it’s just a question of whether there’s enough demand for them (given the tradeoffs they make between privacy and convenience) to make it worth implementing.

### SaltedHashedRekords

In the section on “HashedRekord” objects, we noted that there’s no native support for salting the hashes.

We could easily add an optional salt to the HashedRekord spec (that’s *not included* in the HashedRekord object, though we can note that there *is* a salt). Then, to verify, clients would stick the salt at the beginning of the data stream to hash. The salt would be distributed out of band, with the artifact (this may be rather inconvenient to do in a private way). (This just makes explicit exactly what we suggest clients do for salting in the above HashedRekord section.)

### Pairwise Pseudonymous Identifiers

The OIDC specification[ defines](https://openid.net/specs/openid-connect-core-1_0.html#Terminology) a pairwise pseudonymous identifier (PPID) as an

Identifier that identifies the Entity to a Relying Party that cannot be correlated with the Entity’s PPID at another Relying Party.

That is, rather than telling the relying party that “zack@example.com” wants to log in, we say “b5bb9d8014a0” wants to log in (where “b5bb9d8014a0” is an opaque token that remains consistent over time). This provides *continuity* in the sense discussed above.

For example, the[ Firefox OIDC identity provider](https://github.com/mozilla/fxa/blob/main/packages/fxa-auth-server/docs/oauth/pairwise-pseudonymous-identifiers.md) only shares your identity with internal relying parties, and uses PPIDs for every external service. In general, the PPIDs are *pairwise* in the sense that two relying parties will see different PPIDs, even for the same user. The OIDC specification gives a[ pairwise identifier algorithm](https://openid.net/specs/openid-connect-core-1_0.html#PairwiseAlg) demonstrating these concepts (including support for “sectors”, where different relying parties see the *same* PPID for a user).

Caveat:

- Very few OIDC identity providers support PPIDs (as of May 2022).
- Those that do often rotate the PPIDs every few months, so there’s limited support for continuity.

### OIDC Anonymizing Proxy

If OIDC providers don’t tend to support PPIDs, we’re not stuck: we can make them ourselves!

Specifically, we can introduce an “OIDC anonymizing proxy.” This proxy would be both an OIDC relying party (to an “upstream” IdP) and an IdP itself. Its identities would be deterministically computed from the upstream identities, but look opaque: specifically, the identity would be something like

```
H(<name of upstream IdP> || <upstream identity> || salt)
```

where *H* is a collision-resistant hash function (this salt needs to be kept secret for privacy, but remain the same for consistency).

If you’re interested in working on a prototype of this, I’d love to collaborate with you, as long as you’re willing to call it “O-I-Didn’t-C You There”.

### Redaction

Transparency logs and redaction seem to be fundamentally incompatible at first: the whole point of the transparency log is that anybody can validate the entries for themselves, and the logs are built on technologies that use *every* entry to compute *digests* for the logs. However, it’s possible to substitute the *hash* of an entry for the entry itself and correctly compute the digests.

Because redaction is a relatively rare event, with carefully designed policies, we can still get most of the benefits of transparency. For instance, we can have the log stop serving redacted entries to the general public, but serve them to authorized users (or have them available in a backup somewhere). We can extract subsets of the data to redact (e.g., only the identity of the signer or only the hash of the artifact). Monitors can ensure that this behavior is done legitimately (e.g., with a corresponding request from the identity owner, and after a grace period to prevent attackers from using redaction to hide their activities).

Note that this is distinct from revocation, where the entries remain public.

### Long-term

If we’re not limited by things like “engineering resources” and “practical considerations” then there’s a bunch of cool things we can do!

(Perhaps some of these will migrate into the near-term category over time, as they become less speculative!)

### Support other credentials

There are alternatives to (or possibly, generalizations of) OIDC, such as[ decentralized identifiers](https://www.w3.org/TR/did-core/) (DIDs). DIDs support different “DID methods,” or forms of authentication. One example is[ did:key](https://w3c-ccg.github.io/did-method-key/), which specifies a public key directly; another can[ bridge from OIDC](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html). But the advantage of DIDs is that users can choose a DID method that meets their privacy needs, like[ did:ethr](https://github.com/decentralized-identity/ethr-did-resolver) (which specifies an Ethereum blockchain address).

### Selective disclosure

Some forms of credentials support “[selective disclosure](https://www.w3.org/TR/vc-imp-guide/#selective-disclosure),” where users are issued encrypted or otherwise encoded identity tokens and can choose subsets of the information in those tokens to disclose.

For example, my identity token might say that my email is zack@example.com, that I’m in the US, and that my account has 2FA turned on. I can take those tokens and prove to relying parties (e.g., Fulcio) that it’s a valid token for an account with 2FA, but *nothing else*.

Another advantage here: the tokens on their own are much less valuable without full disclosure; with variations of this scheme, we can do things like store a log of OIDC tokens as proof that Fulcio certificates were issued honestly (without risking the tokens leaking!).

### Redaction transparency

The discussion of redaction above focused primarily on policy for redaction: when is it allowed, and who should be able to access redacted data.

We may also need technical support for implementation redaction without putting a compromised log in a position to hide nefarious activity or equivocate to users.

[Revocation transparency](https://www.links.org/files/RevocationTransparency.pdf) proposes adding an additional feature to a transparency log that allows users to check “this entry is in the log, AND it has *not* been revoked.” Similar techniques might apply to redaction.

### “More research is needed”

There are further possibilities, and I’d love to explore them with you! I have a number of half-baked ideas. For instance, think of “selective disclosure” but where the attribute is not “my identity has this domain” but rather “my identity is authorized to publish the package ‘numpy’”.

I believe that privacy-respecting solutions using Sigstore can be constructed, and if you have a scenario in mind that this doesn’t address, get in touch via the [Sigstore Slack or our community meetings](https://www.sigstore.dev/community).