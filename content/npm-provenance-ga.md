+++
title = "npm's Sigstore-powered provenance goes GA"
author = "The Sigstore Technical Steering Committee"
date = "2023-10-03"
draft = false
type = "post"
tags = ["sigstore","npm","security","node"]
+++

Last week saw the [GA release of npm CLI’s native Sigstore functionality](https://github.blog/changelog/2023-09-26-npm-provenance-general-availability/), a project [over a year in the making](https://www.wired.com/story/github-code-signing-sigstore/). This is a tremendous milestone for the adoption of Sigstore in open source projects and represents huge progress in effecting a cultural shift toward expecting provenance to exist for software components. 

During the [public beta period](https://blog.sigstore.dev/npm-public-beta/) (which lasted from April 2023 until September 26), over 3,800 projects have adopted build provenance (including 134 high-impact projects), resulting in [TK download count] downloads. 

npm is the world’s largest package registry, handling tens of billions of downloads per month and around 40k publishing events each day. The decision to place [SLSA-compliant](https://slsa.dev/spec/v1.0/provenance) provenance-generation functionality into the CLI directly (as opposed to locating it in a CI job or external tool) represents bold leadership in the community of package management tooling. In essence, npm is saying that package maintainers ought to have access to first-class tooling for producing authentic information about the source code and build instructions that produced their package. The effort to produce that functionality in npm resulted in the creation of two powerful new JavaScript libraries: the [sigstore-js](https://github.com/sigstore/sigstore-js) package which produces and verifies Sigstore signatures, and [tuf-js](https://github.com/theupdateframework/tuf-js), which enables the npm CLI to use Sigstore’s TUF trust root to ensure secure communications with the public good instance. Both of these libraries have been donated to their respective organizations, ensuring that the work done by the npm team can have positive network effects in the future.

We see npm’s work to integrate Sigstore as an exemplar for other package managers, which we codified into our roadmap as a [strategic priority](https://github.com/sigstore/community/blob/main/ROADMAP.md#focus-on-oss-package-managers-as-the-primary-path-for-sigstore-adoption-in-the-oss-ecosystems). The more ecosystems adopt signing and provenance, the more confidence the OSS community (and downstream dependers across OSS and proprietary source) will be able to have in the building blocks of open source. That’s an exciting future to be in, and one that we’re working toward every day.