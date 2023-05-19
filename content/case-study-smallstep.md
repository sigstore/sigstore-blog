+++
title = "Bringing Privacy and Security Full Circle Through Automated Authentication"
date = "2023-05-19"
tags = ["sigstore","cosign","supply chain security","open source","devops"]
draft = false
author = "sigstore"
type = "post"
+++

_This is a Sigstore case study contributed by Max Furman of [Smallstep](https://smallstep.com/)_

![](/images/Case%20Study%20Smallsteps.png)

No matter how well you think you secure your software supply chain, the possibility of a breach is always in the back of your mind. We recently had a moment of panic where we thought, “Did someone get access to some things we stored in secrets?” I wondered what damage they’d done and how quickly we could repair it. 

Then I remembered: we'd stopped storing long-term secrets in our CI/CD pipeline completely.

Sigstore offered the perfect solution for our security company. Smallstep was founded on asymmetric cryptography, and we believe that ephemeral certificates are the best way to secure communications between the software infrastructure components. Sigstore shares this philosophy and our vision of software supply chain security. Users shouldn’t worry about what’s under the hood, what certificates they’re signing, and what authentication method they use.

### Making Better Security Easier

In today’s DevOps environments, developers increasingly rely on open-source components to fast-track release cycles, and many businesses use open-source platforms to run their day-to-day operations on a budget. There’s a big market for verification and authentication tools, and [Smallstep](https://smallstep.com/) was founded to meet the needs of this broad customer base. 

Smallstep makes automated certificate management tools for DevOps and DevSecOps. Smallstep issues and renews identities for internal workloads, people, and devices using automated TLS/SSL certificates. Smallstep also provides single sign-on SSH to bridge the gap between identity providers and internal servers, securing mobile and IoT devices and streamlining manual workflows.

We at Smallstep make better security easier for our customers, whether they’re using our open-source repositories for their home labs or our hosted solutions for SMB and enterprise applications. We were confident in the value we provide through our authentication products - and to have that confidence we constantly have to look at our own authentication and authorization practices.

### Reassuring Our Customers with a New Solution

I’m an engineer with over a decade of experience, and I’ve been with Smallstep since 2016. I was the company’s first hire and have progressed from building out concepts and writing the code for our certificate authorities to building our back end. I work directly with our entire engineering team while also maintaining and securing our DevOps pipelines and corresponding with our open-source community.

In my time here, our tools and workflows have evolved. But at one point, we were uploading our releases to GitHub using personal access tokens and TLS instead of using sha1sum and other tools that allow users to verify an artifact. We were uploading our artifacts and trusting the pipeline to be secure, which was fairly safe as long as someone downloaded the artifact from GitHub. However, we couldn’t ascertain the integrity of an artifact downloaded from a mirror. Theoretically, somebody could alter our code to make it produce certificates that gave bad actors access to a user’s system internals, allowing them to decrypt a company’s communications.

> **If customers see you aren’t operating securely, how can they trust you with their business?**

As a company that offers certificate management and authentication tools, we recognized that our customers have to trust our internal verification workflows. Our open-source offering is a funnel to our hosted platform. If people saw we weren’t uploading our artifacts in the most secure way possible—especially as a security company—they might start to wonder whether we had the experience and expertise to handle their security. Could they even trust us? Users let us know they wanted better authentication of our artifacts, and we had to take action if we didn’t want them to take their business elsewhere. 

Our CEO, Mike Malone, had a close relationship with the co-founders of [Sigstore](https://www.sigstore.dev/), so we began to explore some of their tools. Sigstore's Cosign quickly emerged as a way for us to automate our artifact signing and verification, making the process much more secure. We tried it out, and Cosign immediately solved our problem of uploading these artifacts in a uniform way. 

### Cosign Was an Instant Hit

Apart from our good relationship with Sigstore and one of their solutions solving our problem, we also shared a common vision of easily fitting into existing deployment pipelines. We could have set up our own flow for generating these sums and posting them to the various places where we upload artifacts, but then we would’ve had to maintain those flows. Sigstore continues to integrate with many places where we release and deploy artifacts, so it’s useful for us that they maintain all those pipelines. All of our work is based on open standards, and Cosign is doubly attractive because it adheres to our philosophy and frees us from manually signing our artifacts. 

At the same time, the developer and open-source communities were already coming around to using Cosign, even though it was a relatively new product. We didn’t receive any questions from the Smallstep community after we adopted Cosign, which led me to believe that people already understood how Cosign works and how it authenticates artifacts. That initially surprised me, but then I realized that our customers tend to be on the bleeding edge of software supply chain security: If it’s out there and it works, they’re already using it. So, we either follow their lead or take the initiative and get to the next big thing first.

Internally, the feedback was awesome, and people were excited to start telling our users about it. Our developers were happy because they no longer had to create and store secrets for each new repository. It’s been about a year since adopting Cosign, and now we all take the solution for granted. It’s in the background, adding a layer of security to our workflows that is completely transparent to our developers.

> **Developers shouldn’t worry about what’s under the hood, what certificates they’re signing, and what authentication method they use. It should all happen in the background.**

Coming from a place of a shared philosophy of openness and vision of frictionless security, Cosign and Smallstep use the same technology to achieve different ends. Our software automatically signs TSL/SSL certificates and provides single sign-on SSH authentication. Cosign authenticates our codebase artifacts. It is a partnership built on trust that reassures our customers we have their security in mind.

### Integrations That Lead to Seamless Workflows

When we first started using Cosign, they were just beginning their integrations. We had to create an asymmetric public and private key pair for each of our GitHub repositories and releases. I would upload the private part to GitHub Secrets as an action and then use it to sign the various new artifacts being released for that repository. It was time-consuming and took me away from building pipelines.

Cosign recently launched a [GitHub Actions integration](https://github.com/marketplace/actions/cosign-installer) that lets you use a token inside an action and exchange it for an ephemeral certificate within a very brief time frame. I use that certificate to sign and release the artifacts to our community. I don’t have to maintain and manage dozens of public and private keys, and there’s no chance that bad actors can get a hold of them and compromise our codebase. It simplifies my workload and provides peace of mind by eliminating user errors that can lead to security breaches.

Once I enter our GitHub Actions passwords, my team never has to worry about signing our artifacts. Cosign takes care of that part of our CI/CD pipelines. With the most recent Cosign update, everything happens under the hood. My developers don’t need to ask whether everything is working or whether our sha1sums were uploaded to the correct repository. Our workflows are seamless, and we are moving away from passwords, keys, and secrets. 

### Taking Steps Toward a Passwordless and Keyless Future

Cosign is helping Smallstep create a passwordless and keyless future one step at a time. For example, we are still releasing Debian repositories and RPM artifacts, but are confident that Cosign will soon offer integrations with these platforms. Automatically signing these families of releases is an incremental change that will eliminate more secrets and passwords and move us closer to our goal. 

In the meantime, we will continue standardizing and streamlining our workflows to offer our customers better automated certificate management and single sign-on tools.
