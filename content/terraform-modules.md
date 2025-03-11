+++
title = "New Terraform Modules Repository"
author = "Sigstore TSC"
date = "2025-03-05"
draft = false
type = "post"
tags = ["sigstore","terraform"]
+++

# New Terraform Modules Repository

The Terraform modules for running a private deployment of Sigstore have been moved to
a dedicated repository, [terraform-modules](https://github.com/sigstore/terraform-modules).

We currently support Google Cloud Platform as a cloud provider. We welcome any community
contributions for other cloud providers.

# Deprecation for Terraform Modules under Scaffolding

Effectively immediately, the Terraform modules in the
[scaffolding repository](https://github.com/sigstore/scaffolding/tree/main/terraform)
will no longer be updated. If you are relying on Terraform for your private deployment,
please update references for scaffolding to the new
[terraform-modules](https://github.com/sigstore/terraform-modules) repository.
