+++
title = "A New Kind of Trust Root"
date = "2021-06-08"
tags = ["sigstore","infosec","security","kubernetes"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

I’m thrilled to announce that the [Sigstore](http://sigstore.dev/) community is holding our first Root Key ceremony on June 18th at 2pm Eastern, and I’m even more thrilled to announce that it will be hosted LIVE by the always incredible [DanPOP](https://twitter.com/danpopnyc) on his CloudNative.tv show, [Spotlight Live](https://www.twitch.tv/cloudnativefdn). This Trust Root will eventually be used to secure the keys used by the entire Sigstore project, but more importantly we’re planning to make our **trust root available for any open source project that wants to use it**!

![](/images/trust1.png)

This is a huge milestone for the Sigstore project, and we also hope that the trust root we’re establishing will play a critical role in open source supply chain security going forward! In typical Sigstore fashion, we’re planning to do this a little differently. This means we have some fun surprises planned, and have designed the entire ceremony around audience participation! We’re relying on the community to help us establish this shared Trust Root for years to come.

If you’re not sure what a Root Key ceremony is, or why this matters, keep reading! If you have heard of Root Key ceremonies and are about to switch tabs due to the wave of dry boredom you’re likely imagining, please keep reading too!

### What Is a Trust Root

[Public-Key Infrastructure ](https://en.wikipedia.org/wiki/Public-key_cryptography)is a critical component of internet infrastructure and security, but it’s easy to overlook every day. This is the core system that allows two parties to authenticate the identities of each other across an untrusted network. When you type “google.com” into your browser, PKI is how your browser first establishes that the server on the other end of the connection is actually operated by google.com, and not a malicious attacker.

PKI is full of a lot of confusing terminology (see [this excellent explainer](https://smallstep.com/blog/everything-pki/) from [Mike Malone](https://twitter.com/mjmalone?lang=en) if you’re interested in the details), but the most important thing to know is that it’s not magic. To verify the identity of a system on the internet, you ask that system to present you with credentials. You then verify those credentials against something you trust. If everything checks out, you’re good!

The problem with these systems is that you need to establish trust ***to something*** to bootstrap everything else. How do you decide what to trust at first? This is a philosophical question as much as it is a technical one. In Web PKI (the infrastructure behind browser security), these are called [Root Certificate Authorities](https://en.wikipedia.org/wiki/Certificate_authority), (or CAs), and they’re built into browsers and operating systems. These Root CAs go through rigorous audits to prove they operate securely, and apply to be accepted into the trust roots that are bundled by these vendors. When you visit a website, your browser automatically checks that the website has an identity that can be chained back to one of these Root CAs — this is how your browser knows who it is talking to.

### Why Does Sigstore Need a Trust Root?

The goal of sigstore is to improve the supply chain security of open source software by making software signing ubiquitous, free, easy, and transparent. We’re doing this because software signing can make an entire class of supply-chain attacks harder by providing a cryptographically-verifiable chain of custody between an end user and the author or publisher of an artifact. Just like Web PKI, we need a root of trust to do this. Unfortunately the existing Web PKI system can’t be directly used for signing software artifacts, and other similar Code Signing PKIs are costly and difficult for open source projects that often have no formal legal identity.

The Sigstore Trust Root will allow individuals and systems to automatically retrieve digital certificates that prove who they are, and then use these certificates to sign the artifacts they distribute. End users can then verify the signatures and certificates against the Sigstore trust root, allowing them to verify (and trust) that the distributors of the software they use are who they say they are.

This means we’ll be doing the hard (but fun) work to establish this initial Trust Root. We’ll work and get it distributed, and eventually trusted as widely as possible. **Trust needs to be built up over time, so we’re getting started early.**

### OK, How Will This Work?

While Key Signing parties are common, they rarely happen in distributed, open source communities. We’ve carefully designed ours after carefully reviewing all of the others we could find, and of course applying some of the Sigstore principles: transparency, openness, resilience, and most importantly fun!

Here’s the rough plan (if you’re interested in more details, [see the original design):](https://docs.google.com/document/d/1XxphXkZa9MxbiHnXpstwWn99Dsa5mPFTljpHtK-iJwM/edit?usp=drivesdk&resourcekey=0-Zaqmaw0ZATVU_nda3KHgyQ)

- We’ve selected five community members to act as “key holders” based on their expertise and experience within the Sigstore community, from a diverse range of backgrounds, industries and companies, and geographic regions.
- These key holders will be joined on a live stream by the event hosts as they create brand new cryptographic keys on physical, hardware tokens.
- These keys will be published in a new GitHub repository, where attendees will be able to verify them live as they are merged via pull requests.
- All five keyholders will then sign an initial TUF ([The Update Framework](https://theupdateframework.io/)) Root Metadata file, establishing the project’s initial policies and delegations.
- Attendees will then verify the signatures on this initial rust root, and help us distribute it as widely as possible! Even just clicking “Fork” helps, but we’re also hoping for people to get creative. Tweeting the `sha256`, publishing on blockchains and more are all helpful! **There will be prizes!**
- Going forward, a majority of these keys will be required to make any changes to policy or authorize new subordinate keys.

We’re proud to announce that the initial Key Holders are: [Luke Hinds](https://twitter.com/decodebytes) and [Bob Callaway](https://github.com/bobcallaway) from RedHat, [Marina Moore](https://github.com/mnm678) from New York University, [Santiago Torres-Arias](http://twitter.com/torresariass) from Purdue University, and myself (Dan Lorenc) from Google.

### What Next?

After we establish the initial trust root, we’ll use it to protect the root keys used by our [Timestamping Service and Transparency Log](https://github.com/sigstore/rekor) (Rekor), as well as our f[ree Code Signing CA](https://github.com/sigstore/fulcio) (Fulcio). We’ll also be using this trust root to protect the binaries we distribute for all of our client-side tooling. Establishing a trust root is important but also tedious, so we’re also **planning to open up the Sigstore Trust Root to any open source project or community that wants to use it**! If you’re interested in taking advantage of this, please reach [out to us](https://groups.google.com/g/sigstore-dev) as we design the protocols and tooling!

We’ll also be formalizing our continuous verification and rotation program. We’ll start with live meetings **every four months,** where the Key Holders will publish an updated Trust Root that chains back to this initial one. Frequent updates will allow us to prevent/detect device loss, update hardware in the event of cryptographic advances, and most importantly, **to rotate in new community members!** Being a Key Holder is a large responsibility, so we’re planning to rotate Key Holders out continuously, giving them an average term of ~1.5 years. We’ll be announcing some policies for nominating the next Key Holder soon!

### Getting Involved

Watch live on the day of on [Cloudnative.tv](https://www.twitch.tv/cloudnativefdn)! Just watching the event helps! If you want to help live, follow me on [Twitter](http://twitter.com/lorenc_dan) for more instructions on how to independently verify each step live as we go. Join [our slack channel and email list to stay updated](https://github.com/sigstore/community#slack)! If you want to hear more details, I’ll also be discussing this on the [Popcast](https://twitter.com/PopcastPop) on June 16th.

Special Thanks to [Trishank Karthik Kuppusamy](https://twitter.com/trishankkarthik), [William Woodruff](https://twitter.com/8x5clPW2), for their advice, and to [Asra Ali](https://twitter.com/AsraEntr0py) for her design and implementation of the code and process!