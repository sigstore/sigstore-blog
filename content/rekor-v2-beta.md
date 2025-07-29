+++
title = "Rekor v2 Beta - Cheaper to run, simpler to maintain"
date = "2025-08-01"
tags = ["sigstore","rekor","transparency"]
draft = false
author = "Hayden Blauzvern, Google"
type = "post"
+++

We are excited to announce the beta release of Rekor v2!

[Rekor v2](https://github.com/sigstore/rekor-tiles) is a redesigned and modernized
[Rekor](https://github.com/sigstore/rekor), Sigstore's signature transparency log,
transitioning its backend to a modern, tile-backed transparency log implementation to simplify
maintenance and lower operational costs. Learn more about Rekor v2 in our
[previous blog post announcing alpha](https://blog.sigstore.dev/rekor-v2-alpha/).

With the beta release, we have added support for Rekor v2 upload and verification to the
[Go](https://github.com/sigstore/sigstore-go), [Python](https://github.com/sigstore/sigstore-python],
and [Java](https://github.com/sigstore/sigstore-java) clients, and released a new version of
[Cosign](https://github.com/sigstore/cosign) which will upload to and verify proofs from Rekor v2 logs.
We have also released a new version of the
[conformance test suite](https://github.com/sigstore/sigstore-conformance) for client
developers to test support for Rekor v2.

To learn more about the client changes, read our
[client documentation](https://github.com/sigstore/rekor-tiles/blob/main/CLIENTS.md).

Look forward to a GA launch later this year!
If you have any questions, please reach out on Slack on the `#rekor` channel.
