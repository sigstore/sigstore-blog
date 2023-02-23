+++
title = "Celebrating 1,000,000 entries in Rekor"
date = "2022-01-10"
tags = ["sigstore","softwaresupplychain","softwareengineering"]
draft = false
author = "Santiago Torres-Arias"
type = "post"

+++

We’ve finally reached a million entries! We hit a million entries by the end of 2021:

> rekor-cli get — log-index 1000000
> LogID: c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d
> Index: 1000000
> IntegratedTime: 2021–12–27T21:34:27Z
> UUID: abb93264122dc2eb6da3ac77957978dda3ceb3521ab52cdff8490b23eaf791c1

As a double milestone — the turn of the year and the next order of magnitude — this is a good opportunity to look into what’s become of the Sigstore ecosystem since its inception. Throughout the last year, we’ve seen a lot of different initiatives, new types of signatures added, tools, major features, and more from the whole community. This is why it is important to look back at how tremendous of a year 2021 was and learn more from it.

Simply put, it would be great to know three things: how are people signing, what are people signing and who is signing things in Rekor.

### The evolution of types in Sigstore

Rekor already boasts the ability of signing with more than 10/11 different types of signing mechanisms. This is to accommodate the various ways that users sign and verify for artifacts in the whole supply chain. All in all, users are using all of the types available. However, naturally, it appears that some types are more popular than others.

![](/images/evol.png)

While expected, it does appear that x509 is one of the most popular ways to sign Rekord objects around (in fact, Rekord objects can be signed by x509, pgp, minisign or ssh, more on this later). In fact, it would appear that x509-based Rekord entries comprise almost 79% of the entries in the log. Why do we say this is expected? well, first and foremost, we wanted people to sign their software, and it is not surprising to see that people are using Fulcio + Rekor to sign software, which uses Rekord + x509 under the hood. However, that is not the whole story.

## in-toto goes second, and with a growing ecosystem

As one of the lead devs of in-toto, I was quite pleased to see the widespread usage of in-toto within Sigstore. In fact, I was actually shocked to see that people were using in-toto in ways that I hadn’t expected yet.

![](/images/evol2.png)

For in-toto types, there is a growing ecosystem of types. Although the vast majority of them seem to be provenance types (tekton/slsa/in-toto), there seems to be a growing ecosystem that includes tools like Witness (bundles of attestations), as well as SPDX and Cyclonedx (Software Bills of materials). There is also somewhat of a remnant from old pre ITE-6 attestations (marked as in-toto opaque). For those, we would have to unpack and manually verify what each of them means.

## Ecosystem growth, velocity and shifts in popularity

Something that I personally found interesting was that certain tools were growing at different paces. This is expected, as people move on from signing their release artifacts, to sign other parts of their pipelines. Say, for example, after successfully signing their releases with with Rekord, developers may want to add more information in the shape of in-toto attestations. Let’s take a look at the trends in signing.

![](/images/evol3.png)

