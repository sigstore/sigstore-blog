+++
title = "New Sigstore Landscape: Add your signed project"
date = "2022-11-30"
tags = ["sigstore","rekor","cosign","openssf","Software Supply Chain"]
draft = false
author = "sigstore"
type = "post"

+++

A Sigstore section was added to the [Open Source Security Foundation (OpenSSF)’s Landscape](https://landscape.openssf.org/).

The aim of the [Sigstore Landscape](https://landscape.openssf.org/sigstore) is to show the collection of technologies that make up the project’s growing ecosystem. This gives everyone a great overview of how everything fits together.

![](/images/landscape1.png)

### Landscape Sections

The Sigstore Landscape currently has seven different sections.

#### **Architecture/Spec**

Sigstore is a new standard for signing, verifying and protecting software. It can be used to make sure your software is what it claims to be. This section links to the formal description of the architecture of Sigstore. The architecture document may be submitted to a standards body (e.g. IETF) at some point in the future.

![](/images/landscape2.png)

#### **Projects**

Sigstore is made up of a combination of technologies to handle signing, verification and provenance checks that respect privacy and work at scale. This section shows the open source subprojects that make up Sigstore.

![](/images/landscape3.png)

#### **Deployments**

In some cases, Sigstore services are run as public instances, for example, the public instance of the Rekor transparency log used to verify signatures. This section allows you to discover public instances run by the community and organizations that host Sigstore services.

![](/images/landscape4.png)

#### **Integration**

Use of Sigstore is sometimes transparent to users as its signing and verification functionality is seamlessly integrated with version control or build software. This section highlights open source and closed source tools, platforms and applications that integrate Sigstore functionality such that users of those tools may benefit from software signing and verification.

![](/images/landscape5.png)

#### **Language Clients**

To make it easier to integrate Sigstore into every ecosystem, language-specific Sigstore clients are being developed.

![](/images/landscape6.png)

#### **Signed With**

This section highlights open source and closed source software that use Sigstore for signing artifacts. That means that users of these tools are able to verify the integrity of artifacts using Sigstore.

![](/images/landscape7.png)

#### **Case Studies**

This section showcases organizations that currently use Sigstore as part of their software supply chain security toolbox. That means the organizations are, at a minimum, signing internal artifacts with Sigstore. Each organization links to a specific case study to highlight how they are using Sigstore and their journey to adoption.

![](/images/landscape8.png)

### Contribute to the Landscape

The Sigstore ecosystem is ever-growing and we want to hear about your Sigstore Case Study or see your signed project on the Landscape!

- Add your signed project by [creating a pull request](https://github.com/ossf/ossf-landscape)
- [Submit a case study](https://docs.google.com/forms/d/e/1FAIpQLSdN-VAxG97mk4KKX5mdvKyWLRdTbUEo0rqV6ZX5HmN_5eIa7Q/viewform)

The Contribution Guide is [available here](https://github.com/ossf/ossf-landscape/blob/main/README.md#new-entries).
