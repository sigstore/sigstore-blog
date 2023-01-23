+++

title = "Signing and Securing Confidential Kubernetes Clusters in the Cloud with Sigstore"
date = "2022-08-11"
tags = ["sigstore","Confidential Computing","Software Supply Chain","K8s","Kubernetes"]
draft = false
author = "Fabian Kammel"
type = "post"

+++

> **This is a Sigstore case study contributed by** [**Fabian Kammel**](https://twitter.com/datosh18) **of** [**Edgeless Systems**](https://www.edgeless.systems/)

![](/images/fabian.png)

Confidential computing is an exciting new technology that can help make the public cloud more secure. It protects data stored on leased third-party infrastructure and ensures nobody modifies or intercepts it, whether it resides on the cloud or is being routed to or from your internal assets. But it’s also vital that security solutions like those of [Edgeless Systems](https://www.edgeless.systems/) are secure themselves.

### Constellation Applies Confidential Computing to Kubernetes Clusters

I joined Edgeless Systems in April 2022 as a Senior Security Engineer on [Constellation](https://www.edgeless.systems/products/constellation/), which applies encryption, integrity protection and attestation to Kubernetes clusters on the cloud, using the latest Confidential Virtual Machine (CVM) technology. This reduces the Trusted Computer Base (TCB) by creating a Trusted Execution Environment (TEE) spanning the whole Kubernetes cluster. For the first time, we will be able to get the trifecta of encryption at rest, encryption in transit, and **encryption in use**, in a powerful and distributed computing environment.

CVMs are 100% isolated from the cloud service provider. With Constellation users can remotely verify their state and build Kubernetes clusters that are not accessible to a cloud service provider and are protected from malware that affects a provider’s network or cloud management software, e.g., the hypervisor.

### Integrating Sigstore as Our Verification Engine

While working with Constellation, our users need to verify that our software supply chains are secure, so they know they’re receiving exactly what we’ve built. These dependencies also come in many different formats: Kubernetes, CoreOS images and go libraries are just a few.

Supply chain security is tough. I first ran into this problem at my previous job at ESCRYPT, where I worked on cybersecurity solutions in the automotive sector. One of my biggest takeaways was that key management was difficult and expensive. We needed special hardware to store our most private keys and set up complex processes to revoke compromised keys and communicate the breach to all affected parties.

This level of security was only available at the enterprise level and was too costly and complex for many startups and open-source users. But when I joined Edgeless Systems, there was a solution I knew could help secure our supply chain: [Sigstore](https://www.sigstore.dev/).

Think of Sigstore as Let’s Encrypt for software supply chains. There are three parts to their solution: Fulcio, Cosign, and Rekor to sign code, verify signatures, and monitor activity, making it safer to distribute and use open-source software. Imagine a free, automated, open tool that allows you to digitally sign and verify components, trace software back to the source, and create a safer chain of custody. That’s the beauty of Sigstore.

### A Fan Since the Start

I first heard about Sigstore shortly after they launched. I read about the project on Reddit, where I saw their[ first blog post](https://security.googleblog.com/2021/03/introducing-sigstore-easy-code-signing.html). I was drawn to it right away for two reasons. The first is UX. It doesn’t require extra effort from end users because it automates the verification process. The second reason was price. You can’t argue with free, and security is typically a cost center that doesn’t add features or bring in new customers.

I brought up my enthusiasm for Sigstore in my interview at Edgeless Systems. During my interview, our CEO, Felix Schuster, asked me what I wanted to address, and I said, “Supply chain security and Sigstore.” He’d already heard about Sigstore and agreed it was something we needed, so I didn’t have to sell him on it. Like our work with confidential computing, it’s clear the people behind Sigstore also believe strongly in security.

A few months into my role, I started integrating Sigstore into our Constellation release process. One of the best things about it is that you can use Cosign, Fulcio, and Rekor independently of each other. Like working in the Linux ecosystem, each of these components does one thing very well, and they all plug into each other very nicely.

Deploying Sigstore was a snap. I downloaded all the tools, plugged in the correct commands, signed an artifact, and pushed everything out in two hours. Once I understood Sigstore’s ecosystem, it was super straightforward.

### Adding Encryption and More to Kubernetes Clusters

Currently, we’re using Sigstore to sign and verify two things: Constellation’s command line interface (CLI) and the measurements of our node images.

Constellation CLI is the application you run on your machine as an administrator to spawn a confidential Kubernetes cluster. We can either deliver the signature as part of our release, or users can use Rekor to verify it.

Most importantly, we use Sigstore to sign measurements related to our infrastructure components which are used in the attestation process. Our application will talk directly to Rekor to fetch the information. This allows us to provide updates for changing infrastructure components. Any new signatures for pushed artifacts are simply made available to Rekor and the verification process remains automated and transparent.

What this means for our end users is that the barrier to security is lowered. Like I experienced at [ESCRYPT](https://www.escrypt.com/en), key management is often only available to enterprise customers. Every time I work with key management, I re-learn how difficult it is. But verifying the supply chain is important for everyone. If we make verification incredibly easy for our users, we expose ourselves to a wider customer base and increase security for everyone.

Sigstore helped us lower the trust bar for Constellation users and allowed us to release months earlier than planned because we didn’t have to code our own digital signature and verification components. It’s easy to use and has everything a developer needs to get up to speed quickly, including automations, integrations, and APIs.

Developers want solutions that are holistically thought through and secure. If your product is secure, but there are holes in the way you deliver it, that makes you vulnerable. You’re only as secure as your weakest security link. And as we’ve seen in supply chain security, there are a lot of weak links.

We all need to work together to make security top of mind. With both Edgeless Systems and Sigstore, we’re trying to lower the barrier to security. Everyone should have access to the tools that make our work and our solutions as secure as possible.