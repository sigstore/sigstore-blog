+++
title = "How to Sign a Release of OSS"
date = "2021-05-17"
tags = ["sigstore","infosec","kubernetes","Security"]
draft = false
author = "Dan Lorenc"
type = "post"

+++

I’ve heard a TON of questions about how to sign an open source software release lately. Once you get past the impossible tooling/crypto questions, you quickly realize you’ve barely scratched the surface in complexity. These problems aren’t all specific to OSS, but community-driven projects do face some unique challenges that stretch beyond technical and **into the philosophical realm.**

![](/images/oss1.jpg)

*What* does it mean to sign a release? *Who* should do it? *Where* should the keys live? *How* do users verify it? *Why* are we even doing this again? If you (understandably assumed) that this was a solved problem, you’re in good company but about to be disappointed. **Here’s what I think makes the most sense, and what I’m planning to try on the projects I maintain.**

This approach doesn’t make sense for everyone and probably isn’t perfect. It’s designed to be easy to set up, hard to mess up and better than almost any OSS project I’ve seen. There are two steps. **If all you do is the first one, you’re still doing an amazing job.**

**Before we start:** basic hygiene. REQUIRE 2FA. Branch protection on main **and all the release branches**. Require pull reviews. Require CI (this is important for below). **Think you’re doing this?** Double check all the repos, especially the build ones! See the OpenSSF [Security Scorecards](https://github.com/ossf/scorecard) project for automated tooling.

### Step 1: Signed Builds

Configure your CI/build system to sign every build it executes. Sign an envelope (examples below) containing at least the following:

- The **input parameters** to the build. This should contain the source code versions (i.e. git commit sha) and **any other inputs** that can affect the build, as well as information on why the build was invoked.
- The **environment** the build ran in. This is the OS and version, the cloud environment, the tooling versions, and the status of all build time dependencies.
- The **outputs** of the build. Store the hashes, names and any other metadata you need about the artifacts that were built. Sign the logs too!

For the envelope format: [In-Toto links](https://in-toto.io/) work, [Grafeas Provenance](https://github.com/grafeas/grafeas/blob/f11fa45f2a57406a0b69eb3463ecbca9e127ccd2/proto/v1beta1/provenance_go_proto/provenance.pb.go#L55) is common too. There’s some SBOM stuff on the way. Don’t worry too much and pick one you like and will use.

Projects like [Tekton Chains](http://github.com/tektoncd/chains) are designed to make this easier, but many build systems can be configured or extended to run this way as well. [ITE-6 ](https://github.com/in-toto/ITE/pull/15)is an upcoming standard to define this format across systems.

Store the key in a private KMS system, lock down access to it as much as possible. **Do not export or save private keys** locally. Audit key usage regularly.

Publish the releases somewhere off of your Source Code Management system. If you use GitHub, store your releases and signatures on GCS or S3. **Lock down access to your build system**. Audit access. **Lock down access to your release artifacts** to your build system. Now everything on your release page is signed by the build system, and the build can be verified from source code all the way the to published artifact.

Publish these provenances and signatures next to your releases. Store the public key in the repository. Users can find the public key to use for a release IN the source code itself.

If the entire SCM being compromised is in your threat model, you could also sign git tags. I’d instead recommend running the SCM somewhere you can trust. I recommend that anyway.

If the entire build system being compromised is in your threat model, you could try reproducible builds. I’d again recommend running the build system somewhere you can trust. **Reproducible builds are still a good idea anyway.**

### Step 2: Signed Releases

The system in Step 1 gives users verifiable provenance about an artifact. This can show the source it came from and the tooling used to build and it. This is important, but it cannot tell you that the source code used is “correct”, as defined by the project. An example threat model here is a rollback or freeze attack, where an attacker is able to trick the user into installing a specific older version of the software. All builds can be authenticated back to their source code — we do not know if they were “authorized”.

This is where things get philosophical — what does it mean for an open source community to “authorize” an official release? For most projects this is implicit — someone with access to the releases page creates a release. Some projects use a pull request with approvals from multiple other maintainers before merge. Maintainers can define and follow their own decision-making process here, or not. This can be encoded formally into a policy with something like In-Toto or votes on an email list, but it often isn’t.

If you want to address this threat model, think up and document the policy for declaring a release. Follow this process, publicly. Reviews on a PR or +1s on an email list work fine here. Make it public. This should include the exact git commits somewhere. Document how to verify this all. The Node.js project [does an amazing job](https://github.com/nodejs/node#verifying-binaries) here.

**Encode this approval as another signature**. If the automated signature from Part 1 *authenticates* a release, this manual signature on-behalf of the maintainers *authorizes* the release. Place this (different) public key in the repository also. Use KMS with an IAM role limited to maintainers. Audit access. Publish these signatures anywhere you can, including wherever the initial approvals were created (the pull request, ticket or email thread). Transparency logs are coming!

### Other Stuff If You’re Really Paranoid/Concerned

This does not include TUF, explicitly. I think TUF is great for complex update systems, but it is still overwhelming for the majority of small projects. The **time-stamping protocol is a must-have** for anything that auto-updates, but it introduces too much operational complexity and risk for small projects. **Are you auto-updating anything to customers? Use TUF. If not, you can skip it.**

Lock down your build system. And your SCM system. This is way more important than anything above, but after you finish all of this, go back and lock down the systems even more. Disable all access and audit logins. Make builds hermetic. **Declare all the inputs.** No network access. *No network access.* The Kubernetes release group has done an amazing job here.

This **does not** include revocation. **Key revocation just doesn’t work** in update systems, especially in OSS. **Your revocation system is Twitter** or MITRE. **Revoke artifacts**, not keys. Treat compromised or tampered artifacts **like any other** vulnerable artifact. Your **CVE scanners should** detect these, not your PKI.

We’re making binary transparency easier and automatic in the [Sigstore ](http://sigstore.dev/)project. We’ll be able to protect you and your users against key compromise and targeted attacks **with no action** on your part. You’ll be able to integrate directly for automatic personal key management and offline signed-timestamps soon.

Large, public artifact repositories should prepare for a compromise and plan for recovery. [PyPI has done a great job here](https://www.python.org/dev/peps/pep-0480/). TUF key delegation is the best way to do this today. Keep a few root keys offline, require a quorum to sign the subordinate signing key. Rotate. Mix and match attacks can be scary here too. Use TUF.