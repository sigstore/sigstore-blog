+++
title = "PyPI's Sigstore-powered attestations are now generally available"
date = "2024-11-14"
tags = ["sigstore", "security", "pypi", "python"]
draft = false
author = "William Woodruff (Trail of Bits), Dustin Ingram (Google), Hayden Blauzvern (Google)"
type = "post"
+++

*Check out the [PyPI blog] and [Trail of Bits blog] for more user-facing and
technical details, respectively!*

Over the past year, the [Google Open Source Security Team]
and [Trail of Bits] have worked together to implement [PEP 740],
a Python packaging standard that allows users to upload Sigstore-based
attestations to the [Python Package Index].

Today we're pleased to announce that attestation support on PyPI is
**generally available**, meaning that package maintainers
can submit attestations for both PyPI and downstream users to verify.

An important piece of the story for attestations on PyPI is
**default enablement**: if a project uses [Trusted Publishing],
simply upgrading to [`pypa/gh-action-pypi-publish`] [v1.11.0] or newer will result
in attestation generation by default, with no changes required.

Thanks to this default stance, adoption of attestations by publishers has been
rapid: over 20,000 individual attestations have been uploaded to PyPI so far,
and just over 5% of the top 360 packages are already publishing attestations:

![](/images/pep740.png)

To keep track of the ecosystem's overall progress, follow along with
[Are we PEP 740 yet?], which is automatically updated as more top projects
release new, attested versions.

[PyPI blog]: https://blog.pypi.org/posts/2024-11-13-pypi-now-supports-digital-attestations.md

[Trail of Bits blog]: https://blog.trailofbits.com/2024/11/14/attestations-a-new-generation-of-signatures-on-pypi

[Google Open Source Security Team]: TODO

[Trail of Bits]: https://www.trailofbits.com/

[PEP 740]: https://peps.python.org/pep-0740/

[Python Package Index]: https://pypi.org

[Trusted Publishing]: https://docs.pypi.org/trusted-publishers/

[`pypa/gh-action-pypi-publish`]: https://github.com/pypa/gh-action-pypi-publish

[v1.11.0]: https://github.com/pypa/gh-action-pypi-publish/releases/tag/v1.11.0

[Are we PEP 740 yet?]: https://trailofbits.github.io/are-we-pep740-yet/
