+++
title = "Using Sigstore to meet FedRAMP Compliance at Autodesk"
date = "2022-11-23"
tags = ["sigstore","cicd","software supply chain","security","autodesk"]
draft = false
author = "Sigstore"
type = "post"

+++

> **This is a Sigstore case study contributed by Jesse Sanford of** [**Autodesk**](https://www.autodesk.com/)

![](/images/autodesk1.png)

In today’s *-as-a-Service world, platforms are everywhere. Products as complex as entire operating systems and as simple as shared libraries are built and maintained on them. As software engineers, we leverage them for common capabilities. This allows us to focus on our customer needs and leave other cross-cutting concerns to the subject matter experts. Typically, security and compliance capabilities are particularly well suited for being delegated to the platform. However, in doing so, attention must be paid to make sure that the requirements for using them are not so burdensome as to inhibit the use of the platform. This highlights one of the more difficult challenges faced by platform developers: Striking the right balance between freedom and security. How do you give product developers the tools to build apps while securing those projects at scale?

My job is to answer that question. As a Senior Principal Software Engineer within the Developer Enablement team here at Autodesk, my work is currently focused on software supply chain security, ensuring bad actors do not compromise our product SDLC processes.

If you know of Autodesk, you’re probably most familiar with[ AutoCAD](https://www.autodesk.com/products/autocad/overview), our flagship computer-aided design software that revolutionized architecture, engineering, and structural design. But we also make digital prototyping software, like[ Autodesk Inventor](https://www.autodesk.com/products/inventor/overview?term=1-YEAR&tab=subscription) and[ Fusion 360](https://www.autodesk.com/products/fusion-360/overview?term=1-YEAR&tab=subscription), as well as [3ds Max](https://www.autodesk.com/products/3ds-max/overview?term=1-YEAR&tab=subscription) and[ Maya 3D](https://www.autodesk.com/products/maya/overview?term=1-YEAR&tab=subscription), animation software to develop video games and visual effects for films and television.

### **Securing CI/CD Pipelines without Hobbling Developers**

You may have also heard of[ Autodesk Platform Services (formerly known as Forge)](https://forge.autodesk.com/blog/autodesk-forge-becoming-autodesk-platform-services), our cloud platform for building customized integrations and supporting applications for our product suite. Autodesk is investing heavily into modernizing our platform offering and our Developer Enablement team is in the center of that platform vision. A critical component of that vision is the Cloud Deployment Platform that I am helping to build now. It creates a common CI/CD workflow and standardizes the delivery of our cloud services.

I came to Autodesk after spending three years managing cybersecurity at a fintech startup. Like many startups, the company moved fast, and it was hard to implement broad security measures without slowing the development cycle.

I took that lesson to heart. When I joined Autodesk as an engineer, I saw an opportunity to bake security measures into our CI/CD workflows. This approach wouldn’t hobble our developers or affect the pace of software delivery.

### **The Path to a FedRAMP ATO**

FedRAMP is a standardized approach to security assessments, authorization, and continuous monitoring required for all cloud products and services used by the US Federal Government. The certification process magnified the need for our software supply chain security initiatives. Among those, securing our containerized workloads and their software supply chains was a precondition to gaining our ATO (Authority To Operate).

One of the first solutions we put in place was to track the provenance (aka how our container images were built, who built them, and where they originated) of our container artifacts. FedRAMP compliance requires that all containers be built from base “hardened” golden images. This means that the operating system, and any software installed on the base image, must pass hardening benchmarks defined by the National Institute of Standards and Technology (NIST) National Checklist Program (NCP). Additionally, the container images must pass vulnerability scanning and scans must be done every 30 days.

Both of these requirements were ripe additions for our CI/CD automation. If we could demonstrate provenance, prove that the containers were scanned by our approved security tooling, and that it had been done within the 30-day limit, we would be “golden.”

### **The Difficulty with Securing Containerized Workloads**

Securing containers during the build and deployment process is a complex procedure. For containerized apps we start with static analysis tools to scan for problematic code and bad libraries in the app code base. Later we run dynamic analysis on the resulting containers hosting the apps. We use both off-the-shelf third-party and home-grown special purpose tooling to do the scans. In total, multiple overlapping layers of pre-deploy security tools are used. Each of them comes with its own costs and complexities and as such, we are very cognizant of how they impact our development workflows.

The usual course of action when creating and sharing container images involves publishing an image to a remote container registry. However, just the image is published. There’s no stamp on that image that says, “X person produced this image at X time on this build server, and it passed its security scan.” In fact, the log of security scans is usually stashed away inside the security tooling and not easily accessible. Developers can only access the logs of these scans with an API, and only if they’re on the right network and have the necessary permissions. If someone pulls down the container image, they will need to jump through hoops to prove it was scanned and be certain it’s safe to run.

Automation is better; but it still has to be built on a case-by-case basis, depending on which solution is used to deploy the container and which cloud is being used. You still must jump through hoops for each unique workflow; each time re-implementing the connectivity back to that security scan API server to connect and validate the results. It’s a time-consuming, duplicative process that slows the production of new solutions on new clouds.

### **Eliminating Attack Vectors without Additional Infrastructure**

As stewards of developer enablement, any time we can optimize for developer productivity we do so. One such opportunity arose during our container provenance implementation.

To make the process of tracking container provenance possible, we investigated several solutions, including Docker Content Trust and Notary. These and other systems achieve a basis for provenance based on public key infrastructure (PKI) or, in the case of Notary, self-signed asymmetric keys. Sounds great — but then I put on my developer enablement hat, such a solution would require us to either roll out a whole PKI system, somehow utilize one of our existing PKI systems, or ask developers to manage their own keys. So while verification is achievable with these technologies, it would greatly impact how we build software.

During my search, I learned about[ Fulcio’s public key signing ceremony](https://www.youtube.com/watch?v=GEuFsc8Zm9U) on a thread for software supply chain security. Fulcio is part of[ Sigstore](https://www.sigstore.dev/), a free, open source standard for signing, verifying, and protecting software. Sigstore provides a suite of tools to sign and validate container images and to store those digital signatures in OCI compliant registries. With it, we can prove provenance and certify containers without adding more infrastructure.

Sigstore includes Cosign (container signing), Rekor (transparency log and timestamping), and Fulcio (root certification authority). Additionally, Sigstore integrates with OpenID Connect (identity protocol for authenticating sessions and users).

Given the number of open source components in free, commercial, and enterprise software, companies must catch malicious code before a product is released. Whether software is released to customers or sits in an organization’s infrastructure, there is little a container can’t run. A bad actor could package any number of exploits into one, potentially using it as an attack vector. Autodesk, like most companies, has multiple layers of security to catch suspicious activity, but that comes at a cost. Having an incident response team dedicating their time and effort to every potential security issue is resource-intensive, and there’s no reason to allow a situation to get that far.

Sigstore’s suite helps Autodesk eliminate bad actors from our software supply chain. We could help reduce the amount of time spent by our developers implementing provenance and vulnerability scanning attestation, as well as help achieve our FedRAMP ATO status in one fell swoop.

### **Integrated Container Signing and FedRAMP Compliance**

Since our cloud deployment platform had standardized our CI and CD processes, trialing Sigstore’s suite was straightforward. Natural extension points in our pipelines allowed us to plug in Cosign container signing and validation. It only took a week to set up a Sigstore proof of concept and a few more to build it into our deployment platform. Our product teams spent a couple of months validating, and we launched our container provenance solution around May 2022.

Sigstore’s tooling is transparent, and yet many of our developers aren’t even aware it exists. They build and run an application using our deployment platform just like they have been. However, they no longer need to integrate the vulnerability scanning API to validate their containers are compliant. All containers created in CI pipelines are scanned by our security tooling, signed by Cosign, and that signature is verified in our CD pipelines before deployment. Every container Autodesk ships now meets our FedRAMP container compliance requirements.

### **Using Sigstore to Secure Our Software Supply Chain**

We haven’t completed a cost analysis yet, but we know Sigstore saved us hundreds of hours in our CI/CD implementation. Additionally, removing the need for Public Key Infrastructure has material operational cost savings.

In the coming months, we hope to integrate Fulcio, Rekor, and keyless signing with our single sign-on (SSO) solution and move toward validation through machine identity. Removing the need for pre-shared secrets would go a long way toward simplifying Autodesk’s software supply chain and overall security posture as well.