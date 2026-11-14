+++
title = "Mini-Shai Hulud: challenges, opportunities, and the future of the Sigstore ecosystem"
date = "2026-05-22"
tags = ["sigstore","crypto","containers","compromises","slsa"]
draft = false
author = "Santiago Torres-Arias"
type = "post"

+++

A recent headlines-grabbing supply chain campaign, dubbed [mini-shai hulud](), struck last week.
Through the campaign, the attackers managed to compromise about fourty different NPM packages, some with a large amount of users.
However, perhaps the most highlighted aspect of this campaign was not the victims, or the specific platform, but that the attackers went a long way to forge and *log* Sigstore-signed SLSA provenance.

At first sight this may look like a failure of Sigstore, or SLSA, or GHA runner isolation, or a myriad of other factors that take part on the attack chain (see below), but this is really the evergreen story of attackers and defenders racing against each other.
Yet, while the attack ultimately succeeded, there were various aspects of these tools that allowed for quick incident response, as well as widely complicate an attacker's job.
This is why a better approach to study this compromise is to understand the current landscape to build a stronger suite of software supply chain security tools.



# A Quick Background on the Mini Shai-Hulud Attack chain.

There are various writeups on the attack chain of mini-shai hulud. 
Without loss of generality, the major steps that took place in this case were:

