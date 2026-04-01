+++
title = "Sigstore Signing Is Evolving: Keep Your Verifiers Up to Date"
date = "2026-02-26"
tags = ["sigstore","rekor"]
draft = false
author = "Jussi Kukkonen, Google"
type = "post"
+++

_Sigstore Public Good instance on sigstore.dev is switching to [Rekor v2](https://github.com/sigstore/rekor-tiles) as its default transparency log in May 2026. While existing signatures remain valid, older clients may fail to verify signatures created after this switch.

<!--more-->

sigstore.dev has been running the new Rekor v2 signature transparency log in parallel with the original Rekor v1 [for a few months now](https://blog.sigstore.dev/rekor-v2-ga/). The services seem to be running well and client support looks good. The next step is using Rekor v2 log entries in new Sigstore signatures by default. This post outlines the plan and addresses compatibility concerns inherent in changing signature content.

### What changes?

The v2 log is already running, we only need to change configuration to tell clients to to start using it. Doing this in a smart way requires a bit of infrastructure: Modern signing clients get the service URLs they need via a `SigningConfig` json file, delivered automatically via TUF (The Update Framework) -- this is the same mechanism that verifying clients use to get the `TrustedRoot` file with the trusted public key material.

In May 2026 the Rekor v2 URL will be added to the Public Good instance [`SigningConfig`](https://github.com/sigstore/root-signing/blob/main/targets/signing_config.v0.2.json). This will signal Sigstore clients to start creating signature bundles that contain transparency log entries from the Rekor v2 service instead of Rekor v1. Sigstore clients can still use Rekor v1 if they are still working on supporting v2 or if the user explicitly requests v1 entries.

### How Does This Impact You?

Soon, Sigstore signature bundles will contain Rekor v2 log entries. While most current Sigstore clients can handle these new bundles, older clients will fail to verify them. Here’s what you need to do to prepare.

**For Users of Sigstore CLIs (like Cosign)**

The best thing you can do is keep your client software up to date (see [compatibility list](#which-clients-are-compatible) for details). This will ensure your existing workflows continue to work without interruption. If you're using an older client, you'll still be able to verify and sign signatures with Rekor v1 entries, but you won't be able to verify signatures that use Rekor v2.

**For Sigstore Integrators**

Anyone integrating Sigstore verification should ensure they are using an implementation compatible with Rekor v2.

Integrators (e.g. package ecosystems) that manage the signing process have more control over the transition. You can choose to pin the Rekor version in your signing process to manage the switchover time. However, if you do this, you are responsible for switching to v2 shortly after the Sigstore defaults change.

**NOTE**: Integrators must still prepare for more frequent log rotation in future. Make sure your signing process does not hard code service URLs and instead uses `SigningConfig` (either via TUF or a more DIY mechanism).

### FAQ

#### My Sigstore verification started failing, what now?

While we have been successful in getting major ecosystems up to date, you may encounter environments that have not yet updated their tooling. If you think you have a valid signature but are encountering Rekor v2 related verification issues, you can manually confirm your signature is valid and then notify the owners of the verification flows that an update is required.


1. Install cosign 3.0.5+ (or another up-to-date client)
```
$ go install github.com/sigstore/cosign/v3/cmd/cosign@v3.0.5
```

2. Check if the bundle contains a Rekor v2 log entry by looking for `hashedrekord:0.0.2` or `dsse:0.0.2`
```
# for blobs
$ cat <my-bundle> | jq '.verificationMaterial.tlogEntries[0].kindVersion'
{
  "kind": "hashedrekord",
  "version": "0.0.2"
}

# for images
$ cosign download signature <my-image> | jq '.verificationMaterial.tlogEntries[0].kindVersion'
{
  "kind": "dsse",
  "version": "0.0.2"
}
```

3. [Verify](https://docs.sigstore.dev/cosign/verifying/verify/) the signature.
```
# for blobs
$ cosign verify-blob <artifact> --bundle=<bundle.sigstore.json> --certificate-identity=<identity> --certificate-oidc-issuer=<issuer>
Verified OK

# for images
$ cosign verify <image> --certificate-identity=<identity> --certificate-oidc-issuer=<issuer>
  ...
```

If you are verifying container images with older cosign clients, you may also encounter issues for signatures that use the [OCI referrers API](https://github.com/sigstore/cosign?tab=readme-ov-file#oci-artifacts). Ensure your clients are up to date to find OCI referrers signatures and handle new Sigstore signatures.

#### Why not just hardcode the log URL?

`SigningConfig` was not always part of the Sigstore design (and as a result support for it is not yet universal, especially in older client releases). It was added because dynamic discovery of service URLs allows clients to support other Sigstore instances, enables adding entries in multiple logs, and allows instance maintainers to rotate transparency log instances without service disruption. Log rotation was always planned but has been avoided so far because it would be disruptive, as the log URL is hardcoded in many places.

#### Which clients are compatible?

This is an overview of the situation at time of writing: please refer to [Sigstore Conformance](https://github.com/sigstore/sigstore-conformance/) client Conformance Report for up-to-date conformance test results and the client projects documentation for details.

Compatible clients and the first version with Rekor v2 support:
* [sigstore/cosign](https://github.com/sigstore/cosign) 3.0.5[^1]
* [sigstore/sigstore-go](https://github.com/sigstore/sigstore-go) 1.1.0
* [sigstore/sigstore-java](https://github.com/sigstore/sigstore-java) 2.0.0[^2]
* [sigstore/sigstore-js](https://github.com/sigstore/sigstore-js) 0.9
* [sigstore/sigstore-python](https://github.com/sigstore/sigstore-python) 4.0.0
* [prefix-dev/sigstore-rust](https://github.com/prefix-dev/sigstore-rust) 0.6.0

Clients known to not have Rekor v2 support at time of writing:
* [sigstore/sigstore-rs](https://github.com/sigstore/sigstore-rs)
* [sigstore/sigstore-ruby](https://github.com/sigstore/sigstore-ruby)

[^1]: Cosign supports Rekor v2 from 2.6.0 onwards but these earlier versions require additional flags: see `--use-signed-timestamps` and `--new-bundle-format`
[^2]: sigstore-java 2.0.0 still requires enabling Rekor v2 support, see [README](https://github.com/sigstore/sigstore-java)

