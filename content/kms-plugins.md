+++
title = "KMS Plugins for Sigstore"
author = "Ramon Petgrave (@ramonpetgrave64)"
date = "2025-02-19"
draft = false
type = "post"
tags = ["sigstore","kms", "plugin"]
+++

Cosign and private deployments of Fulcio and Rekor can use a KMS-managed key for signing artifacts. We currently have built-in support for AWS, Azure, Google Cloud Platform, and Hashicorp Vault KMSs. This has been a challenge for customers that require alternative or custom KMS solutions.

To enable such use-cases, we have implemented a new [plugin system](https://github.com/sigstore/sigstore/tree/main/pkg/signature/kms/cliplugin) for alternate KMS providers. Organizations can independently and privately develop & distribute their plugins without needing downstream updates to libraries to support additional KMS providers as build-time dependencies.

[Alibaba KMS](https://github.com/mozillazg/sigstore-kms-alibabakms) support is now possible thanks to a new plugin by community member @mozillazg.

### How it Works

The out-of-tree Sigstore KMS plugins will be separate, installable programs. Users will invoke Cosign specifying their plugin with the key ref like `cosign sign … -- key mykms://my-key-id`, where `mykms` refers to a plugin program `sigstore-kms-mykms` located on the system PATH. The existing key references for the existing built-in KMS providers will continue to work the same and take precedence.

### Plugin Development

Like [kubectl](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/#writing-kubectl-plugins) plugins, Sigstore KMS plugins can be written in any language on any OS that can write programs that accept CLI arguments and stdin, and output to stdout. Plugin authors see more [details](https://github.com/sigstore/sigstore/tree/main/pkg/signature/kms/cliplugin) and can follow our [example](https://github.com/sigstore/sigstore/blob/main/test/cliplugin/localkms) plugin program as a guide.

Developers that use sigstore/sigstore as a library can also use plugins with the `kms.Get()` function.

The Plugin program will:

1. Provide a handler for all methods defined by the `kms.SignerVerifier` interface, which interact with a real KMS backend.
1. Interpret command-line arguments that correspond to the `kms.SignerVerifier` method arguments.
1. Relay the method arguments to the handler and send the returned values back to the main program via standard output (stdout). Files to be signed or verified will be received over standard input (stdin).

The Sigstore library will:

1. Identify the appropriate plugin program on the user's system based on the `KeyResourceID`. For example, a plugin program for `mykms://my-key-id` should be named `sigstore-kms-mykms` and located in the user's system PATH.
1. Create a `kms.SignerVerifier` object that forwards all method arguments and return values between the main program and the plugin program. Files to be signed or verified will be sent over standard input (stdin).

### What’s Next

We’re continually open to feedback from the community on improvements to the plugin system.
