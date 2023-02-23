+++
title = "How to become the next Sigstore Evangelist?"
date = "2022-12-20"
tags = ["sigstore","cosign","sigstorecon","evangelist","bestsigstoreevangelist"]
draft = false
author = "developer-guy"
type = "post"

+++

![](/images/evangelist.png)

### How to become the next Sigstore Evangelist?

My story began by seeking a solution for signing container images at my company Trendyol. You will appreciate that few solutions were available at that time, almost two years ago. The only solution was [DCT (Docker Content Trust)](https://docs.docker.com/engine/security/trust/) based on a [Notary](https://github.com/theupdateframework/notary) project, which is an implementation of the [TUF (The Update Framework) specification](https://github.com/theupdateframework/), which allows you to verify both the integrity of the image and the publisher of all the data received from a registry by creating and using digital signatures. But, one of the downsides and challenging parts of using and spinning up a Notary server was that it required another database to store signatures. In addition, it was a solution supported by only a Docker runtime; no other runtimes were keeping it.

One day, I was hanging out on GitHub’s timeline and saw the [Cosign](https://github.com/sigstore/cosign) project because it was given a star by one of the people I follow. The description explained it was a tool for signing and verifying container images. I was shocked that it used the OCI (Open Container Initiative) registry to store the signatures; no additional databases were required. As a person who struggled with the difficulties of using Notary, the use of the OCI registry was an excellent idea because we can see that the Notary v2 project is also designed to use the same approach and provides for multiple signatures of an OCI Artifact (including container images) to be persisted in an OCI conformant registry. This is how my journey of being a Best Sigstore Evangelist began.

Since the Cosign project was entirely new and in the early stages, there were many contribution opportunities (and there are still opportunities to contribute today)! I took this chance and became one of the first project contributors. Day by day, my interest in the Cosign project increased, indirectly increasing my knowledge of this subject. With this confidence, I tried to use this knowledge in other projects and set sail ⛵️ for an endless adventure. When the Cosign project was first announced to the public, it gained popularity quickly. This made me believe how hungry the sector was for this solution and showed me that I could contribute significantly to the industry.

### Breaking Down “Evangelism”

Let’s explain the term “**Evangelist**” because this might be essential for the people thinking of becoming the next Evangelist. To be honest, I was also unfamiliar with that term until I saw the explanation on the [**Sigstore Community Awards**](https://github.com/sigstore/community/tree/main/awards#sigstore-community-awards) page. **If you visit the page, you can review the categories of their award program:**

- **Most Valuable Contributo**r — for the individual who has made a significant impact on the project this year (winner [Asra Ali](https://twitter.com/AsraEntr0py))
- **Best Evangelist** — for the individual who has gone above and beyond to spread the word about Sigstore (winner [Batuhan Apaydın](https://twitter.com/developerguyba) (shameless-plug ))
- **Best User Adopter** — to the individual, team, or organization who has adopted Sigstore and has shared their impactful story with others (winner [SLSA GitHub Generators](https://github.com/slsa-framework/slsa-github-generator))

You can find all the other highlights about the SigstoreCon in the blog post written by [Tracy Miranda](https://twitter.com/tracymiranda) and officially published on the official OpenSSF website [here](https://openssf.org/blog/2022/11/15/sigstorecon-highlights/). I was also honored to be selected as one of the program committee members of the SigstoreCon, announced via the blog post [here](https://blog.sigstore.dev/sigstorecon-program-announced-509efb918970) and highlighted in the SigstoreCon program announcement on [the official Sigstore blog](https://blog.sigstore.dev/sigstorecon-program-announced-509efb918970).

Sigstore has always been sharing monthly updates on its [blog](https://blog.sigstore.dev/), where you can find helpful resources, news, and planning related to its community. Please subscribe to this blog to keep up with the latest updates.

Then, in the [September 2022 Update](https://blog.sigstore.dev/sigstore-update-september-2022-bb7a25a3d287), I heard about the reward system for the first time and was very excited about it. I was then notified of an [issue](https://github.com/sigstore/community/issues/124) created by GitHub for the Best Sigstore Evangelist selections because people nominated my name by mentioning me in this issue, which was a wonderful, unforgettable moment in my life as I saw how many great friends I made during my OSS contribution experience.

![](/images/evangelist2.png)

### Steps to Becoming an Evangelist

As you see from my story, the Evangelist is a person who is actively spreading knowledge about Sigstore to other projects and people in the ecosystem. But at that time, there was nothing like an evangelist, so I made my contributions only to help the community to adopt Sigstore tools to their projects for a better and more secure future. I was not thinking about becoming an evangelist at all. 😇

If you would like to share Sigstore with the broader community, you need to constantly search for projects to adopt Sigstore tooling into it; of course, it’d be good if it is a CNCF project 😁. While it can help, you don’t need experience contributing to open-source software (OSS) projects. Still, from now on, I highly encourage you to start and gain experience working on OSS projects. Trust me, when you start contributing to OSS projects, the feeling that comes after your first pull request merges is priceless.

In addition to contributions to spreading the adoption of Sigstore to other communities through code contributions, you can also share knowledge. You can become a Sigstore evangelist by writing blog posts, recording videos, doing live streams, organizing events, and more: these are also counted as contributions. You have everything I didn’t have so that you can learn Sigstore tooling quickly, such as the [edX course (LFS182x)](https://edx.org/course/securing-your-software-supply-chain-with-sigstore?utm_medium=partner-marketing&utm_source=affiliate&utm_campaign=linuxfoundation&utm_content=blog-lfs182), [SigstoreCon YouTube Playlist](https://www.youtube.com/playlist?list=PLj6h78yzYM2MUNId2hvHBnrGCCbmou_gl), and [Chainguard Academy](https://edu.chainguard.dev/) to achieve your goal of becoming the next Best Sigstore Evangelist.

So, please waste no time and start working on your goal of becoming the next Best Sigstore Evangelist because there are still so many opportunities; if you have any questions or need help, please feel free to ping me 🤝 on the [Sigstore Slack](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ).

Before closing, I’d like to talk a little bit about my contributions and give you a bunch of issues that can help you to make an impact in the Sigstore community!

### Talk is Cheap: Show me your Contributions.

In this section, I will introduce some of my notable contributions and give you some open Issues so you can start contributing. There are different ways to contribute to OSS projects to take advantage of Sigstore tools, here’s what comes to my mind in the first place based on the path I followed:

1. Add Cosign into the release workflow to make artifacts signed
2. Embed signing and verification logic by using Sigstore SDK
3. Sign Helm charts stored as OCI Artifacts

These are the main categories above that I focused on while contributing to the OSS projects. My two notable contributions were one for Flux and another for Kyverno projects with the help of my best friend, [Furkan Türkal.](https://twitter.com/furkanturkaI)

Here are the links that you might want to take a look at:

- [**[RFC-0003\] Implement OCIRepository verification using Cosign**](https://github.com/fluxcd/source-controller/pull/876)
- [**Retrieving public keys for signing/verifying container images with other locations like KMS, URI, etc.**](https://github.com/kyverno/kyverno/pull/2607)

Here are the links that can give you a chance to make an impact:

- [**Feature: Enable package signature validation with cosign**](https://github.com/crossplane/crossplane/issues/3048) **from cross-plane**
- [**CFP: upload Helm charts to OCI registry and sign them with cosign**](https://github.com/cilium/cilium/issues/22157) **from cilium**
- [**upload Helm charts to OCI registry and sign them with cosign**](https://github.com/cert-manager/cert-manager/issues/5566) **from cert-manager**

I look forward to seeing your contributions!