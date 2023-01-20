+++
title = "Announcing the 1.0 release of sigstore-python"
date = "2023-01-14"
tags = ["sigstore","python"]
draft = false
author = "di_codes"
type = "post"
+++

The sigstore-python project [began 1 year ago in January 2022](https://github.com/sigstore/sigstore-python/commit/eee7535484e5b200c727363cda48ec9b227ba55b) with the goal of providing a Sigstore-compatible client similar to cosign, but built entirely with Python and easily adoptable by the Python ecosystem.

Today, thanks to the support of the community and work of [18 unique contributors](https://github.com/sigstore/sigstore-python/graphs/contributors), we’re excited to announce [a usable and reference-quality 1.0 stable release of that client](https://pypi.org/project/sigstore/1.0.0/), which includes an importable Python API as well as a fully functional CLI. This means that users can now reliably and easily integrate with Sigstore via the sigstore-python client from Python.

The sigstore Python client is not just for signing Python things! It’s a fully featured, general-use way to interact with the public good instance of Sigstore (or other instances) and has the following features:

- Signing and verifying arbitrary files and blobs with any Sigstore-supported identity
- Ambient identity detection for GitHub Actions and Google Cloud Platform environments
- Easy installation from PyPI

The client is already in use by some early adopters, for example, the latest releases of CPython itself are signed with it, [and it can be used for verification of CPython releases as well](https://www.python.org/download/sigstore/).

The release of a fully native and idiomatic Python client is the first step towards bringing Sigstore to the Python community. We have much additional work to do to make integrity via Sigstore widely available to all Python users.

Head over to [the Trail of Bits blog](https://blog.trailofbits.com/2023/01/13/sigstore-python/) to learn more about the library and their work on the 1.0 release. You can also [install from PyPI](https://pypi.org/project/sigstore/), [read the API documentation](https://sigstore.github.io/sigstore-python/), [star us on GitHub](https://github.com/sigstore/sigstore-python), or [find us in the #python channel of the Sigstore slack](https://app.slack.com/client/T01CP44M5K9/C024FPJKC6L/). We look forward to hearing from you!