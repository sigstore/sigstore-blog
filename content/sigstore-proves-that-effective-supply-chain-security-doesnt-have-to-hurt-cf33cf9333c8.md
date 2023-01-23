+++
title = "Sigstore Proves That Effective Supply Chain Security Doesn’t Have to Hurt"
date = "2022-10-22"
tags = ["sigstore","Supply chain security","case study"]
draft = false
author = "Sigstore"
type = "post"

+++

> **This is a Sigstore case study contributed by Brandon Gulla, CTO at** [**Rancher Government Solutions**](https://rancherfederal.com/)

![](/images/rancher.png)

Traditionally, everyone in IT assumed good security had to hurt a little bit. If it didn’t hurt, security wasn’t strong enough. But computing trends in software supply chains have shifted in recent years, moving toward centralized development and software factories. When you have that common infrastructure throughout the organization, you can isolate a lot of that pain within the process — without too much developer interaction and disruption.

I’ve always been a hands-on engineer, working primarily in the US Department of Defense and the US federal community. In my tenure with those three-lettered agencies, I’ve seen the challenges and opportunities within the US government that separate that space from the commercial world. Many of these have to do with security.

### Recent Attacks Illuminate Supply Chain Threats

Computing security has shifted over the past two decades. At one point, it was about “security by trusted vendor,” meaning that something was safe if it originated from a well-known software organization in the continental United States. While the public sector once was hesitant to adopt open-source software, the Big Data boom of the 2010s ushered in a new willingness to collaborate. Open-source software lowers the barrier of entry to adoption, but it also comes with inherent threats. While the community is collaborative and leads to empowerment in the software artifacts, the model breaks the chain of trust, or at least the idea that there is a collective unit responsible for the health and integrity of a software package.

Before the open-source model became prevalent, anyone who came across a bug or critical vulnerability could call a 1–800 number and connect with someone at a software company to alert them of the issue. Today, there typically isn’t one established company behind a particular library or software project. Time and time again, we’ve seen organizations establish their entire foundation on a tool that isn’t commercially supported, and even closed-source entities still rely on these open-source projects. While the shift toward open-source computing has opened many doors for software and cloud-native adoption, it has also complicated the security story.

In the Kubernetes and containerized world, the reality was a reactive security posture that identified security threats after they caused an issue on a network. We see it all the time: images are shipped and ran in production, with no idea of the integrity or the vulnerability of the image baseline until after deployment. To mitigate this risk, we have had to enhance security throughout the entire software pipeline, and evaluating library and image dependencies is just the start.

Before the SolarWinds attack, identifying threats within dependencies fell to developers. Everyone was in control of their piece of the puzzle and would make assumptions based on the existing supply chain. Recent attacks have proven that isn’t enough anymore. The entire pipeline must be verified secure, not just part of it. That SolarWinds attack revealed just how critical it is to scan not only your dependencies but everything before that — the dependencies of the dependencies of the dependencies — to ensure everything originates where it says it originates and their packages are also secure.

As the federal market has shifted to a software-factory methodology that centralizes DevSecOps processes, we’ve seen those gate checks move into the software build itself. Several worlds are colliding at the perfect time to help facilitate a more secure and declarative cloud-native development experience, and [Rancher Government Solutions](https://rancherfederal.com/) is poised to take advantage of this change.

### The Power to Thrive in a Highly Restricted Space

By nature, the government tends to lock things down and restrict the number of software libraries and components that establish their infrastructure. In many instances, they must go a step further and disconnect from the public internet. This isolation leads to an air gap situation, complicating security and dependency management. Most cloud-native tooling leaves much to be desired in such a restricted setting.

Rancher’s portfolio of cloud-native tools embrace the philosophy of non-prescriptive, lightweight infrastructure. In founding Rancher Government Solutions (then named Rancher Federal), I set out to treat air-gap customers as first-class citizens. We wanted to accelerate the path to production for our air-gap customers, without the compromises that are often inherent with other tightly-coupled Kubernetes vendors.

Rancher Government Solutions addresses our US Federal customers’ unique security and operational needs as they relate to application modernization, containers, and Kubernetes. Many users assume that just because Rancher is now owned by the operating-system company SUSE, that our willingness to support other software architectures (OSs, distributions, etc) would change, but that could not be further from the truth. That extends to Rancher Government Solutions as well. We are a US-based FOCI-mitigated company dedicated to supporting Rancher’s Kubernetes and cloud-native capabilities across the Department of Defense (DoD), the intelligence community, and civilian agencies.

My colleagues and I are not new to the challenges of the US public sector. We have each spent decades as “hands-on-keyboard” engineers, thriving in these complicated, highly sensitive environments. Our team takes software builds from Rancher and SUSE, signs them, enriches them with attestation reports and software build materials, and then hosts them privately for our internal customers. Essentially, we take the community versions of Rancher and SUSE software assets and wrap them in body armor for our federal customers.

### Fortifying the Cloud-Native Experience

Kubernetes has been hot within the US government for the past several years. Initially, we saw organizations willing to take risks on this new technology because of its mission value, even if it didn’t check every security box. Over time, these organizations have become more mature and declarative in their security infrastructure, and they now must check every single security box before moving an application to production. Security validation is no longer optional.

On top of that, Kubernetes isn’t as high velocity as it once was. It’s moving horizontally instead of vertically, and we are now seeing a special interest group (SIG) movement to build new supporting capabilities in ecosystems such as storage, networking, and of course, security. Amidst all this movement, Sigstore emerged.

Like many people, I first heard about Sigstore at KubeCon 2021, which put a spotlight on supply chain security. I knew this was a defining moment because supply chain security is typically an afterthought. During a discussion between Chainguard’s Dan Lorenc and Google’s Tom Hennen, I was exposed to how Google fortifies their software build process. Many of today’s cloud-native tooling, including Kubernetes, began at Google, so I knew I had to take a closer look at Sigstore.

### An Invisible Part of the Build Pipeline

Very rarely in the government space does something come online and become the de facto industry standard so quickly, but Sigstore is the exception. Sigstore takes a Linux/Unix approach to tooling, so it’s not a monolith. With components such as Cosign, Rekor, and Fulcio, anyone can pick and choose the best tools for their needs. Not having to establish the entire tool suite of supply chain security right off the bat means you can take a “crawl, walk, run” approach to adoption, which disrupts the developer experience as little as possible. We started small at Rancher, first working on digital signatures, then adding attestations and software build materials.

Seamless integrations make it easy to incorporate Sigstore into an existing pipeline. Where traditionally, a developer might have had to run scans and validate every single image manually, Cosign supports container signing, verification, and storage in an OCI registry. It’s made signatures an invisible part of the build pipeline and moved the task away from the developers themselves. GitHub is also going to continue increasing connectivity with Sigstore elements, including [Gitsign](https://github.com/sigstore/gitsign), which is a great way of tacking onto the current developer experience.

We have more than 500 digitally-signed images in our registry right now. If a sys admin had to validate every one of those and their dependencies manually, the amount of time required would be staggering. It would take years. Meanwhile, software moves faster than ever. Sigstore allows us to constantly validate those images while avoiding such an incomprehensible demand on our workforce.

The biggest thing we have learned in adopting Sigstore is the importance of continually improving your security posture. Proving the providence and validity of your security back to the source is extremely difficult, and there will never be the perfect set of tools. Taking an iterative approach avoids “paralysis due to analysis.” Every step toward a more secure pipeline is an improvement on where you were before.

### Taking an Automated, Proactive Approach to Security

Effective supply-chain security is complex, but implementing Sigstore was more manageable than we expected. Its ecosystem of security-driven tools and thriving community made for a smooth journey. SUSE has been incredibly supportive of our Sigstore adoption because — unsurprisingly — they’ve had great success with it, too. SUSE uses Sigstore as part of their Elemental toolkit, which makes them one of the top 10 users globally.

Sigstore enables us to take an autonomous, proactive approach to security. Rather than relying on reactive, manual processes, we can automatically deny unsigned or vulnerable images from running on a cluster. Sigstore enhances visibility at build time and bakes security components into the build process, helping create a unified security policy.

There’s a lot of strength in providing security without disrupting developers, and we now know we can provide effective supply-chain security without pain. As the number of open-source contributors grows and software becomes increasingly more compatible, it will only get more exciting. Security is good for everyone, so in the end, everyone wins.