1. Identify a compromised GHA runner workflow (i.e., one that didn't force publish flows to the main/develop/publish branch)
1. Modify the runner to generate a malicious package and publish it to NPM
1. Inspect the runner's memory to dump Sigstore's and other secrets (e.g., via /proc/{self,parent,etc}/ introspection
1. Crash the build and publish manually (i.e., bypassing the actions that would normally store the attestations)

As many reports argue, this was a widely complicated attack chain that did a lot of flips to bypass various security mechanisms.
Explicitly put, it:

1. Bypassed GHA protections by abusing misconfigured workflows (indeed, some mitigations is do not allow runs like these)
1. Abused runner secret handling by abusing procfs and runner isolation
1. ``forged'' (not really) Sigstore SLSA attestations by injecting malicious code into the runner and creating the Sigstore signature and SLSA payloads

This is actually not rare, no realistic attack is the consequence of a single point of failure.
Design techniques such as [defense in depth](https://csrc.nist.gov/glossary/term/defense_in_depth) are a reaction to this reality.
Such was grandma's wisdom to ``never put all [defense] eggs in one basket.''


# Sigstore still works! (or, Verification is not Validation)

That said, Sigstore worked! 
In a nutshell, a sigstore signed provenance tells you ``this was built here, using this recipe, and with this output.''
That is very much still true on the forged provenance statements. 
However, the attack is very much there, which serves as a good reminder of the difference between verification and validation.
The distinction is as follows:

- **Validation:** A signature is just a signature, and a statement is just a statement. 
 When you compute a signature over a statement, you know that the owner of the private key (e.g., GitHub, on the project's behalf) actually signed a statement.
 This is, by the way, why we can rest assured that mini-shai-hulud had access to the material needed to create SLSA attestations.
- **Verification:** A statement is a claim backed by a signature. 
 After verifying that the signature is valid (i.e., validation), one must then check what the stated claims exist in a signature, and whether they match reality.
 For example, a signature created by Bob saying "I am Alice" is a valid signature (the math is ok), but it does not mean Bob is Alice --- an obvious yet often overlooked issue in public key cryptography.

In practice, somebody validating the contents of an attestation against the originating repository could have found out that something is odd.
Bob, after all, is not Alice, so what gives?
We will detail a bit more about this below, but it is important to note that, as mature as our signing story is, the verification story can be improved (as research [suggests](https://www.usenix.org/system/files/usenixsecurity25-kalu.pdf)).
Don't fret, we are actively working on this within Sigstore and as part of broader efforts across the open source ecosystem.

Another important aspect of this is that signatures, even if verification is not done (or done poorly) still serves as an important paper trail to audit the impact of a compromise.
The most irrefutable evidence that this compromise was abusing GitHub workflows is that it in fact created provenance with GitHub materials.
Thanks to Sigstore, teams addressing this and other campaigns can initiate their incident response using ground truth data regarding who, what, and where.

# Sigstore as Part of a Larger Supply Chain Security Ecosystem

We are left with the question: what would have stopped this attack?
Let us imagine that there is no adaptive attacker for the sake of simplicity.
Are there tools out there that would fit nicely in the Sigstore workflow that would help prevent attacks of these nature?
A variety come to mind, both from inside, and outside of the Sigstore world --- though we are using inside and outside rather loosely, it's a big family and nobody is excluded.

## Inside of Sigstore: monitoring and hardened signing

Turning incident response methodologies into anomaly detection is a way to quickly operationalize the benefits mentioned above.
For example, if we were able to learn release patterns for a project, or identify identity provider switches, we can immediately flag packages as suspicious, and call for an investigation.
Even more powerful, if we were able to create a policy such as "for every slsa provenance I need to find a signed tag on this repository" we could immediately force attackers to not compromise a single step, but rather a whole software pipeline.

This, and more, interesting policies, can be done by means of strong **monitoring**.
A quick way this could've been identified is if the maintainers of the affected packages had a version of [sigstore-monitor](https://github.com/sigstore/sigstore-monitor) running and configured to notify whenever a signature is created using their identity.
Transparency logs are incredibly useful for this very use-case, and as monitoring matures these types of attacks will become hard to hide.

Beyond monitoring, other extensions to our signing process can allow for client **hardening**.
For example, if the tokens used to generate the provenance (and hell, even the provenance generation code) were kept outside of the purview of the running process, process inspection via `procfs_pid_mem(5)` could not be used to generate forged provenance.
DiVerify is a community-based experimental tool that allows you to separate such a process, and to raise the bar for tool and identity compromise by moving the signing code into a secure enclave, and by aggregating multiple proofs (not just identity providers, but device fingerprints and more) into a single claim that can then be verified.


## Outside of Sigstore: More Attestations and Supply Chain Intelligence

When I mentioned monitoring could compare the results of a repository state with the SLSA attestation or the NPM state, you may have also wondered if this can be automated with more attestations.
If you did, you were right.
Requiring a longer chain of attestations (not just one) can allow us to have a more expressive validation chain. 
Currently, a SLSA attestation can tell you "was this built in GHA using a good runner and on a commit part of a repository".
More attestations can curry this chain of claims to include predicates like "and somebody wanted it to run" (i.e., release attestation), or "and somebody validated the workflow using a bespoke set of rules".
Attestations using in-toto are additive, so introducing and validating more claims force attackers to compromise more portions of the supply chain.
And yes, you could make a monitor verify not only the contents of Fulcio (i.e., the certificates), you could also have them reason about attestations logged in rekor and identify malicious behavior this way.
Cool projects such as [GUAC](https://guac.sh) may allow us to do this at scale.

Finally, the reason we found out this attack happened is because people are hard at work analyzing data to identify malicous packages.
These projects are also part of the ecosystem, and play a central role in identifying things that push-based techniques (e.g., attestations or SBOMs) do not catch.
Constant analysis of code, metadata, and logs will always be helpful to surface misbehavior that went the extra mile to hide itself.
For this, we must continue to push for software transparency, specially in open source, so that supply chain intelligence gathering becomes more widespread, faster, and more precise.


# Wrapping Up: A Reminder of What's to Come

All of the techniques above are somewhat known, and would help prevent future mini shai-huluds.
As a community, we are working towards making these solutions not only available, but widespread.
In order to strenghten sigstore and beyond, we are working on the following:

1. Increasing monitoring capabilities and deployability: people should be able to deploy a monitor easily, and be notified whenever provenance is created for them. 
1. Improving the verification story beyond validation: verifying information signed using sigstore should be as easy as creating signatures using sigstore.
1. Allowing for automated supply chain intelligence work: providing interfaces for teams looking for supply chain campaigns to quickly sweep through data.
1. Introducing hardened signing capabilities: much like 2fa protects the most influential projects in an ecosystem, hardened signing would allow to raise the bar for compromise without losing user friendlyness.
1. Post Quantum Sigstore: this is not exactly against MSH but we did start this post talking about a race between attackers and defenders after all.

These, and more, are efforts that are only possible by committed and passionate members of the Sigstore and sister communities to stop attacks like these and others.
We would be remiss if I did not mention that all of this community is always ready to accept new contributors and that as you can see there is a lot of work to be done. 
Join us!

### Acknowledgements

I'd love to thank [yourself here]