First of all, it is amazing to see [that a project created on Jun 2020](https://github.com/sigstore/rekor/commit/e678f70e84ab642d9f27c8e60a9f2d4630d4a86c) (Ahoy Captain!) really saw an uptick of usage in late August of 2021(in fact, usage was quite lacking before that). What could’ve caused this? Well, this actually matches [Priya Wadhwa’s first stress-testing run](https://github.com/sigstore/rekor/issues/519). However, it does appear that after the early delta, there is still plenty of activity going on for the major four types (x509, in-toto, Hashed Rekord and Minisign)

![](/images/evol4.png)

Second, (and let’s zoom into the bottom right of the picture), It does appear that certain signing types are on the upswing. That small line on the bottom right (zoomed in below) corresponds to minisign, that is slowly climbing, along with the the “Hashed Rekord” type and may soon also overtake in-toto usage to claim the second spot!

![](/images/evol5.png)

### What about the others?

The rest of the types may be hard to distinguish in comparison, so let’s take a look at them in isolation.

![](/images/evol6.png)

There we go! Still a healthy growth between all of them. In particular, RFC3161 seems to be on the upswing again (that’s the one in charge of providing verifiable timestamps). Other signing mechanisms, such as TUF and SSH are on the up as well, and I expect them to start coming up once Rekor sees more usage in ecosystems such as GitHub and PyPI. Lastly, package-specific types such as RPM, Alpine and JAR have little adoption (e.g., there is only one Alpine package so far), but will hopefully gain more momentum soon.

### What is being signed?

It is hard to tell what is being signed, as not all the types can tell us information about the package itself. Either way, I took the liberty of checking the most popular hashes (Rekord), Artifact URI (in-toto) and package names (JAR/TUF/RPM/etc) to see how many signatures are over the same data:

```
gcr.io/tekton-nightly/github.com/tektoncd/pipeline/cmd/controller: 70582
2296748d375388175d182715bf4419c44ecc9cae30143097bf6e85dc47340bbf:  33770
8d9e51a1485fbecd5c361452daf9fdeb5e75f90b7a824445d070572e4375d7bf:  33770
da231de2353469c2b92c39be473b3e64a96d3e47fa1f32bf7f86b07f89b4ca3b:  33770
f4fdd3c4fe274b8fee43d8d1fd44a6acb119bc03c0b97dd0118c057d7b478ca5:  33770ecosystem
15a43b62d18bb1ab561276ade8010f479949195bdf7a33b19274288b4d230d13:  33769
4f704e401f364b9798aef32cfb14e713794b5be9165a186c4bc2aa8ee2858b06:  33769
5e16e7933c11752282429cda9dff02fc9a42ef03d597ee446012a0ed48ea0466:  33769
66725be547470c9b23d1d554e4016ca9b1386487579983aa86e776c32f3db28e:  33769
045db3b80adcca3583503b3dd212ca9a4a1a67165ed15ab08df241dcc95cecb2:  28400
```

Ah, well the most signed type is tekton nightly releases (actually, there are a series of containers that I aggregated on the same type at the top). After, it appears that some types are signed thousands of times but, interestingly-enough, they are often repeated. These generally correspond to reproduced images for distroless, that are continuously built and are generally reproducible.

We could, however, filter anything that looks like a hash to see more interesting info:

```
gcr.io/.../github.com/tektoncd/pipeline/cmd/controller: 70582
index.docker.io/myrepo/controller-...: 5371
index.docker.io/concaf/kaniko-chains: 187
gcr.io/distroless/base: 147
gcr.io/.../github.com/tektoncd/trigger/cmd/interceptors: 143
index.docker.io/library/busybox: 107
us.icr.io/gitsecure/ssm-app: 106
scribesecuriy.jfrog.io/scribe-docker-public-local/stub_local: 28
scribesecuriy.jfrog.io/scribe-docker-public-local/stub_remote: 17
gcr.io/tekton-.../github.com/tektoncd/pipeline/cmd/controller: 13
quay.io/shbose/kaniko-chain: 13
ghcr.io/ckotzbauer/message-bot: 12
ghcr.io/ckotzbauer/security-test: 12
busybox: 10
gcr.io/kg-image-registry/gajananan-akmebank-app-cl1:roles-stage: 9
```

### Who is signing?

Last, but not least, let’s take a look at who is signing all of these entries. Below is a list of the top 10 users of Rekord (with some arbitrary redactions to be on the safe side).

```
 keyless@distroless.iam.gserviceaccount.com: 12658
 tekton-chains-controller@[redacted].iam.gserviceaccount.com: 487
 hir[redacted]ahara@gmail.com: 269
 lhinds@redhat.com: 194
 kga[redacted]an@redhat.com: 194
 rkudo@redhat.com: 133
 mattmoore: 203
 leo[redacted]lva@gmail.com: 88
 yuj[redacted]tanabe.ibm@gmail.com: 80
 bcallawa@redhat.com: 53
 dlorenc: 91
 priyawadhwa@google.com: 43
 kaniko-workload@[redacted].iam.gserviceaccount.com: 34
 dfe[redacted]mn@gmail.com: 33
 roc[redacted]vre@shopify.com: 32
 g132[redacted]@is.ocha.ac.jp: 31
```

You could extend the question to what providers are being used to sign and, well, here it is:

```
https://token.actions.githubusercontent.com: 17179
https://accounts.google.com: 6930
https://github.com/login/oauth: 316
https://login.microsoftonline.com: 15
https://allow.pub: 14
https://oidc.eks.xxxx.amazonaws.com/id/xxx: 17
https://oauth2.sigstore.dev/auth: 6
https://any.valid.url/: 3
```

### Closing Thoughts

Overall, it’s heartwarming to see this activity uptick. It’s amazing to see the many ways that users are interacting with the Sigstore ecosystem, and the uptake in various projects that are starting to tighten their software supply chains. Here’s for a more secure software supply chain in 2022!