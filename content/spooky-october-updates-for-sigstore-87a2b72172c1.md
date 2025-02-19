+++
title = "Spooky Updates for Sigstore!"
date = "2021-10-29"
tags = ["sigstore","infosec","security"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

October is almost done, so it’s time for another update! The supply chains are clearly haunted, so this one has a spooky theme.

![](/images/update1.jpg)

The community is still growing quickly, and the fancy new “[Contributor Strength](https://insights.lfx.linuxfoundation.org/projects/sigstore/dashboard;quicktime=time_filter_6M)” dashboard reflects it!

![](/images/update2.png)

In other numbers, we’re at 820 commits from 85 committers, and our slack channel has reached 710 members! Keep the PRs coming everyone!

![](/images/update3.png)

The Sigstore talk at Kubecon went very well, and the Sigstore booth was a huge success! We had over 300 people attend the virtual booth, and countless people come to the in person one with questions and support!

### Cosign

The next release (v1.3.0) of [cosign](http://github.com/sigstore/cosign) is nearing, with lots of changes and refactoring under the hood. We finally added support for standard POSIX flags ( `—-hello-world`!), and cleaned up our libraries considerably. The [cosigned helm chart](https://artifacthub.io/packages/helm/sigstore/cosigned) has been released in the CNCF Artifact Hub, and we spent some time cleaning up our own repo hygiene using the OpenSSF’s Scorecard project.

The most exciting new feature (in my opinion) is the support for validating [Cue](https://cuelang.org/) and [Rego](https://www.google.com/search?q=rego+language&oq=rego+language&aqs=chrome.0.0i512l5j0i22i30l5.1375j0j7&sourceid=chrome&ie=UTF-8) policies against [In-Toto Attestations](http://github.com/in-toto/attestation). You can see a demo of Cue [here](https://twitter.com/lorenc_dan/status/1447257529386471425). Huge thanks to [developerguy-ba](https://github.com/developer-guy), [erkanzileli](https://github.com/erkanzileli), and [Dentrax](https://github.com/Dentrax) for getting this in!

### Rekor

The last piece of work in [Rekor](http://github.com/sigstore/rekor) before declaring GA is temporal sharding of the log, which [Lily Sturman](https://github.com/lkatalin) has been driving. The log is close to 800k entries, including some important ones, like the provenance information generated during builds of the [Distroless](https://github.com/GoogleContainerTools/distroless) container images used to meet SLSA2. See the full blog post from Priya Wadhwa showing how to look these up and verify them [here](https://security.googleblog.com/2021/09/distroless-builds-are-now-slsa-2.html).

### Sigstore

The Sigstore TUF Root V2 has now been signed, thanks to Asra Ali’s hard work! This root contains delegations for Rekor, Fulcio, and the ability to add more federation delegations for other open source projects and communities! In the words of [Trishank Karthik Kuppusamy](https://twitter.com/trishankkarthik), this is “A Big Deal”:

![](/images/update4.png)

### Fulcio

October has been a big month for Fulcio, or free Code Signing CA! Mark Bestavros landed support for [AWS CloudHSMs](https://github.com/sigstore/fulcio/pull/187), we have full support for [GitHub Actions OIDC](https://github.com/sigstore/cosign/blob/main/KEYLESS.md#identity-tokens) tokens, and Bob Callaway finished up a long-standing task around fully propagating the OIDC Issuer URL used for a specific challenge all the way into the final certificates (which live forever in transparency logs).

This means that in addition to an email address or machine account, you can also find the actual OIDC endpoint that Sigstore used to validate the identity against in the x509 certificate. For example, we can see that my email address was validated using accounts.google.com:

![](/images/update5.png)

### Community

We’ve also seen a huge increase in activity outside of the core Sigstore organization, with more and more projects adding support for supply chain transparency. The [Syft](http://github.com/anchore/syft) project from [Anchore](https://anchore.com/) is working on [direct signature and attestation](https://www.google.com/search?q=syft+cosign&oq=syft+cosign&aqs=chrome..69i57j69i59l2j0i271j69i60l4.1024j0j7&sourceid=chrome&ie=UTF-8) support for SBOMs using cosign. [Quay 3.6](https://www.google.com/search?q=quay+cosign&oq=quay+cosign&aqs=chrome..69i57j33i10i160l3.1088j0j7&sourceid=chrome&ie=UTF-8) now supports cosign signatures, and Kyverno 1.5 provides native support for [In-Toto Attestations](https://nirmata.com/2021/10/22/introducing-kyverno-1-5-0-tackling-complex-policies-with-ease/). Keep these integrations coming, we love to see them!
