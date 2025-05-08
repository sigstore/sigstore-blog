+++
title = "sigstore-go 1.0 is now available"
author = "Cody Soyland (GitHub)"
date = "2025-05-12"
tags = ["sigstore", "go", "clients"]
type = "post"
draft = false
+++

We love Go within the Sigstore community, and it's been our language of choice since we got started. Cosign, Rekor, Fulcio, Policy Controller, and Timestamp Authority are all written in Go, and we're lucky to have such a vibrant community of Go developers.

Cosign was the de-facto Sigstore "client" from the beginning. Originally designed as a container image signing tool, it has become much more, introducing "keyless signing" using Fulcio, blob signing, attestation support, multi-cloud KMS support, and many more features. Cosign was designed first and foremost as a CLI, and it continues to fulfill that role admirably. However, as we started to build more tools around Sigstore, the Go API started to show limitations, and the dependency tree grew larger. Additionally, as the community grew, we wanted to standardize around simpler data structures and specification-driven workflows.

In 2023, we [began work](https://blog.sigstore.dev/announcing-sigstore-go/) on a new Go library, [`sigstore-go`](https://github.com/sigstore/sigstore-go), to provide a more minimal and friendly API for integrating Go code with Sigstore. The goal of `sigstore-go` was never to replace Cosign, but rather to provide a more modular and flexible API for developers who want to integrate Sigstore into their own applications. Today, `sigstore-go` is used to provide [Sigstore Bundle](https://docs.sigstore.dev/about/bundle/) support in Cosign, and we plan to refactor more of Cosign to use `sigstore-go` in the future. Additionally, GitHub, a major contributor to `sigstore-go`, uses it within [GitHub Artifact Attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds). Other projects, such as [Kyverno](https://kyverno.io/) and [Sigstore Policy Controller](https://docs.sigstore.dev/policy-controller/overview/) also use `sigstore-go` to provide support for the Sigstore Bundle format.

Today, we're excited to announce the release of `sigstore-go` 1.0! This marks the end of a long journey of development and testing, and we're proud to say that `sigstore-go` is now a stable and reliable library for integrating Sigstore into your Go applications. 

Please take a look at the [release notes](https://github.com/sigstore/sigstore-go/releases/tag/v1.0.0). We look forward to your feedback and contributions as we continue to improve and expand the library. If you have any questions or suggestions, please feel free to reach out to us on [GitHub](https://github.com/sigstore/sigstore-go/issues).
