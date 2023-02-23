+++
title = "Signatus, ergo securus? Who can sign what with TUF and Sigstore"
date = "2022-12-13"
tags = ["sigstore","softwaresupplychain","identity","tuf","intoto"]
draft = false
author = "Zachary Newman, Marina Moore"
type = "post"
+++

![](/images/signatus1.jpg)

Photo by [Brett Jordan](https://unsplash.com/@brett_jordan?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

[Sigstore](https://sigstore.dev/) is an open-source project and service run by the [OpenSSF](https://openssf.org/) to make signing software easy! Before Sigstore, a developer who wanted to sign software needed to manage a GPG key. With Sigstore, they can use their *identity* (for instance, a Gmail account) to sign. It also brings *transparency*: actions must be posted on a public log, so they can be audited to detect bad behavior and to analyze damage after-the-fact.

It’s easy to get caught up in the hype and think that because more developers are signing software, the consumers of that software are necessarily more secure. However, a signature is only useful if consumers verify it *correctly*. One common failure mode is to [verify *that* some software was signed, but not check *who* signed it](https://caremad.io/posts/2013/07/packaging-signing-not-holy-grail/). This means that you’ll treat a signature from evil@hacker.com the same as a signature from yourself! Fortunately, Sigstore is in the process of [making that hard to do by accident](https://github.com/sigstore/cosign/issues/2056).

It’s all well and good to decide we want to check that software came from the right person, but how do we know who that is? In some sense, that’s not really Sigstore’s job — the answer will be different when you’re downloading something from a package repository compared with downloading a container image produced internally by your organization. There are other projects — most notably, [The Update Framework](https://theupdateframework.com/) (TUF) — which aim to solve exactly that problem. In this post, we’ll see how you can figure out the answer to that question — securely — in order to verify Sigstore signatures, and see how Sigstore can concretely improve the security of open source package repositories, internal container registries, and everything in between:

- **Verification policies** are the way you can tell which signatures are trustworthy and which aren’t.
- **TUF and in-toto work with Sigstore** to provide verification policies in a variety of settings.
- Ongoing work is **securing open source repositories** by adding verification policies.
- Each organization has **unique verification needs**; **Sigstore can help** express many of them.

### Why sign software?

Before we start signing packages (this post focuses on package signing, but container signatures are similar), we should make sure that signing packages will actually help with the problems that we have!

What does it mean when a package is signed? I could be flippant and say: it means that the package is signed. This actually isn’t as unhelpful of an answer as it first seems. Signatures don’t, on their own, have an unambiguous interpretation. For instance, what does it mean if I sign a compiled binary? I certainly didn’t hand-write every byte. Am I vouching for the source of the binary? The build process? Both?

It’s *verification policies* that imbue signatures with semantics. If I’m the only developer at my organization, I can write a policy that only installs software on production machines that I’ve personally signed. Then, we can interpret a signature to mean “I believe that this binary is ‘good.’” You might want to check more subtle things, like “developer X says that this Git repo at checkout Y is good AND my build server says that it built binary Z from the source at checkout Y.” The [in-toto](https://in-toto.io/) project excels at expressing these sorts of requirements.

Signatures help us check that a specific party is making a specific claim about particular data, and we have the flexibility to decide which claims we require. This makes them really useful for checking that a trusted party vouches for the artifact (authenticity), and that the artifact hasn’t been tampered with (integrity). For instance, signing would prevent *a hacker* from compromising my account on npm and then uploading a malicious binary as the latest release of a package I control. Done right, it can even prevent *npm itself* from doing this (even if [a vulnerability](https://github.blog/2021-11-15-githubs-commitment-to-npm-ecosystem-security/) lets a hacker upload a release for that package). Another use is making sure that published artifacts were built from a specific source (as in [this proposal for npm](https://github.blog/2022-08-08-new-request-for-comments-on-improving-npm-security-with-sigstore-is-now-open/)).

So what *can’t* signing do? Well, if somebody that I trust suddenly starts publishing malware (perhaps [under duress](https://xkcd.com/538/) or [after being bought](https://www.bleepingcomputer.com/news/security/the-great-suspender-chrome-extensions-fall-from-grace/)), signing won’t help with that. Similarly, if a package itself has a vulnerability, signatures don’t fix it. And if I accidentally download a package from somebody I *don’t* trust (for instance, a [typosquatting](https://www.darkreading.com/vulnerabilities-threats/beware-the-package-typosquatting-supply-chain-attack) package), I’ll accept their signatures.

**Conclusion:** if we know who owns a software project, signing means nobody else can impersonate them to publish a release; however, it doesn’t mean that the signed software is *good* or *does what we think it does*.

### Software verification policies

With the benefits and limitations of software signing in mind, how should we set up a system to figure out who’s allowed to sign?

The answer is: there’s no *single* system that can do that. My weekend project might have different security standards from a high-security company (which might hand-audit every line of code they import, and only trust their security team). I might trust a package when the Debian maintainers sign it as they repackage it; you might want to get a signature straight from the upstream.

Despite this open-ended answer, the problem is somewhat tractable due to *context*. Most of the software I install comes from *repositories*, which includes tightly-run operating system repositories, community-operated package repositories, and container registries. If I `pip install` a package, I can have [PyPI](https://pypi.org/) tell me who’s in charge of it (and, if PyPI configures [TUF](https://theupdateframework.com/) to prevent this, a hacker on the live PyPI server won’t be able to convince me otherwise). Because of the context, users don’t need to choose a verification policy: we can build these policies into existing package managers for automatic security. Power users can augment these with policies of their own.

Here’s how this might look in various settings.

### Open source repository, limited set of maintainers

Many operating system package repositories (for instance, the apt repositories for Debian or Ubuntu) rely on a trusted team of package maintainers. These maintainers typically use [GPG keys](https://wiki.debian.org/SecureApt) to sign their software. This works well, for the most part: these maintainers can manually verify changes from upstream before pulling them in and generally don’t go rogue. However, there are a few issues with the way some repositories implement this signing and verification procedure:

- **All keys are equally valuable:** a maintainer who typically packages fun things like [cowsay](https://en.wikipedia.org/wiki/Cowsay) also has the ability to sign a release of [openssl](https://www.openssl.org/) or other critical software.
- **Recovery:** if a key leaks, you can’t revoke it and it can still be used to publish packages. What happens if a maintainer moves on? Quits? Is fired?
- **Combining packages from multiple repositories gives them equal power:** I may [add a repository](https://manpages.ubuntu.com/manpages/trusty/man1/add-apt-repository.1.html) to install a single software package; I don’t want it to be able to serve me *other* packages! (There are some mitigations in apt for this, like repository ordering, but they don’t totally solve the issue.)

Such repositories could run [TUF](https://theupdateframework.com/), which was designed for exactly this scenario. It contains features to help with all of these issues: delegations, which allow giving a particular maintainer the power to sign some specific packages, key rotation and revocation, which can handle leaked keys, secure [repository combining](https://github.com/theupdateframework/taps/blob/465bcfed2e74855afdb74509ab18f78e49d1ce30/tap4.md), thresholds of signatures (we can require some packages be signed by multiple people). The TUF instance for a repository would be managed by the same organization, but it moves the most critical keys offline for security, and prevents a number of other subtle attacks (e.g., serving an old package version, which might have a valid signature, but has since been replaced due to vulnerabilities).

There are a number of repositories [using TUF in practice](https://theupdateframework.com/adoptions/). However, even with TUF, there’s a remaining issue:

- **Attackers can be sneaky:** if I compromised a maintainer’s key, I could go for years without detection if I served most users legitimate builds, but sent backdoored ones to specific targets.

By adding Sigstore, we get auditability (because package signatures must be published in a publicly-auditable place)!

### Open source repository, community maintainers

Many language-specific package repositories allow anybody to sign up for an account and publish a package. This is great for increasing the number of high-quality libraries available, but a little harder to manage for a few reasons:

- **Maintainers must manually manage keys:** non-professional maintainers ([and even many professionals](https://words.filippo.io/giving-up-on-long-term-pgp/)) don’t want to manage cryptographic keys. They risk losing them or leaking them. Even if they don’t, signing adds extra workflow steps (you can only deploy from specific machines, or you have to dig a USB security key out of a closet). This is one reason why adoption of package signing is often low on repositories that support it.
- **Account recovery:** community users expect to be able to recover their accounts. This is easy if packages aren’t signed, but if a package repository can rotate your keys unchecked, it could also start signing on your behalf if compromised!
- **No identities for build machines:** community users may want to use build systems (like GitHub Actions) to actually compile their releases. What key should we use for GitHub actions, and how do we know that it was building the right package?

Our solution from the previous scenario uses TUF and traditional cryptographic key pairs in combination with Sigstore for transparency. We can extend it to use [a proposed enhancement to TUF](https://github.com/theupdateframework/taps/pull/141) that allows delegating to *identities*, rather than keys. This means that developers can use their Gmail account to sign packages, and lean on Google for account recovery (which they have exhaustive procedures for, especially compared to a volunteer-run package repository). Further, Sigstore supports [workload identity](https://dlorenc.medium.com/a-bit-of-ambiance-comes-to-sigstore-f80d1d6b1c30), so we can have a GitHub Actions job sign a statement that it built a particular binary!

Excitingly, real package repositories are interested in such a solution:

- PyPI has ongoing work to implement TUF ([PEP 458](https://peps.python.org/pep-0458/)) and TUF delegations ([PEP 480](https://peps.python.org/pep-0480/)), and is [using Sigstore to sign CPython releases](https://www.python.org/download/sigstore/); in the long run, we hope these efforts converge.
- RubyGems has a [similar proposal](https://github.com/rubygems/rfcs/pull/37).
- npm has [proposed](https://github.blog/2022-08-08-new-request-for-comments-on-improving-npm-security-with-sigstore-is-now-open/) verifying that all packages are built on trusted build machines from publicly available sources.

In the long run, such repositories could require both author signatures *and* build machine signatures, specified via [in-toto](https://in-toto.io/) layouts which would be distributed by TUF.

Package signing is certainly not sufficient to solve all risks to package repository security. It doesn’t help if you *request* a malicious package (for instance, because you mistyped the name of the one you actually wanted) or if a maintainer goes rogue. But it does help with certain kinds of compromises. To learn more, check out the [Securing Software Repositories working group](https://github.com/ossf/wg-securing-software-repos) at the OpenSSF.

### Autoupdaters

If you download a browser directly from a website (rather than via a package manager), it will often autoupdate by default. Other client software often does the same. A compromise in the package repository [can cause users to download malware](https://www.trendmicro.com/en_us/research/22/h/irontiger-compromises-chat-app-Mimi-targets-windows-mac-linux-users.html); package signing can help!

For such applications an initial download is still somewhat high-risk: if the update server is compromised, an attacker can give you a bad copy of the application! But thereafter, a good autoupdater can implement TUF with Sigstore support (similar to our “limited set of maintainers” repository above). And, developers could use Sigstore identities to sign to make their lives easier, like in the community setting.

### Internal to an organization

Each organization has its own needs, so the possibilities are endless here. You can use something like the Sigstore [policy-controller](https://github.com/sigstore/policy-controller) to require that software you run is signed by your security team, that it has an attached [SCA scan](https://www.synopsys.com/glossary/what-is-software-composition-analysis.html), or to allowlist specific external developers. Something like TUF may or may not make sense. Reach out on the [Sigstore Slack](https://links.sigstore.dev/slack-invite) if you’d like advice and discussion!

### Other software

Above, we mentioned that what makes verification policies achievable to create is that they aren’t universal: each ecosystem can manage their own. But what do we do about the long tail of software that *isn’t* installed from package repositories? Is there any respite from the curl | sudo bash installation instructions?

The path here is much less clear, but Future Research™ will hopefully provide an answer. Work on [Sigstore Trust Delgations](https://docs.google.com/document/d/1lNW2lNu9QmDhCgXJaNTYxBUzCjuWS2wj8fCHSadbzxQ/edit#heading=h.nnzefxoi1ri) will help extend the reach of the Sigstore trust root. There’s also an exciting [early-stage design](https://docs.google.com/document/d/1WPOXLMV1ASQryTRZJbdg3wWRR4ckK558kUnxn7Eixaw/edit) (join [sigstore-dev@googlegroups.com](https://groups.google.com/g/sigstore-dev) to access) for “transparent and federated TUF.” This combines Sigstore and TUF even more intimately. It allows flexible policies to hang off of a central trust root, from global [trust-on-first-use](https://en.wikipedia.org/wiki/Trust_on_first_use) to custom per-domain policies.

Just like Sigstore currently makes signing easy and something developers don’t have to think about, the Sigstore of the future will make *verification policy* something developers don’t have to think about. Hopefully a future blog post will share all the details!

### Conclusions

Sigstore is still an early-stage project, and it has many promising future directions to secure open source (and proprietary) software, especially in concert with excellent projects like TUF and in-toto. And in contrast with GPG signing, which has seen little traction over nearly three decades, the usability benefits mean that there’s a viable path to widespread adoption, which makes it infinitely more useful than a signing scheme nobody uses.

*This post owes a great debt to Alex Floyd Marshall, who wrote* [*many thought provoking questions*](https://medium.com/@alexfloydmarshall/kubecon-na-2022-day-3-who-you-trust-matters-4d23a191580a) *about this issue, the OpenSSF Securing Software Repositories working group, which is full of hard-working repository maintainers making open source more secure, and Justin Cappos, Santiago Torres-Arias, and William Woodruff, with whom I’ve had many enlightening discussions on these topics.*
