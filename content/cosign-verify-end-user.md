+++
title = "Verifying Sigstore Budles as an End User"
author = "Zach Steindler (GitHub)"
date = "2025-05-02"
tags = ["sigstore", "cosign", "softwaresupplychain", "security"]
type = "post"
draft = false
+++

There's a mnemonic for quickly determining if a bicycle is safe to ride: "ABC" for checking the air in the tires, ensuring the brakes are functional, and checking the chain. It doesn't definitively answer the question "is this bike safe?" but it does give you a quick starting point for your assessment.

![Mint-colored Surly Straggler bicycle in ready-to-ride condition](/images/bike.jpg)

Let's say you download some software and it comes with a Sigstore bundle. Similarly, there isn't a quick, definitive answer to "is this software safe to use?" but you can use the mnemonic "III" to remember the crucial steps of verifying a bundle: instance, identity provider, and identities.

To start, use an official Sigstore client like [cosign](https://docs.sigstore.dev/cosign/system_config/installation/). There are a [lot of steps](https://github.com/sigstore/architecture-docs/blob/main/client-spec.md#4-verification) clients need to implement to do verification correctly, and you don't want to start from scratch. But even when using cosign, there's a few questions you need to answer in order to verify a bundle.

While there are many different ways to use Sigstore, by far the most popular way (which will help us illustrate the questions you need to answer to verify) is:

- using the public good Sigstore services (e.g. fulcio.sigstore.dev and rekor.sigstore.dev)

- using "keyless" signing (e.g. getting your signing certificate from Fulcio)

- using workload identity from your CI/CD system instead of having a user go through a web-based OAuth flow to provide identity

This is the way package repositories (like npm, PyPI, Maven Central, Homebrew, and Ruby Gems) use Sigstore to provide attestations. Not only does this provide a signature over the bytes of a build artifact, but it also provides non-falsifiable links back to the source code, build instructions, and build logs for that artifact, which is a very useful security capability for vetting dependencies or doing incident response.

To verify, you need to tell cosign what you're expecting for each of those items listed above:

- What Sigstore instance are you expecting this bundle to come from? Public good? A private instance?

- What identity provider are you expecting was used?

- What specific identities **from** that provider are you expecting?

Let's go through each of these questions in more detail.

### What Sigstore instance are you expecting this bundle to come from?

Anyone can set up a Sigstore instance. You want to make sure your bundle came from one you trust, where (among other things) Fulcio was diligent in verifying the OIDC tokens it received. Most of the time this will be the public good instance, but there might be cases where you're using a Sigstore service hosted by a trusted vendor, or maybe even running your own instance.

The bundle contains the signed material, but you also need verification material that corresponds to the instance you're expecting. In addition, you want to obtain that verification material "out of band"; if someone compromises a Fulcio server and starts signing content with a malicious key, it doesn't do any good to ask that Fulcio server what keys you should be using.

It's best practice to have verification material in a trusted root file, which includes ranges of time for when key material is valid, allowing you to easily rotate key material periodically or due to a security incident. It's also best practice to obtain this trusted root file with The Update Framework (TUF) which signs trusted root files, making it difficult to tamper with them.

When using cosign, you can specify `--trusted-root` to provide a trusted root file. If you don't provide verification material, cosign will assume you're using the public good instance and use the corresponding TUF public keys it ships with to fetch and verify updated verification material.

### What identity provider are you expecting this bundle to have used?

The public good Fulcio instance is configured to trust several different workload identity providers, and so the next step in getting ready to verify is to determine what CI/CD system you're expecting was used to produce this artifact.

You can tell cosign what you're expecting with `--certificate-oidc-issuer` (or `--certificate-oidc-issuer-regexp` if you're on a provider where issuers follow a certain pattern). Fulcio will bake the OIDC token issuer into the signing certificate it returns to you, and then cosign will compare what your provide with what's in the certificate in the bundle.

Common issuers include `https://token.actions.githubusercontent.com` for GitHub Actions or `https://gitlab.com` for GitLab Pipelines.

### What identities from that provider are you expecting?

Of course, just knowing that the OIDC token came from a certain provider often isn't enough. In many cases, anyone could sign up for an account with these providers and use the CI/CD system with the public good Sigstore instance. So in many cases you must also specify something that ties the bundle back to a specific account on the provider you're expecting.

You can tell cosign what you're expecting with `--certificate-identity` (or `--certificate-identity-regexp` if you want to trust an account more broadly on a given platform). Fulcio will bake the identity into the signing certificate it returns to you, and then cosign will compare what your provide with what's in the certificate in the bundle.

On GitHub Actions, the certificate identity will look something like `https://github.com/my-org/my-repository/.github/workflows/build_workflow.yml@refs/heads/main`. Where on GitLab pipelines it will look something like `https://gitlab.com/my-group/my-project//.gitlab-ci.yml@refs/heads/main`.

### Putting it all together

Let's take what we've learned about verifying a Sigstore bundle as an end-user and apply it to build attestations in Maven Central.

First let's get our `.jar` and the corresponding Sigstore bundle:

```
$ wget https://repo1.maven.org/maven2/org/leplus/ristretto/2.0.0/ristretto-2.0.0.jar
...
$ wget https://repo1.maven.org/maven2/org/leplus/ristretto/2.0.0/ristretto-2.0.0.jar.sigstore.json
...
```

We're expecting the bundle to use the public good instance, so we can use cosign's defaults there. I want to make sure this `.jar` came from `https://github.com/leplusorg/ristretto`, which of course also means the identity provider is GitHub Actions. So now I have everything I need to call cosign to verify the bundle:

```
$ cosign verify-blob --bundle ristretto-2.0.0.jar.sigstore.json --certificate-oidc-issuer https://token.actions.githubusercontent.com --certificate-identity-regexp='^https://github.com/leplusorg/ristretto/.+' ristretto-2.0.0.jar
no --trusted-root specified; fetching public good instance verification material via TUF
Verified OK
```

Perfect! We checked for updated public good instance verification material out-of-band with TUF. We told cosign not only what identity provider we were expecting, but crucially **what** identities on that provider we trust, and the bundle verified successfully. Now we know the artifact hasn't been tampered with since the build, and we can use the attestation properties as the start of our security assessment.

Instance, identity provider, and identities - as easy as riding a bike.
