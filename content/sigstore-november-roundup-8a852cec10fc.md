+++
title = "Sigstore November Roundup"
date = "2022-11-10"
tags = ["sigstore","python","sigstorecon","ga","rekor"]
draft = false
author = "sigstore"
type = "post"

+++

### Sigstore GA

Sigstore is excited to announce General Availability (GA) for the Rekor transparency log and Fulcio certificate authority public benefit services! The community has been working hard all year to accomplish this milestone, and we are thrilled that open source communities can now confidently rely on Sigstore for production-grade stable services for artifact signing and verification.

[Read the Full Post](https://blog.sigstore.dev/sigstore-ga-ddd6ba67894d) by the Technical Steering Committee

![](/images/ga.png)

### SigstoreCon Recap

SigstoreCon on October 25 in Detroit was Sigstore’s first-ever event and we’re so happy to say that it was a success!

Thank you everyone for the:

- awesome talks; thank you to the speakers
- full program covering various topics by a dozen different companies; thank you to the Program Committee
- great organization; thank you to the Linux Foundation Events Team
- energy; thank you to all our attendees

[Watch the talk recordings](https://www.youtube.com/playlist?list=PLj6h78yzYM2MUNId2hvHBnrGCCbmou_gl)

{{< youtube  WRDTpNWq6sE>}}

### Sigstore Awards

🏆 At SigstoreCon we also hosted our first award ceremony!

![](/images/awards.png)

The 2022 Sigstore Award Winners are…

#### **Most Valuable Contributor**

This award is for the individual who has made a huge impact to the project this year.

🏆 **Asra Ali —** Asra has built many of the fundamental components in Rekor, and her work on the Sigstore TUF root of trust has been so critical to the security and GA launch for Sigstore. Beyond contributing directly to Sigstore, Asra has done a lot to build on top of Sigstore too with her work on SLSA!

#### **Best Evangelist**

This award is for the individual who has gone above and beyond to spread the word about Sigstore

🏆 **Batuhan (developer-guy) Apaydin** — developer-guy has done amazing work spreading knowledge around Sigstore in many different ways (blog posts, tweets, meetups, videos) which have been instrumental in bringing newcomers to the community, helping them get up to speed faster and feel more comfortable.

#### **Best User Adopter**

This award is for the individual, team or organization who have adopted Sigstore and have shared their impactful story with others

🏆 [**SLSA GitHub Generators**](https://github.com/slsa-framework/slsa-github-generator) The SLSA GitHub Generator project hosts a collection of trusted builders that can produce SLSA Level 3 compliant provenance. This project is a key part of connecting Sigstore to the wider supply chain security ecosystem and has also been a key source of feedback on both feature enhancements as well as on regressions and issues in Sigstore services.

Congratulations! 🎉 And thank you for everything you’ve done for the Sigstore Community.

![](/images/sigstorecon.jpg)

### Python Continues to Embrace Sigstore

The new release of [Python 3.11](https://www.python.org/downloads/release/python-3110/) was one of the most exciting Python releases in a while, not just for the significant speed upgrades, but also it is one of the first new versions of Python to be signed with Sigstore by default. Read more on [Sigstore verification of Python Releases](https://www.python.org/download/sigstore/).

In addition, [sigstore-python](https://github.com/sigstore/sigstore-python) 0.7.0 was released this past month. This release now supports offline verification of Rekor entries, the ability to verify non-email identities, [and more](https://twitter.com/8x5clPW2/status/1588562179044691969?s=20&t=ZrxX9WmrfCbsc2z0unFYYA).

### New Case Study

Brandon Gulla, CTO at Rancher Government Solutions, contributed a new Sigstore Case Study.

Read it: [Sigstore Proves That Effective Supply Chain Security Doesn’t Have to Hurt](https://blog.sigstore.dev/sigstore-proves-that-effective-supply-chain-security-doesnt-have-to-hurt-cf33cf9333c8)

![](/images/rancher.png)

### Blog: How Sigstore quickly patched an upstream vulnerability

Hayden Blauzvern contributed a blog post about a Sigstore vulnerability found in June by Joern Schneeweisz from the GitLab Security Research Team. [Find out how it was fixed](https://blog.sigstore.dev/how-sigstore-quickly-patched-an-upstream-vulnerability-76ba84ef1122).

### Latest Releases

#### Sigstore

Sigstore is currently on [version 1.4.5](https://github.com/sigstore/sigstore/releases/tag/v1.4.5).

#### Cosign

Cosign is container signing, verification and storage in an OCI registry. Its latest release is [v1.13.1](https://github.com/sigstore/cosign/releases/tag/v1.13.1).

#### Fulcio

Fulcio issues code-signing certificates bound to OpenID Connect identities for use within the Sigstore ecosystem. Its most recent release is [v1.0.0](https://github.com/sigstore/fulcio/releases/tag/v1.0.0)!

#### Gitsign

Keyless Git signing with Sigstore! Its latest release is [v0.3.2](https://github.com/sigstore/gitsign/releases/tag/v0.3.2).

#### Rekor

Rekor’s aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. Its latest release is [v1.0.0](https://github.com/sigstore/rekor/releases/tag/v1.0.0)!

### Join the Community

Sigstore welcomes new contributors and users with open (source) arms. We take pride in being friendly to new folks and fostering a welcoming and safe environment. There is always lots to do for everyone, no matter your experience level (not all of them being complex coding tasks).

Valued contributions include:

- helping with documentation
- general testing
- sharing your love of Sigstore (Tweet about us [@sigstoreproject](http://projectsigstore/)) 🐦

Join our [Slack workspace](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ)!