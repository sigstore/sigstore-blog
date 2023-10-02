+++
title = "Announcing sigstore-go"
author = "The Sigstore Technical Steering Committee"
date = "2023-10-05"
draft = false
type = "post"
tags = ["sigstore","go","clients"]
+++


# Announcing sigstore-go

Today we’re excited to announce a new open source library, [sigstore-go](https://github.com/sigstore/sigstore-go) that represents the future of Sigstore’s support for the Go programming language. Since the beginning, Sigstore has been primarily written in Go but there has been a gap over the past year or so since we established the [Protobufs-based bundle format](https://github.com/sigstore/protobuf-specs): the de facto standard client (Cosign) lacks support for it. sigstore-go represents the TSC’s vision for future integrations in Go and will be used to eventually make Cosign smaller and more performant in the future, as well as provide the basis for Sigstore bundle support in the [policy controller](https://github.com/sigstore/policy-controller).

## Customizable trust
The new library implements the “[Signed Entities and Verifiers](https://github.com/sigstore/sigstore-go-archived/issues/35)" interface proposal from earlier this year, giving integrators fine-grained control over what signatories, transparency logs, and timestamp authorities to trust. It also gives users the ability to bring a custom TUF trust root and makes [Rekor](https://github.com/sigstore/rekor) verification (offline or online) configurable and optional.

## Developer ergonomics
Users integrating this library into their code get simple, functional-options configuration with sane defaults, an easy-to-process `VerificationResult` [struct](https://github.com/sigstore/sigstore-go/blob/main/pkg/verify/signed_entity.go#L156), and the ability to specify an artifact digest at verification time if desired in order to accommodate situations where the digest may not be the semi-standard SHA-256 used in most examples of provenance calculation (e.g. npm packages use SHA-512).

## Conformance testing
Lastly: it wouldn’t be a contemporary Sigstore client without rigorous conformance testing! This client has been developed with the same rigor as sigstore-js and sigstore-python to ensure that it truly offers the guarantees that it purports to.

We’re very excited about the future of Sigstore clients in general and eager to see what the community makes with this new library in particular. We will be migrating Go-based Sigstore projects to use this functionality over time and making Cosign and other fundamental pieces of kit both smaller and more performant in the process. We invite you to join the conversation in the [#clients](https://sigstore.slack.com/archives/C03SZ6SHU90) channel on Sigstore Slack!