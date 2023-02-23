+++
title = "Securing Your Software Supply Chain Without Changing Your DevOps Workflow"
date = "2022-12-15"
tags = ["sigstore","devops","cosign","supplychainsecurity"]
draft = false
author = "sigstore"
type = "post"

+++

> ***This is a Sigstore case study contributed by Tobias Trabelsi of\*** [***DB Schenker\***](https://www.dbschenker.com/de-de)

![](images/casestudy1.png)

DevOps has transformed the way software is built. The practice is ubiquitous, and organizations, big and small, use this approach to streamline development and accelerate release cycles. Many DevOps tools are created and supported by the open source community, but some companies shy away from these applications, preferring enterprise products with 24/7 support and established companies behind them.

However, open source solutions, especially initiatives with active communities of engineers, can provide excellent support and frequent updates by leveraging the collective wisdom of an engaged user base of coders and engineers as well as enabling a level of transparency that cannot be achieved with proprietary solutions. This is especially true of products that protect software supply chains, as[ DB Schenker](http://dbschenker.com/) discovered when we looked to increase security awareness within our Kubernetes clusters.

### **We Couldn’t Trust Our Docker Images**

DB Schenker is a leading global logistics provider. We facilitate the global exchange of goods through land transport, worldwide air and ocean freight, supply chain management, and contract logistics, including warehousing for the automotive industry. I’m a DevOps engineer working for our DevOps Platform team. I am part of a group of six engineers who maintain our DevOps platform and our CI/CD tools to speed up the development process, shorten the feedback loop, and provide stability and scalability to our application teams. We don’t code DB Schenker’s internal and customer-facing apps, but we maintain the infrastructure our engineers use to bring these apps to life as well as working close to the development teams to enable them to build the best software as possible.

As a logistics company, we understand the importance of securing and properly documenting physical supply chains, but developers are increasingly looking at securing software supply chains, too. Within a Kubernetes environment, this, among other things, means validating the Docker images inside our registry. We already had strong transport layer security (mTLS) and validation checks for applications deployed in our clusters, and we trusted our GitLab registry. But there was no way for us to cryptographically validate that images were not compromised already in the registry, and we couldn’t trace the contents of Docker images to their points of origin.

In short, our DevOps team didn’t trust the Docker images inside the registry. However, my team has no say in what our development teams do. Our application engineers are entirely free to do what they want and choose the components in their Docker images, whether they are internal or open source code based on guidelines and a steering Open Source Community. We needed a way to maintain their independence while securing our supply chain. That meant finding a tool that would allow us to give our seal of approval and enforce some vulnerability measures without impeding our coders’ workflow.

### **We Found a Nimble Digital Signature Tool**

We first looked at Notary to manage and sign Docker image metadata, but it was incredibly complicated to set up and maintain. There is too much logic directly tied to the docker cli and the key management needs to be handled either within the notary tooling or manually. We would need to build a new chain of trust only for the sake of notary as it does not integrate well with Hashicorp Vault, which we use as a KMS and primary trust anchor. The automated integration in a Kubernetes admission hook seems also not that common, which started our search for alternative solutions.

Here we discovered Sigstore’s[ Cosign](https://github.com/sigstore/cosign) on Twitter. Cosign was lean, easy to adopt, and had minimal dependencies. It had an excellent integration with our trust anchor tool, so we could use what we already had without changing the infrastructure around it. Deploying Cosign was simple. I enabled the Encryptions as a Service functionality in vault, connect it to Cosign and everything was ready to go. Two weeks later, after we issued a pull request to fix a showstopper for us and a swift release from the Sigstore team, we rolled it out to our environment.

### **Leaning on the Open Source Community**

Using Cosign requires minimum effort — I don’t have to type in hundreds of commands to sign an image — but I still have to support and maintain it. When things do get complicated, I can count on the open source community to help me resolve an issue. For example, when we first rolled out Cosign, we discovered that it generated all signatures with the same time stamps (31.12.2000). This was to ensure the repeatability of signatures independent from changing factors like the current timestamp.

Unfortunately, this workaround conflicted with the GitLab registry’s clean-up policy. In essence, GitLab automatically deletes old builds after a certain number of days or months. But it was also deleting our signatures because deletions were based on time stamps, not file types. We were searching for solutions to this issue and discovered that others were facing the same issue and there seems to be a consensus to change the behavior of cosign in favor of more adaptability.

I worked with the community to resolve the issue, and issued a Pull Request to change this behavior. Even though it was just a few lines of code it enabled us at DB Schenker to move forward with our Cosign rollout. It is empowered to directly interacting with other engineers to resolve a problem that was plaguing our company and other Sigstore users. It felt good to have a hand in writing and maintaining this application instead of calling a 1–800 number, opening a ticket, and letting someone else figure it out when using commercial or enterprise software. The process was a lot faster. Sometimes you have to wait months for a bug fix from a major player, but the open source community is a lot more responsive and agile, which fits perfect in a modern way of working.

### **We Are Exploring Cosign’s Potential**

For now, we are using Cosign at a very basic level to sign our Docker images, but it has so much more potential. We can use this same tool to sign our deployment artifacts and configurations, move toward Supply Chain Levels for Software Artifacts (SLSA) Level 3 compliance, and sign Software Bills of Materials (SBOMs) as well as possibly the deployment manifest as well. We have only begun to secure our software supply chain, and Cosign will allow us to substantially increase trust levels. However, we’re taking baby steps because the only constant in software development is change. We don’t know what new challenges lie ahead, but we know that Cosign is an amazing tool that is easy to manage by our DevOps team and transparent to our developers.

Securing your software supply chain doesn’t have to cost a fortune and shouldn’t require you to rethink your entire DevOps workflow. The days of complex installs and costly, proprietary security solutions may be coming to an end. Cosign is lean and incredibly interoperable, keeps things simple in a complex environment, and can easily be integrated with dozens of other tools. I’ve never seen a signing tool so quickly adopted by the DevOps community. In an ever-shifting environment that is accelerating faster than ever before, tools like Sigstore’s Cosign deliver speed, flexibility and trust while not increasing complexity.