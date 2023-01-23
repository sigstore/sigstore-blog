+++

title = "Is Sigstore Ready for a Post-Quantum World?"
date = "2022-07-17"
tags = ["sigstore","nist","Post-Quantum Cryptography"]
draft = false
author = "Zachary Newman"
type = "post"

+++

![](/images/quantum.jpg)

Photo by [Anton Maksimov 5642.su](https://unsplash.com/@juvnsky?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

A couple of weeks back, NIST made big news in the cryptographic community by [announcing](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms) that they have selected four quantum-resistant encryption and digital signature algorithms for standardization. In recent years, worries about the threats that quantum computers pose to current encryption algorithms have precipitated a major effort to establish a “post-quantum” (PQ) cryptographic toolkit. NIST’s 99-page [full report](https://nvlpubs.nist.gov/nistpubs/ir/2022/NIST.IR.8413.pdf), which reflects six years of work by a group of expert cryptographers details the algorithms and their performance and security characteristics However, the report omits the answer to the question on every Sigstore user’s mind: is Sigstore ready for a post-quantum world?

Worry not — here, we cover what “post-quantum” means and why it matters before diving into an analysis of all the ways in which Sigstore uses cryptography and which of those ways are or are not quantum resistant. We’ll see that there’s a long way to go before the internet is ready for a post-quantum world, and that many internet building blocks need to be upgraded before Sigstore itself can resist an attack by a quantum computer. Nevertheless, we can glimpse the path to a post-quantum Sigstore, and see what we should do *immediately* and what we should do *eventually* to get us there.

### What does post-quantum mean, anyway?

To answer this question, it helps to know what makes a system “quantum-resistant” or “post-quantum” in the first place, so we’ll start there.

Cryptography secures the digital world, including web browsing, sending email, and keeping medical records private. Much of cryptography, especially “asymmetric cryptography,” is based on mathematical objects with convenient structures. The objects most commonly used today include [Sophie Germain primes](https://en.wikipedia.org/wiki/Safe_and_Sophie_Germain_primes), [elliptic curves](https://en.wikipedia.org/wiki/Elliptic_curve), and [semi-prime numbers](https://en.wikipedia.org/wiki/RSA_(cryptosystem)). Often, there are operations with a “trapdoor:” they’re hard to compute, *but* if you know some secret value, they become easy (the most famous example is factoring: it’s very fast to multiply two large prime numbers, but impractically slow to factor their product into the component primes).

However, with a sufficiently large quantum computer, an attacker could quickly defeat cryptosystems based on the above techniques using [Shor’s algorithm](https://en.wikipedia.org/wiki/Shor's_algorithm). This could be catastrophic, exposing private information and allowing malicious adversaries to install malware on computers. Fortunately, many of the algorithms we use today, mostly “symmetric” cryptography, aren’t vulnerable to these attacks. These are “post-quantum” or “quantum-resistant” algorithms, which simply means that we don’t *currently* know how to break them with a quantum computer.

The NIST post-quantum cryptography project aims to find replacements for in-use algorithms that are *not* quantum-resistant. Indeed, cryptographers have risen to the challenge and presented candidates for replacement asymmetric [algorithms based on error-correcting codes, lattices, supersingular isogenies, hashes, and systems of multivariate equations](https://en.wikipedia.org/wiki/Post-quantum_cryptography#Algorithms). NIST has chosen a lattice-based algorithm called CRYSTALS-Kyber for encryption, and lattice-based algorithms CRYSTALS-Dilithium and FALCON, along with a hash-based algorithm SPHINCS+, for digital signatures.

Given all this fuss, you might think that we’re in danger of societal collapse due to imminent quantum computers — but you’d be wrong. There *are* quantum computers, and they may even [outperform classical computers on specific tasks](https://www.nature.com/articles/s41586-019-1666-5) (though this [claim is disputed](https://arxiv.org/abs/1910.09534)). But these computers aren’t powerful enough to aid in cryptanalysis. The size of a quantum computer is measured in [qubits](https://en.wikipedia.org/wiki/Qubit), and despite the heroic efforts of teams of physicists and engineers, the [largest quantum computers constructed to date](https://en.wikipedia.org/wiki/Timeline_of_quantum_computing_and_communication#2019) have well under 100 qubits; to tackle meaningfully large factorization problems, [millions could be required](https://arxiv.org/pdf/1512.00796v1.pdf). Overall, many professionals conclude that [quantum cryptanalysis is more bark than bite](https://www.lawfareblog.com/quantum-cryptanalysis-hype-and-reality).

The best known quantum attacks on SHA2, AES, and ChaCha20 use [Grover’s algorithm](https://www.rfc-editor.org/rfc/rfc9162.html#name-signed-tree-head-sth) or the [BHT algorithm](https://arxiv.org/abs/quant-ph/9705002), which gives a quadratic improvement to brute-force search. However, even with this quadratic improvement, the attack is still slower than the best-known classical attacks. See [further reading](https://crypto.stackexchange.com/questions/59375/are-hash-functions-strong-against-quantum-cryptanalysis-and-or-independent-enoug) on [AES](https://eprint.iacr.org/2019/272.pdf), [SHA2](https://eprint.iacr.org/2020/213), and [ChaCha20](https://crypto.stackexchange.com/questions/70492/how-resistant-are-stream-ciphers-like-salsa20-or-chacha-in-a-post-quantum-world)/[Poly1305](https://www.cryptrec.go.jp/exreport/cryptrec-ex-2601-2016.pdf).

That is to say, NIST is spending this much time and energy on post-quantum cryptography as a preventative measure. This is smart: it takes years or decades to roll out new technologies, as we’ve seen with [HTTPS](https://transparencyreport.google.com/https?hl=en), [TLS 1.3](https://arxiv.org/pdf/1907.12762), and [Python 3](https://stackoverflow.blog/2019/11/14/why-is-the-migration-to-python-3-taking-so-long/). Also, scientific progress is unpredictable: if quantum computers experience an exponential takeoff just like classical computers did, quantum cryptanalysis could be here before we know it.

### Sigstore

In what ways does Sigstore use cryptography? These are the components which might have to be upgraded to make Sigstore quantum-resistant (cryptography **bolded**):

1. Sigstore, well, stores signatures. Specifically, it stores digital signatures. Users create **asymmetric key pairs** and use those to **sign artifacts**, then optionally store these signatures alongside the artifact being signed (for instance, in a container registry). Other users **verify those signatures**.
2. In [“keyless”](https://github.com/sigstore/cosign/blob/main/KEYLESS.md) mode, users don’t manage their own keys. Instead, they generate a **short-lived key pair for digital signing**. They use [OIDC](https://openid.net/connect/) to log in with an identity provider (for instance, using their Google account), which gives them an identity token that’s **digitally signed**.
3. Fulcio **verifies the digital signature** on this token, then issues a certificate to the user (**containing the ephemeral key**, and **digitally signed by Fulcio**).
4. The user **signs the artifact**, then publishes an entry to Rekor, and throws the ephemeral private key away.
5. Rekor is an [append-only log](https://transparency.dev/), which means that you don’t have to trust that Rekor doesn’t change entries — you can verify that fact. Rekor’s log uses [**Merkle trees**](https://en.wikipedia.org/wiki/Merkle_tree), a cryptographic construction based on hash functions. Rekor also issues and**digitally signs** [signed entry timestamps](https://certificate.transparency.dev/howctworks/#logs-return-scts-to-the-ca) and [signed tree heads](https://www.rfc-editor.org/rfc/rfc9162.html#name-signed-tree-head-sth).
6. Clients get the root certificates for Fulcio and Rekor, which are used to **verify certificates and** [**signed certificate timestamps**](https://certificate.transparency.dev/howctworks/), via [TUF](https://theupdateframework.com/), which uses **digital signatures** under the hood.
7. All communication between all parties is **encrypted and authenticated** using TLS.

Looking at this, it turns out that Sigstore *isn’t* quantum-resistant: almost all of the signatures in the Sigstore ecosystem, both by users and by Rekor and Fulcio are using ECDSA, an elliptic curve signature algorithm. However, the append-only log *is* quantum-resistant, as hash functions are not definitively broken by quantum computers.

Then, to make Sigstore “post-quantum,” the main task is to replace its use of ECDSA signatures with a quantum-resistant signature algorithm. Fortunately, there’s a [go-dilithium](https://github.com/dis2/go-dilithium) library implementing the CRYSTALS-dilithium signature algorithm. The work of porting Sigstore to use it involves replacing every reference to crypto/ecdsa with go-dilithium. There are hundreds of these references, but the work is fairly mechanical and it should be possible to pull together a proof-of-concept fork in a matter of days. However, because Sigstore has real users, changing over all at once would be a disaster — old clients couldn’t verify new signatures, and vice versa. A proper rollout would take lots of time, including opt-in and opt-out phases, along with thoughtful support for backwards compatibility that didn’t open the door to “downgrade attacks” to vulnerable algorithms.

Even after this rollout, we’re not done: there are a few cryptographic components in this protocol that are outside of the control of Sigstore. OIDC uses Java Web Tokens (JWTs), which currently [don’t support any post-quantum asymmetric signature algorithms](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1), for its tokens. Similarly, TLS 1.3 supports [a number of signature algorithms](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2.3), but none are quantum-resistant; neither are [the key exchange algorithms](https://datatracker.ietf.org/doc/html/rfc8446#section-2). Still, we should expect that, after the conclusion of NIST’s standardization process over a few years, these protocols will be extended to support quantum-resistant algorithms, and using these will be a simple matter of configuration.

The following table summarizes each component, its use of cryptography, whether the component is quantum-resistant, and the post-standardization plan:

```csv
Component,Cryptography,Quantum Resistant,Plan (after standardization)
Signing/verifying artifacts (short/long-lived keys),Digital signatures (currently ECDSA),❌,Replace with PQ signature algorithms.
Rekor append-only log,Merkle tree (currently SHA2 hash),✔️,
"Fulcio/Rekor signatures on certificates, SETs, SCTs, and STHs",Digital signatures (currently ECDSA),❌,Replace with PQ signature algorithms.
TUF: distributing Sigstore root certificates,Digital signatures (currently ECDSA),❌,Add PQ signature algorithms to TUF implementation.
OIDC tokens for identity,Digital signatures (currently RSA),❌,OpenID Foundation should add (and require) PQ algorithms in OIDC.
TLS: secure point-to-point communication,"Key exchange (ECDHE, DHE)",❌,IETF should update TLS to support PQ key exchange and signature algorithms.
,"Signatures (RSA, ECDSA, EdDSA)",❌,
,"Encryption (AES, ChaCha20/Poly1305)",✔️,
```

### Conclusion

While I personally hope to be using Sigstore to secure my binary artifacts and container images well into the potentially quantum future, it’s probably too early to go all-in on these new algorithms. The standardization process might require incompatible changes, or we might learn more about the ideal use cases. These algorithms are slow in comparison with their classical predecessors (especially since there won’t be widely-available hardware acceleration until after standardization), and early implementations may be riddled with bugs and vulnerabilities. Worse — even if Sigstore does everything right here, without updates to OIDC and TLS, users will still be vulnerable.

As the post-quantum ecosystem matures, adoption will become easier and safer. And quantum threats to encryption are hopefully well into the future. Sigstore should (1) plan for an eventual migration to use PQ algorithms, (2) reduce lock-in to any one signature algorithm, and (3) watch the maturity of implementations and relevant protocols. Still, it’s prudent to emulate NIST in this domain — a little planning and preparation can mean that even if the world changes quickly around us, Sigstore should remain secure against even the most motivated and powerful adversaries for years to come.