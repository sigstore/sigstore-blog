+++
title = "Announcing sigstore-go"
author = "The Sigstore Technical Steering Committee"
date = "2023-10-05"
draft = false
type = "post"
tags = ["sigstore","go","clients"]
+++


# Announcing sigstore-go

Today we’re excited to announce a new open source library, [sigstore-go](https://github.com/sigstore/sigstore-go), that represents the future of Sigstore’s support for the Go programming language. Since the beginning, Sigstore has been primarily written in Go but there has been a gap over the past year or so since we established the [Protobufs-based bundle format](https://github.com/sigstore/protobuf-specs): the de facto standard client ([Cosign](https://github.com/sigstore/cosign)) lacks support for it. Cosign-the-library is also heavily focused on OCI use cases, which makes it difficult for library integrators who want to limit their implementations to core sign/verify flows and it also supports a wide variety of verification options, which creates potentially confusing duplication. Finally, sigstore-go represents the TSC’s vision for future integrations in Go and so it can/should replace utility functionality that has been located in the (confusingly named) [sigstore/sigstore](https://github.com/sigstore/sigstore) lib. It can be used today as a smaller alternative to depending on Cosign's internal libraries (which can come with lots of potentially unecessary transitive dependencies like cloud SDKs and support for OCI registries), and will provide the basis for Sigstore bundle support in the [policy controller](https://github.com/sigstore/policy-controller).

## Customizable trust
The new library implements the “[Signed Entities and Verifiers](https://github.com/sigstore/sigstore-go-archived/issues/35)" interface proposal from earlier this year, giving integrators fine-grained control over what signatories, transparency logs, and timestamp authorities to trust. It also gives users the ability to bring a custom TUF trust root and makes [Rekor](https://github.com/sigstore/rekor) verification (offline or online) configurable and optional for private deployments.

## Developer ergonomics
Users integrating this library into their code get simple, functional-options configuration with sane defaults, an easy-to-process `VerificationResult` [struct](https://github.com/sigstore/sigstore-go/blob/main/pkg/verify/signed_entity.go#L156), and the ability to specify an artifact digest at verification time if desired in order to accommodate situations where the digest may not be the semi-standard SHA-256 used in most examples of provenance calculation (e.g. npm packages use SHA-512).

## Conformance testing
Lastly: it wouldn’t be a contemporary Sigstore client without rigorous conformance testing! This client has been developed with the same rigor as [sigstore-js](https://github.com/sigstore/sigstore-js) and [sigstore-python](https://github.com/sigstore/sigstore-python) to ensure that it truly offers the guarantees that it purports to.

We’re very excited about the future of Sigstore clients in general and eager to see what the community makes with this new library in particular, especially the potential move by [SLSA](https://slsa.dev/) to move their [slsa-verifier](https://github.com/slsa-framework/slsa-verifier) project to it. We will be migrating Go-based Sigstore projects to use this functionality over time, making Cosign and other fundamental pieces of kit both smaller in compiled size and easier to reason about in the process. We invite you to join the conversation in the dedicated [sigstore-go channel](https://sigstore.slack.com/archives/C0440BFT43H) on Slack!