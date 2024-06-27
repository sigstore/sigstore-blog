+++
title = "sigstore-go verification and signing now in beta"
author = "Zach Steindler (GitHub)"
date = "2024-06-26"
tags = ["sigstore", "go", "clients"]
type = "post"
draft = false
+++

# sigstore-go verification and signing now in beta

[sigstore-go's v0.4.0 release](https://github.com/sigstore/sigstore-go/releases/tag/v0.4.0) includes [signing support](https://github.com/sigstore/sigstore-go/blob/main/docs/signing.md), as well moving both the verification and signing API from unstable to beta.

sigstore-go is used in several open source projects like the [SLSA verifier](https://github.com/slsa-framework/slsa-verifier), the [GitHub CLI](https://github.com/cli/cli), and [Stacklok Minder](https://github.com/stacklok/minder).

Cosign and sigstore-go are similar in that they are both written in Go, but the main differences are that sigstore-go is not a full-fledged CLI, and that it supports the [protobuf bundle format](https://github.com/sigstore/protobuf-specs/blob/main/protos/sigstore_bundle.proto). We're hopeful that cosign will use sigstore-go in the near future to add protobuf bundle support. If you use cosign's API to sign things (other than containers), try sigstore-go as a lighter weight and more user friendly alternative.

Give it a try and let us know your feedback! There may be minor interface changes between now and the upcoming v1.0.0 release.
