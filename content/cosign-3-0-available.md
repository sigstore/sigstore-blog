+++
title = "Cosign v3 is now available"
author = "Zach Steindler (GitHub)"
date = "2025-10-08"
tags = ["sigstore", "cosign", "opensource"]
type = "post"
draft = false
+++

The past few years have been incredible in the Sigstore ecosystem, seeing Sigstore-signed in-toto attestations be adopted by [Homebrew](https://blog.trailofbits.com/2024/05/14/a-peek-into-build-provenance-for-homebrew/) (May 2024), [PyPI](https://blog.pypi.org/posts/2024-11-14-pypi-now-supports-digital-attestations/) (November 2024), [Maven Central](https://central.sonatype.org/news/20250128_sigstore_signature_validation_via_portal/) (January 2025), model signing in [NVIDIA's NGC](https://developer.nvidia.com/blog/bringing-verifiable-trust-to-ai-models-model-signing-in-ngc/) (July 2025), and several others.

These deployments make use of great Sigstore features, like the ability to verify content offline, being able to fetch new verification key material with The Update Framework, and the ability to use a [tile-based transparency log that's much easier to operate and scale](https://blog.sigstore.dev/rekor-v2-alpha/). These features were developed and deployed to client libraries (sigstore-go, sigstore-python, sigstore-java, ...) but for a variety of reasons these new capabilities weren't making it back to Cosign, the flagship CLI of the Sigstore project.

A year ago, with some effort, we made it so [Cosign could verify attestations from these deployments](https://blog.sigstore.dev/cosign-verify-bundles/) with opt-in, off-by-default flags. That was a great step forward, but it only highlighted how far Cosign had fallen behind, and how we'd have to make breaking changes to support the capabilities expected by these newer deployments. We [outlined a path forward at SigstoreCon 2024](https://coffeehousecoders.org/blog/cosign_and_clients.html), and we're now ready to deliver on that plan.

Cosign 2.6.x will support all the new capabilities of the recent deployments with opt-in, off-by-default flags: things like `--new-bundle-format` for having your signed material all in one place including information for offline verification, `--trusted-root` for having your verification material all in one place that supports key rotation without having to update your client software, and `--use-signing-config` so that the transparency log shards can be rotated without having to update your client. In addition, there are subcommands that can help you move to these new capabilities, like `cosign bundle create`, `cosign trusted-root create` and `cosign signing-config create`.

We think these new capabilities, and interoperability with popular public Sigstore deployments, will make these flags quite popular. Starting in Cosign v3, these flags will be **on by default**, but will still allow you to disable them if you want the old operation. We hope that still having the flags available, but switching their default, is a nice balance between indicating the direction of Cosign while also easing the transition for folks who are on private deployments or using the old data formats with the public good instance.

There probably won't be many releases of Cosign v3, and soon we will start removing the old functionality for the initial release of Cosign v4. Not only will this simplify the CLI ([removing roughly half the flags](https://github.com/sigstore/cosign/issues/4354)!) it will also drastically simplify the Cosign codebase, making it easier to maintain for years to come.

Sigstore is successful because of the people using it. We do our best to balance being flexible to support the many ways people use Sigstore without overcomplicating things or breaking existing users. If you have feedback for our plan with Cosign v3, v4, and beyond, [let us know](https://github.com/sigstore/cosign/issues/4221).
