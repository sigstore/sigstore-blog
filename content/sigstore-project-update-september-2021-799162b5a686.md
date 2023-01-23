title = "Sigstore project update — September 2021"
date = "2021-10-01"
tags = ["sigstore","security","devsecops"]
draft = false
author = "Luke Hinds"
type = "post"

+++

Well another month has passed and as per usual in the sigstore world, a lot has happened!

Since our last update in August we have **over double the amount of contributors** working on sigstore! There has been a leap from 46 to 98! wow!

![](/images/sep21.png)

### KubeCon NA 20201

sigstore will be at [KubeCon North America ](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)with it’s own booth, so if you’re in person at the event, come and say hi! We will be at booth number S86

![](/images/sep211.png)

There will be swag available in the form of a stickers and security keys. However there is small caveat that you try out sigstore with one of our community members, so come on over at get your stuff signed!

![](/images/sep212.png)

![](/images/sep213.jpg)

### Sigstore talk

On Wednesday October 13 11:55am — 12:30pm. Dan Lorenc and Bob Callaway will be performing a talk on sigstore ‘How we started, where we are, and where we are headed’.

### Project Updates

#### **Rekor**

Our transparency log is edging close to a 1.0 release now, while we just fine tune the API for our approach to sharding.

sharding is required if ever a corruption occurs, this allows us to freeze the former log and create a new one, with no loss of data or service.

Once sharding is landed, we cut 1.0 and rekor is then considered production grade.

#### cosign

cosign is developing fast. Work is being carried out to refactor the CLI over to [cobra](https://github.com/spf13/cobra) and we recently included github’s OIDC into cosign (more on the OIDC change under fulcio), along with support for GCP environments without workload identity.

cosign is now at [version v1.2.1](https://github.com/sigstore/cosign/releases/tag/v1.2.1)

#### fulcio

GitHub recently released its own OpenID connect functionality, which enables sigstore to provide an even more seamless keyless signing experience.

With fulcio keyless signing it is required that a developer be present to perform the OpenID connect grant. Sigstore clients open a browser to our oauth2 server, which then prompts the user to ‘login’ with a provider account (Google, Github, Microsoft etc). This grant then allows us to obtain the email addresss of the user and place it into a sigstore X509 signing certificate.

Now the above works great for what we term an ‘attended signing’, but what about ‘unattended’, e.g. a headless machine with no human present to interact with the browser to allow the grant.

With Github OIDC now, we can provide an unattended experience to signing artefacts (such as a container image) within a github action. The great factor here, is that GitHub have added sigstore as an OIDC Audience (idToken Field).

![](/images/sep214.png)

Keep an eye out for an entry on this blog on how to get set up for unattended sigstore signing!

### Community

As always, the slack channel is counting to grow, we are now at 646 members

The [community meetings](https://github.com/sigstore/community#community-meetings) are also seeing lots of new faces, do come along if you have any interest in sigstore.

### Talks

There have been a few talks recently.

[Luke Hinds spoke at #romhack in Rome Italy about sigstore.](https://www.youtube.com/watch?v=JXcqX5ozuvc)

[Dan Lorenc Chatted with commiting to cloud native](https://www.youtube.com/watch?v=JhqQ_6Z1SCA)

[Luke Hinds, technically speaking with Chris Wright](https://www.youtube.com/watch?v=TGC1zT_BNBM&t=341s)

[Andrew Block discusses the sigstore helm plugin](https://www.youtube.com/watch?v=cjY26RRHpo8)