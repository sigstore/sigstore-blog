+++
title = "Homebrew's Sigstore-powered provenance is in beta"
date = "2024-05-14"
tags = ["sigstore", "security", "homebrew"]
draft = false
author = "William Woodruff (Trail of Bits), Hayden Blauzvern (Google)"
type = "post"
+++

Last November, [Alpha-Omega] and [Trail of Bits]
[announced a collaboration] to [bring build provenance to homebrew-core].

Today, we are pleased to announce that the core of that work is live
and in public beta: [homebrew-core] is now using Sigstore to cryptographically
attest to all bottles built in the official [Homebrew] CI.

This is aligned with [Sigstore's mission]: to support frictionless and
transparent provenance on all artifact registries.

Homebrew's build provenance
follows last year's [npm provenance] feature, making Homebrew the second major
packaging ecosystem to adopt Sigstore!

![](/images/brew-verify.png)

In other words, going forwards, each bottle built by Homebrew will come with
a cryptographically verifiable statement binding the bottle’s content to the
specific workflow and other build-time metadata that produced it.

This metadata includes (among other things) the git commit and GitHub Actions
run ID for the workflow that produced the bottle, making it a
[SLSA Build L2]-compatible attestation:

![](/images/github-attestations.png)

This work is still in **early beta**, and involves features still
under [active development] within both Homebrew and GitHub.

However, for the adventurous, we recommend checking out the [Trail of Bits blog]
for a longer explainer on Homebrew's build provenance, how it was implemented,
and how early adopters can begin to play with it!

[Alpha-Omega]: https://alpha-omega.dev/

[Homebrew]: https://brew.sh

[Trail of Bits]: https://www.trailofbits.com/

[announced a collaboration]: https://openssf.org/blog/2023/11/06/alpha-omega-grant-to-help-homebrew-reach-slsa-build-level-2/

[bring build provenance to homebrew-core]: https://repos.openssf.org/proposals/build-provenance-and-code-signing-for-homebrew

[homebrew-core]: https://github.com/Homebrew/homebrew-core

[active development]: https://github.blog/2024-05-02-introducing-artifact-attestations-now-in-public-beta/

[SLSA Build L2]: https://slsa.dev/spec/v1.0/levels#build-l2

[npm provenance]: https://blog.sigstore.dev/npm-provenance-ga/

[Sigstore's mission]: https://github.com/sigstore/community/blob/main/ROADMAP.md#mission-aka-our-purpose

[Trail of Bits blog]: TODO
