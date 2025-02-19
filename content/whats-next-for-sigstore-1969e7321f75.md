+++
title = "What’s Next for Sigstore?"
date = "2021-05-29"
tags = ["sigstore","crypto","infosec","kubernetes","docker"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

![](/images/next1.jpg)

Photo by [Joshua Hoehne](https://unsplash.com/@mrthetrain?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

If you’re new to the Sigstore project, we officially launched on [March 9th 2021](https://www.linuxfoundation.org/en/press-release/linux-foundation-announces-free-sigstore-signing-service-to-confirm-origin-and-authenticity-of-software/) with a mission of improving the open source supply chain by making it easy to sign and verify code. We’re planning to provide free tools, APIs, and services as a public-benefit/non-profit. This post is to give a quick recap of where we are today, where we’re headed and what we’re focusing on next. I’ll also outline some areas we need help, and how to get involved!

While the public announcement was only a few short weeks ago, the Sigstore community started coming together about 9 months ago. Today we’ve grown to [30 contributors from across 10 different organizations](https://insights.lfx.linuxfoundation.org/projects/sigstore/dashboard), with 250 people in our Slack instance! We need every bit of your help, and we thank you for participating!

![](/images/next2.jpg)

Let’s get onto some project updates!

### Rekor

![](/images/next3.jpg)

You would never plug in a thumb drive you found on the sidewalk, but most people don’t think twice about installing something off of NPM or PyPI. Unfortunately today, there’s not much of a difference between these. This is where Rekor, our Binary Transparency project comes in!

We started on [Rekor](http://github.com/sigstore/rekor) back in June with a goal of providing a transparent, auditable discovery mechanism for verifiable supply-chain metadata. Our plan for Rekor is to allow users to discover and verify all of the provenance data associated with any random artifact they find, even on a sidewalk!

We’ve been working directly with open source communities while developing Rekor, and that constant feedback has helped us improve and solidify our design. We’re hoping to build on that progress and to publish our first beta API version next month. This release will include support for new supply-chain metadata types like [In-Toto Metablocks](https://in-toto.readthedocs.io/en/latest/model.html), [WASM signatures](https://github.com/sigstore/rekor/issues/239), [Alpine packages](https://github.com/sigstore/rekor/issues/240), and a few other API refinements to better align with the [Trillian](http://github.com/google/trillian) data store.

While we still won’t be ready for anyone to rely on the log as a production service for a little longer, this more stable API will unblock support for more tooling and getting this in the hands of more developers to play around with. **Please be aware** that we may need to delete existing entries or rotate signing keys with little to notice as we harden our infrastructure and trust roots.

### Trust Roots

Speaking of trust roots, that’s next on our list too! We’re currently in the process of designing our [plan for root key management](https://github.com/sigstore/fulcio/issues/12). These root keys will eventually be used to sign and protect all the other keys we use from day to day across the project, including **everything from the keys used to sign our Transparency Logs to our Root CA itself**. While the [Transparency Logs](http://transparency.dev/) can help us detect and mitigate any issues in the long term, we still need to do the best we can to manage our key material.

We’re excited to have the support of the TUF community on this design. We’re currently working with [Santiago Torres](http://twitter.com/torresariass), [Marina Moore](https://github.com/mnm678), and others to design our playbook for bootstrapping our community-based, distributed, hardware-based root key system. We’re hoping to publish the first draft of this plan by mid April for community feedback. I’m really excited for this part! We have a lot of fun ideas for the initial key ceremony and the ongoing rotations :) Follow along [on GitHub](https://github.com/sigstore/fulcio/issues/12) if you’re interested!

This part is going to take some time, but we promise it will be worth the wait! Even after we’ve designed and implemented this plan, we’ll still need to exercise ALL the scenarios in our playbook. We can’t rush this step, it is needed to give the sigstore community and the maintainers confidence in our operational systems. Until we’re finished here, our Transparency Log and CA services (Fulcio/Rekor) are only **recommended only for demonstration and test purposes**. This is going to take some patience, but we’re working as quickly as we can here!

### Tooling

Time for some good news! For everyone that can’t wait to get started using sigstore, we’re happy to announce the v0.2.0 release of [cosign](http://github.com/sigstore/cosign) — our trigonometry-filled tool for signing containers images. You can try out cosign to sign and verify containers across over 8 different public registries! Cosign currently supports signing with your own keys or KMS systems, and we’re working on some plans to validate our workflows by signing some other real production images of our own too.

Full “keyless” / ephemeral signing using OpenID is still experimental, however if you’re really curious to try this out, you can always ask for help setting it up, in our [community slack workspace](https://github.com/sigstore/community)! Cosign is also available on [GitHub actions](https://github.com/sigstore/cosign-installer), several different package managers and published as a container for use in other CI workflows.

We already use cosign to sign our own releases, but we’re also integrating it into the release process of a few other important container images from outside the sigstore project. We’ll have some exciting updates coming here soon! If you maintain any important OSS images and are interested in trying out cosign early, we’d love to help! Please reach out via our [mailing list](https://groups.google.com/g/sigstore-dev) or slack.

Up next in cosign, we’re working on support for [Yubikeys](https://github.com/sigstore/cosign/issues/108) and other hardware keys, more build system and KMS integrations, and as well as figuring out our plans to move the other sigstore services out of experimental. We do expect that the signature, payload and OCI registry formats will change in future releases as we work upstream on the specifications we need, but we don’t anticipate these changes will be very disruptive.

Going forward, we’re expanding our tooling support across the sigstore project with other signing tools designed for different language ecosystems. We’re just getting started on [maven](https://github.com/sigstore/sigstore-maven-plugin) and [ruby plugins](https://github.com/sigstore/), as well as an overall [sigstore](http://github.com/sigstore/sigstore) “kitchen sink” signing tool. Please let us know what you think about these integrations and how we can make the experience as seamless as possible!

### Getting Involved

We truly welcome contributors and users to our community. We take pride in being friendly to new folks and fostering a welcome and safe environment. Being a large open source project, there is always lots to do and it’s not always complex coding tasks, helping with documentation, general testing or just telling others about sigstore are all valued contributions.

If any of the stability warnings above make you sad and the dates we have in mind seem too far out for you, come help out! We tag all new contributor issues with a [good-first-issue](https://github.com/sigstore/rekor/issues?q=is%3Aissue+is%3Aopen+label%3A"good+first+issue") tag, which is a great way to get your feet wet working in the Sigstore ecosystem.

If you’re stuck and can’t find any ideas, you could also join our office hours! These are casual calls where maintainers are just working on issues together and we’re always happy to help get new contributors setup to help out. We run these during US office hours and Euro / Asian friendly hours. Follow our [community calendar for updates](https://github.com/sigstore/community)!