+++
title = "Rekor v2 - Cheaper to run, simpler to maintain"
date = "2025-04-18"
tags = ["sigstore","rekor","transparency"]
draft = false
author = "Hayden Blauzvern, Google"
type = "post"
+++

We are very excited to announce the first release of Rekor v2!

[Rekor v2](https://github.com/sigstore/rekor-tiles) is a redesigned and modernized
[Rekor](https://github.com/sigstore/rekor), Sigstore's signature transparency log,
transitioning its backend to a modern, tile-backed transparency log implementation to simplify
maintenance and lower operational costs.

Major changes include:

* A new storage backend, replacing [Trillian](https://github.com/google/trillian) with
  [Trillian-Tessera](https://github.com/transparency-dev/trillian-tessera). Tile-based logs
  are cheaper to run and easier to deploy, maintain and scale. To learn more about the benefits
  of tile-based logs, read [this blog post](https://transparency.dev/articles/tile-based-logs/)
* A redesigned and simplified API, using the learnings from operating public-good Rekor over the past 2 years
* Stronger security guarantees that the log remains append-only by integrating
  [witnessing](https://blog.transparency.dev/can-i-get-a-witness-network) directly into Rekor (To be implemented)

For the initial release, we are providing a binary and container for developers. Note that the API and libraries
are likely to change, and we don't recommend deploying this in a production environment at this time.
If you have any feedback or find bugs, please [file an issue](https://github.com/sigstore/rekor-tiles/issues).

To learn more about the goals and design of the project and to download the latest release,
go to [sigstore/rekor-tiles](https://github.com/sigstore/rekor-tiles), the temporary repository
for this project. You can also follow along with development on our
[project tracker](https://github.com/orgs/sigstore/projects/14).

In the upcoming beta launch, we plan to have client compatibility with the new API along with an instance
of Rekor v2 in our staging environment. Look forward to a GA production launch later this year!

If you have any questions, please reach out on Slack on the `#rekor` channel.
