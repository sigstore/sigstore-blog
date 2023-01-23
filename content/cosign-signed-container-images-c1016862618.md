+++
title = "Cosign — Signed Container Images"
date = "2021-05-11"
tags = ["sigstore","golang","kubernetes","Security"]
draft = false
author = "Dan Lorenc"
type = "post"
+++

I’ve seen a lot of questions about signing container images in the last few months, and unfortunately there aren’t many great options or answers today. So I decided to write a simple tool called `cosign`. It can sign container images! Here’s what it looks like to use:

![](/images/cosign.gif)

You can get it installed and start signing containers in minutes. There are almost no configuration options, by design. There is only one supported signature algorithm (ECDSA-P256) and one payload format ([Red Hat Simple Signing](https://www.redhat.com/en/blog/container-image-signing)).

Public keys are stored in plain old PKIX files. They look like:

```
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEroVS8KdYXp5SSI5YDwwQymSByQAM
7MDgk9po3wpp/hHZAzCLsu+j3axrJJ5nMet9tqX1eH8yk21G626Z8lrkQA==
-----END PUBLIC KEY-----
```

Private keys are stored as encrypted PEM files (using `scrypt` and Golang’s `nacl/secretbox` implementation). They should be safe to put in a GitHub repo next to your code, so you can decrypt/sign with a password stored in a secret manager as part of your CI system. Here’s one of mine:

```
-----BEGIN ENCRYPTED COSIGN PRIVATE KEY-----
eyJrZGYiOnsibmFtZSI6InNjcnlwdCIsInBhcmFtcyI6eyJOIjozMjc2OCwiciI6
OCwicCI6MX0sInNhbHQiOiJrSGE5Q1ozTzdFSUtubTNWbnh2WVdvY2k2RWNhcEFD
ZE9NbWxXRTV4YTY0PSJ9LCJjaXBoZXIiOnsibmFtZSI6Im5hY2wvc2VjcmV0Ym94
Iiwibm9uY2UiOiJXcUxtcHZJaml5SklmWGt4YnJaUStXUzg3dFlBeUY1SiJ9LCJj
aXBoZXJ0ZXh0IjoiR1BXMjB3Q1l6d09PeUt2aDMwUzFudnFmLyt0ZVBYSmtXM3F3
TzFEMmwrdk1GQ3o2MHFKU1I1ZTF1UTRlcWxUaTdmdjNkYVlLcUJyNlltcnFGV1Yx
cnlDQ2gwMXhzOGFsd3BxSS85U0pTTjNVVnZXODkxc1hESHc2SEo5dkNIZHdNUldv
WHVVTUdZb0FDd2dyUWZiK3lGNnFOYkQ3dEkrdExlWjRSb1owa3R3Q24zQVErd2hj
U0h4ZjYvQmFNVUwzK1gyK3dnNENVM1dtb0E9PSJ9
-----END ENCRYPTED COSIGN PRIVATE KEY-----
```

The `cosign`signatures are stored in an OCI registry next to the container image, and can be located via a simple naming scheme. Anyone who has access to the image should be able to access and read the signatures. The `cosign triangulate` command can help you find them if you’re curious about what they look like.

If you want to build more advanced workflows support on top of simple signing/verifying, `cosign sign`supports the optional `-a flag` which adds protected annotations (key-value pairs). These annotations are protected by the signature, and can be verified with the `-a` flag to `cosign verify` . You could use this flag to include the build timestamp, git commit or any other information related to how the container was built.

![](/images/cosign2.gif)

Arbitrary blobs can also be signed and verified (`cosign sign-blob` and `cosign verify-blob`), but then you’re on your own for storing and distributing signatures.

`Cosign` has been tested and works with Docker Hub, Google Container Registry and Azure Container Registry. `Cosign` uses the [google/go-containerregistry](http://github.com/google/go-containerregistry) library which has excellent cross-registry support, but some registries struggle with the way signatures are stored/formatted. Let me know if you try others!

I can’t really recommend using this for anything critical yet, but I would love for you to give it a try and see how it might fit into your workflows. I’m going to put it through the paces a bit more myself, then start using `cosign` to sign its own releases. From there, I’ll start publishing public keys and signing some other popular images I maintain. If you’ve read this far, you’re probably using those images in some capacity, whether you know it or not. Stay tuned for instructions on how to check those signatures!

I’m really really excited about this project. We have **a ton** of more awesome features planned here, including:

- Remote KMS integrations to simplify key management
- Support for [Rekor](http://github.com/sigstore/rekor), our Signature Transparency log to help detect key compromises.
- X509 Code Signing certificates from [Fulcio](http://github.com/sigstore/fulcio), our free root CA for Code Signing.

For a sneak peak at what this all might look like put together, see my [tweet](https://twitter.com/lorenc_dan/status/1369721873894944769)!

`Cosign` is developed as part of the `sigstore` project, feel free to join us on [GitHub](http://github.com/sigstore/cosign) or [Slack](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ)! I’m also happy to help you get started, feel free to reach out on [Twitter](http://twitter.com/lorenc_dan)!