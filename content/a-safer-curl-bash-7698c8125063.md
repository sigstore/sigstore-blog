+++
title = "A Safer curl | bash ?"
date = "2021-05-01"
tags = ["sigstore","infosec","golang","softwaresupplychain"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

This post is about using container registries (Docker registries, OCI registries, whatever you want to call them) for the storage and distribution of generic, non-container-related binary artifacts.

I explain the reasoning below, but first: code and demos

### Demos!

Here’s a quick walkthrough of a [draft tool (still WIP!)](https://github.com/sigstore/cosign/pull/292) to securely fetch published contents from an OCI registry, called `sget.` `sget`is part of the [sigstore](http://sigstore.dev/) project, and is a standalone client that allows you to retrieve scripts or binaries from any OCI registry.

The `sget` tool works with `cosign`, and is designed to make it easy to do the right thing, by *requiring* users to verify the contents they fetch, somehow. This can be either a `sha256` content check or a digital signature. `sget` and `cosign` are both built to work well with the rest of the components of the `sigstore` project, so users get support for [binary transparency](http://github.com/sigstore/rekor), our [free certificate authority](http://github.com/sigstore/fulcio) and more over time!

Here’s how it all works:

Upload your binary artifact to a registry:

```
$ cosign upload-blob -f artifact gcr.io/dlorenc-vmtest2/artifactUploading file from [artifact] to [gcr.io/dlorenc-vmtest2/artifact:latest] with media type [text/plain; charset=utf-8]File is available directly at [us.gcr.io/v2/dlorenc-vmtest2/readme/blobs/sha256:b57400c0ad852a7c2f6f7da4a1f94547692c61f3e921a49ba3a41805ae8e1e99]us.gcr.io/dlorenc-vmtest2/readme@sha256:4aa3054270f7a70b4528f2064ee90961788e1e1518703592ae4463de3b889dec
```

You get a normal URL your users can use to fetch it with curl or standard tooling if they want, and a built in digest to verify:

```
$ curl -L gcr.io/v2/dlorenc-vmtest2/artifact/blobs/sha256:97f16c28f6478f3c02d7fff4c7f3c2a30041b72eb6852ca85b919fd85534ed4b > artifact$ curl -L gcr.io/v2/dlorenc-vmtest2/artifact/blobs/sha256:97f16c28f6478f3c02d7fff4c7f3c2a30041b72eb6852ca85b919fd85534ed4b | shasum -a 25697f16c28f6478f3c02d7fff4c7f3c2a30041b72eb6852ca85b919fd85534ed4b -
```

As always you can sign with a key, a token or nothing at all!

```
$ cosign sign -key cosign.key gcr.io/dlorenc-vmtest2/artifactEnter password for private key:Pushing signature to: gcr.io/dlorenc-vmtest2/artifact:sha256–3f612a4520b2c245d620d0cca029f1173f6bea76819dde8543f5b799ea3c696c.sig
```

For automatic digest checking, signature verification and binary transparency, users can also use`sget` to fetch artifacts by digest using the OCI URL:

```
$ sget us.gcr.io/dlorenc-vmtest2/readme@sha256:4aa3054270f7a70b4528f2064ee90961788e1e1518703592ae4463de3b889dec > artifact
```

You can also use sget to fetch contents by tag. Fetching contents without verifying them is dangerous, so we require the artifact be signed in this case:

```
$ sget gcr.io/dlorenc-vmtest2/artifact
error: public key must be specified when fetching by tag, you must fetch by digest or supply a public key$ sget -key cosign.pub us.gcr.io/dlorenc-vmtest2/readme > fooVerification for us.gcr.io/dlorenc-vmtest2/readme —
The following checks were performed on each of these signatures:
- The cosign claims were validated
- Existence of the claims in the transparency log was verified offline
- The signatures were verified against the specified public key
```

The signature, claims and transparency log proofs are all verified automatically by `sget` as part of the download.

`curl | bash` isn’t a great idea, but `sget | bash` is slightly less-bad!

### What Did I Just See?

The OCI Distribution API provides a few ways to do this in practice, but the basic idea is that you take some random artifact (could be a binary executable, an OS package, a video file, a set of YAML configurations, anything), wrap them in some JSON and tarball formats to make them look like a Docker image, and push them to any docker registry. Your clients then use a special tool to pull the files from the registry and extract them back out from their JSON/TAR wrapping.

While people (including myself) have been pushing the limits of the registry APIs and doing things like this since registries first existed, the technique has continued to grow in popularity. Helm was probably one of the first large projects to start to “[formalize](https://github.com/helm/community/blob/main/hips/hip-0006.md)” this model with their experimental support for OCI storage. [TektonCD](http://tekton.dev/) followed suit quickly after in [TEP-0005](https://github.com/tektoncd/community/blob/main/teps/0005-tekton-oci-bundles.md), and the CNCF [ArtifactHub](https://artifacthub.io/) project now lists support for the discovery of [nine different formats](https://artifacthub.io/).

Recently projects outside of the “cloud native” umbrella have even begun to switch to this model, with the [HomeBrew](https://github.com/Homebrew/brew/blob/master/Library/Homebrew/github_packages.rb) package manager migrating binary storage to GitHub Container Registry after the shutdown of BinTray. The [WASM community](https://www.solo.io/blog/announcing-the-webassembly-wasm-oci-image-spec/) is also making use of OCI registries for binary distribution, with the new WASM Hub. A new registry called [bundle.bar](http://bundle.bar/) has even appeared specifically designed for storing other, small config files!

![](/images/curl.png)

Throughout this whole process, there has been considerable debate on whether or not this is a good idea. Matt Farina [summarized a lot of the pros and cons here](https://codeengineered.com/blog/2021/oci-artifacts/), and seems to be leaning against this approach. I’ve gone back and forth myself over the last few years, changing my mind a few times in the process. **Over the last few weeks though, I’ve decided this is a good idea.** With a few caveats, I now believe we should encourage using OCI artifacts as general purpose artifact stores.

Let’s explain why!

# Why OCI? More Like OC-Why-Not!

I think there are two main arguments in favor of using an OCI registry rather than a plain blob store or file server for non-container-related binary artifacts. There are probably dozens of reasons to not do this, but like all good blog posts I’ll pretend those don’t exist for now.

The first big plus is out-of-the-box support for content-checking. The OCI registry API, while minimal, does provide some great building blocks for a secure package management system. Whether you choose to build a full [TUF](https://blog.sigstore.dev/the-update-framework-and-you-2f5cbaa964d5) implementation or stick with the defaults, OCI registries are a better starting point than a GCS/S3 bucket for the majority of use cases. Sure, publishing hashes and uploading to versioned names with `gsutil cp`isn’t that hard, but only a tiny percent of your users will check the hashes, and distribution is tricky enough to trip up many users.

If you want to release a few versioned binaries, **put them in a registry**. You get a better-than-average release archive system out of the box, which is important when it comes to supply-chain-security. OCI registries are immutable and content-addressable, so by publishing things inside them your users get two sets of URLs for free: floating, tag based ones that can contain semantic information around release names and “tracks”, and immutable, content-addressable ones that cannot change over time.

Clients can decide (with your guidance) whether to use the floating tags to always get the latest release, to pin to fixed versions by resolving floating tags to a digest, or even to mix and match between these. You don’t have to publish digests out-of-band. You don’t need to worry about scripting a digest checker and verifier. **You get tags and digests, easily verifiable and immutable, for free.**

The second major benefit is the **ecosystem**. The GCS/S3 APIs are relatively commoditized, but not really changing. OCI registries are formally standardized, supported by dozens of providers, and have a thriving ecosystem innovating both inside AND outside of the specification. Building on this API means your system can take advantage of new registry features and tooling developed by the community while you sleep.

![](/images/curl2.png)

Building on other APIs, even when they seem like an awkward fit is always less fun than building your own custom-system. This is a virtuous cycle — people are building things for the OCI api because it’s commonly used, which leads to even more people adopting it!