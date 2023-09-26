+++
title = "Announcing sigstore-python 2.0"
date = "2023-09-29"
tags = ["sigstore", "python", "clients"]
draft = false
author = "William Woodruff (Trail of Bits), Dustin Ingram (Google)"
type = "post"
+++

We are delighted to announce the 2.0 release of [sigstore-python], a Python
client for signing and verifying Sigstore signatures!

```bash
$ python -m pip install -U sigstore
$ python -m sigstore --version
sigstore 2.0.0
```

This release has been in the works for a while, and contains a number
of significant improvements and breaking changes to both the `sigstore`
CLI and Python APIs.

We've also updated the official [`sigstore/gh-action-sigstore-python`] action
to use the latest 2.0 release. You can use this action to join the
[growing ecosystem] of projects producing Sigstore signatures through
GitHub Actions!

Read on for a summary of our biggest changes, or check out our
[CHANGELOG] for additional details!

## CLI changes

* Sigstore's [bundle format] is now preferred throughout the CLI, and is
  the default input and output format! This means that
  `sigstore sign secret.txt` and `sigstore verify identity secret.txt` will now
  generate or verify `secret.txt.sigstore`, respectively.

* `sigstore verify` is **no longer** a backwards-compatible alias for
  `sigstore verify identity`, as it was in the 1.x series. Users must now
  invoke `sigstore verify identity` or `sigstore verify github` explicitly.

* `sigstore sign` and `sigstore get-identity-token` now support the
  `--oauth-force-oob` flag, providing a CLI option for the pre-existing
  `SIGSTORE_OAUTH_FORCE_OOB` environment variable.

## API changes

Check out [our API documentation] for additional details, including
usage examples!

* sigstore-python's APIs have been significantly refactored to improve type
  hygiene. In particular, the `IdentityToken` type has been stabilized and made
  part of the public interface, replacing many sites where a raw OIDC token
  was previously passed in.

* The `Signer` API is now two different APIs: `Signer` and `SigningContext`.
  This change better reflects sigstore-python's interior lifetimes and
  allows developers to reuse an ephemeral keypair across multiple inputs,
  saving unnecessary network round-trips!

* Bundle generation is now exposed as part of the public API:
  `VerificationMaterials.to_bundle()` and `SigningResult.to_bundle()` can
  now both be used to produce an interoperable Sigstore bundle.

## Project-level changes

* Our minimum Python version is now 3.8! This keeps us
  consistent with the broader Python ecosystem, which has considered Python 3.7
  [EOL since June 2023].

* We now interact with the public trust root a little
  differently: it now assumes that the trust root contains a [trust bundle],
  rather than falling back to the deprecated individual TUF targets.
  Additionally, sigstore-python now comes with an initial baked-in
  copy of the trust bundle, to ease bootstrapping (and offline verification).

## Exciting user: CPython

We've been overjoyed to see both developers and end users join the Sigstore
ecosystem through sigstore-python!

As part of this announcement, we wanted to highlight the hard work of
[Seth Larson] to prepare the CPython release process for sigstore-python 2.0:
[Seth backfilled old signatures into the new bundle format] and updated
[the documentation on python.org]

## Up next

This 2.0 release of sigstore-python is filled with internal changes that
set us up for new public-facing features and enhancements, including
[support for Fulcio's newer claim formats],
["full" offline verification support], and [additional "plumbing" CLI routines]
for Sigstore power users.

Many thanks to everybody who contributed to the 2.0 release, with special
thanks to Alex Cameron (Trail of Bits), Maya Costantini (Red Hat),
Jussi Kukkonen (Google), Jack Leightcap (Trail of Bits), and Andrew Pan
(Trail of Bits) for their significant feature contributions!

[sigstore-python]: https://pypi.org/p/sigstore

[CHANGELOG]: https://github.com/sigstore/sigstore-python/blob/main/CHANGELOG.md

[bundle format]: https://github.com/sigstore/protobuf-specs

[EOL since June 2023]: https://www.python.org/downloads/release/python-3717/

[our API documentation]: https://sigstore.github.io/sigstore-python/sigstore.html

[trust bundle]: https://github.com/sigstore/protobuf-specs/blob/main/protos/sigstore_trustroot.proto

[support for Fulcio's newer claim formats]: https://github.com/sigstore/sigstore-python/issues/425

["full" offline verification support]: https://github.com/sigstore/sigstore-python/issues/483

[additional "plumbing" CLI routines]: https://github.com/sigstore/sigstore-python/issues/718

[`sigstore/gh-action-sigstore-python`]: https://github.com/sigstore/gh-action-sigstore-python

[growing ecosystem]: https://packaging.python.org/en/latest/guides/publishing-package-distribution-releases-using-github-actions-ci-cd-workflows/#signing-the-distribution-packages

[Seth Larson]: https://sethmlarson.dev/

[Seth backfilled old signatures into the new bundle format]: https://github.com/python/pythondotorg/issues/2300

[the documentation on python.org]: https://www.python.org/download/sigstore/
