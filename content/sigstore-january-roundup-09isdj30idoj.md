+++
title = "Sigstore January Roundup"
date = "2023-02-01"
tags = ["sigstore","security","software supply chain","supply chain security","update"]
draft = false
author = "Sigstore"
type = "post"

+++

This month, we are thrilled to have [announced the 1.0 release of sigstore-python](https://blog.sigstore.dev/announcing-the-1-0-release-of-sigstore-python-4f5d718b468d). This project started a year ago to provide a Sigstore-compatible client similar to cosign, but built entirely with Python and easily adoptable by the Python ecosystem.

![](/images/python-logo.png)

A big thank you to all the contributors and maintainers for making it to 1.0! [Read more](https://blog.sigstore.dev/announcing-the-1-0-release-of-sigstore-python-4f5d718b468d)

## Latest Blog Posts

Thank you to Andrew, Felix and Zachary for contributing the following blog posts this month. You should definitely give them a read:

* [A Guide to Running Sigstore Locally](https://blog.sigstore.dev/a-guide-to-running-sigstore-locally-f312dfac0682) by Felix Wolff and Andrew Block
* [Why you can’t use Sigstore without Sigstore](https://blog.sigstore.dev/why-you-cant-use-sigstore-without-sigstore-de1ed745f6fc) by Zachary Newman

## Upcoming Events

### CloudNativeSecurityCon NA 

![](/images/csc.png)

[CloudNativeSecurityCon NA](https://events.linuxfoundation.org/cloudnativesecuritycon-north-america/) is happening February 1 – 2 in Seattle, WA.

Congratulations to many members of the Sigstore community giving talks!

Wednesday, February 1
11:50 am [So You Want to Run Your Own Sigstore: Recommendations for a Secure Setup](https://sched.co/1FV3Y) by Hayden Blauzvern
2:15 pm [Lightning Talk: Securing Your Source Repositories - 5 Tips to Get Started!](https://sched.co/1FV0G) by Billy Lynch
3:50 pm [Who Are You? I Really Want to Know...the Magic Behind OIDC](https://sched.co/1FV4H) by Eddie Zaneski

Thursday, February 2
3:50 pm [Not All That's Signed Is Secure: Verify the Right Way with TUF and Sigstore](https://sched.co/1FV2v) by Zachary Newman & Marina Moore
4:40 pm ["Keyless" Code Signing Without Fulcio](https://sched.co/1FUzp) Nathan Smith

### FOSDEM

![](/images/fosdem.png)

FOSDEM is a free event for software developers and takes place in Brussels every year. This year, it’ll be on the weekend of February 4 – 5.

Don’t miss the Sigstore talk in the Security Devroom on Saturday, February 4 at 16:00 [What Does Rugby Have To Do With Sigstore? Learning Sigstore via Rugby](https://fosdem.org/2023/schedule/event/security_rugby_sigstore/) by James Strong & Lewis Denham-Parry.

Many of the Sigstore community folks will be attending the conference, so keep an eye out and say hello!

## Sigstore Landscape

![](/images/landscape10.png)

The [Sigstore Landscape](https://landscape.openssf.org/sigstore) is growing its collection of ecosystem technologies with two new additions: Caddy Server (under “Signed With”) and Open Policy Containers (under “Integrations”).

If you’d like to add your projects, here are [all the details you need](https://blog.sigstore.dev/new-sigstore-landscape-add-your-signed-project-dda0517723b6).

## Office Hours 

Twice a month (or fortnightly, if you please), we host Sigstore Office Hours. Everyone is welcome to join to discuss how you use Sigstore. If you didn’t know about them, feel free to [watch the previous ones here](https://www.youtube.com/watch?v=x86jxe3yfks&list=PLM6mY5TOhY1GKMC-CbRPzm496wTpPyDLN).

https://www.youtube.com/watch?v=x86jxe3yfks&list=PLM6mY5TOhY1GKMC-CbRPzm496wTpPyDLN 


## Latest Releases

### Sigstore
Sigstore is currently on version [1.5.1](https://github.com/sigstore/sigstore/releases/tag/v1.5.1).

### Cosign
Cosign is container signing, verification and storage in an OCI registry. 

The community is working steadily towards a release of Cosign 2.0. The pre-release for Cosign 2.0 is out: v2.0.0-rc.0. Feel free to try it out, but know there are many breaking changes from 1.0 and the prereleases may continue to change. Otherwise, please use [v1.13.1](https://github.com/sigstore/cosign/releases/tag/v1.13.1).

### Fulcio
Fulcio issues code-signing certificates bound to OpenID Connect identities for use within the Sigstore ecosystem. Its most recent release is [v1.0.0](https://github.com/sigstore/fulcio/releases/tag/v1.0.0).

### Gitsign
Keyless Git signing with Sigstore! Its latest release is [v0.5.2](https://github.com/sigstore/gitsign/releases/tag/v0.5.2). Highlights include new features for the credential cache - systemd support and the ability to forward interactive flows over the socket (incl. over SSH)!

### Rekor
Rekor's aims to provide an immutable tamper-resistant ledger of metadata generated within a software projects supply chain. Its latest release is [v1.0.1](https://github.com/sigstore/rekor/releases/tag/v1.0.1).

## Join the Community

We’re looking forward to a great 2023 with all our maintainers, contributors and users. 

We take pride in being friendly to everyone, including new folks, and fostering a welcoming and safe environment. There is always room for more people in our community. 

Find us on Slack and Twitter [@projectsigstore](https://twitter.com/projectsigstore).

See y’all next month!

