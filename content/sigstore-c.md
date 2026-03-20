+++
title = "Introduction sigstore-c"
author = "Zach Steindler"
date = "2026-04-01"
tags = ["sigstore", "clients", "opensource"]
type = "post"
draft = false
+++

These days, there's many interoperable Sigstore client libraries in various programming languages. But there are still a few places where you might struggle to run Sigstore, like on a Intel 8086 processor (first released in 1978), say running a version of DOS.

Until today. Introducing [sigstore-c](https://github.com/steiza/sigstore-c), which focuses on portability over features (and correctness!)

You can build sigstore-c on any system with a C89 compiler, including modern Linux environments with `gcc` or the aforementioned DOS environment with something like [Open Watcom v2](https://github.com/open-watcom/open-watcom-v2). If you don't have a native DOS environment available, you might try emulating it with [DOSBox](https://www.dosbox.com/).

![sigstore-c running in DOS showing verification](/images/sigstore-c.gif)

sigstore-c can verify Sigstore protobuf bundles that have been signed with a ECDSA P-256 key. For example, you could sign some content with Cosign:

```
$ echo "Hello, world!" > a.txt
$ cosign generate-key-pair
Enter password for private key:
Enter password for private key again:
Private key written to cosign.key
Public key written to cosign.pub
$ cosign signing-config create > empty.signingconfig.json
$ cosign sign-blob --key cosign.key --signing-config empty.signingconfig.json --bundle signature.sigstore.json a.txt
Enter password for private key:
Using payload from: a.txt
Wrote bundle to file signature.sigstore.json
```

And then verify it with sigstore-c:

```
$ ./sigstore signature.sigstore.json cosign.pub a.txt
Digest matches
...
Signature verified successfully
```

It also understands attestations, with very likely the first C89 implementation of parsing DSSE. So you can attest with Cosign:

```
$ echo "{}" > predicate.json
$ cosign attest-blob --key cosign.key --signing-config empty.signingconfig.json --bundle dsse.sigstore.json --predicate predicate.json --type example-attestation a.txt
Using payload from: a.txt
Using payload from: predicate.json
Enter password for private key:
Wrote bundle to file dsse.sigstore.json
```

And again, verifying with sigstore-c:

```
$ ./sigstore dsse.sigstore.json cosign.pub a.txt
Digest matches
...
Signature verified successfully
```

It does not support any signing, or verifying content from Fulcio, Rekor, or a timestamp authority. You might also notice that ECDSA verification can take awhile on a 10 MHz emulated 8086 processor. We would consider accepting 8086-compatible optimized assembly for ECDSA verification, although that would preclude us from offering PDP-11 compatibility.

Do reach out if you manage to run this natively. And in case it wasn't clear, and for some reason you're still reading this, sigstore-c is [not an official Sigstore client library](https://github.com/sigstore/sig-clients?tab=readme-ov-file#projects).
