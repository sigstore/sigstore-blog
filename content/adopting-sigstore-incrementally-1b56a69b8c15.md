+++
title = "Adopting Sigstore Incrementally"
date = "2022-08-02"
tags = ["sigstore","security","opensource","softwaresupplychain"]
draft = false
author = "Hayden Blauzvern"
type = "post"

+++

![](/images/sigstorelogo.jpeg)

Developers, package maintainers, and enterprises that would like to adopt Sigstore may already sign published artifacts. Signers may have existing procedures to securely store and use signing keys. Sigstore can be used to sign artifacts with existing self-managed, long-lived signing keys. Sigstore provides a simple user experience for signing, verification, and generating structured signature metadata for artifacts and container signatures. Sigstore also offers a community-operated, free-to-use transparency log for auditing signature generation.

Sigstore additionally has the ability to use code signing certificates with short-lived signing keys bound to OpenID Connect identities. This signing approach offers simplicity due to the lack of key management; however, this may be too drastic of a change for enterprises that have existing infrastructure for signing. This blog post outlines strategies to ease adoption of Sigstore while still using existing signing approaches.

## Signing with self-managed, long-lived keys

Developers that maintain their own signing keys but want to migrate to Sigstore can first switch to using [Cosign](https://github.com/sigstore/cosign/) to generate a signature over an artifact. Cosign supports importing an existing RSA, ECDSA, or ED25519 PEM-encoded PKCS#1 or PKCS#8 key with `cosign import-key-pair --key key.pem`, and can sign and verify with `cosign sign-blob --key cosign.key artifact-path` and `cosign verify-blob --key cosign.pub artifact-path`.

### Benefits

* Developers can get accustomed to Sigstore tooling to sign and verify artifacts.
* Sigstore tooling can be integrated into CI/CD pipelines.
* For signing containers, signature metadata is published with the OCI image in an OCI registry.

## Signing with self-managed keys with auditability

While maintaining their own signing keys, developers can increase auditability of signing events by publishing signatures to the Sigstore transparency log, [Rekor](https://github.com/sigstore/rekor). This allows developers to audit when signatures are generated for artifacts they maintain, and also monitor when their signing key is used to create a signature.

Developers can upload a signature to the transparency log during signing with `COSIGN_EXPERIMENTAL=1 cosign sign-blob --key cosign.key artifact-path`. If developers would like to use their own signing infrastructure while still publishing to a transparency log, developers can use the Rekor [CLI](https://docs.sigstore.dev/rekor/CLI) or [API](https://github.com/sigstore/rekor/blob/143e9ec36296cd27c3c1d45495dc081741584a90/openapi.yaml). To upload an artifact and cryptographically verify its inclusion in the log using the Rekor CLI:

```
rekor-cli upload --rekor_server https://rekor.sigstore.dev \
  --signature <artifact_signature> \
  --public-key <your_public_key> \
  --artifact <url_to_artifact|local_path>

rekor-cli verify --rekor_server https://rekor.sigstore.dev \
  --signature <artifact-signature> \
  --public-key <your_public_key> \
  --artifact <url_to_artifact|local_path>
```

In addition to PEM-encoded certificates and public keys, Sigstore supports uploading many different key formats, including PGP, [Minisign](https://jedisct1.github.io/minisign/), SSH, PKCS#7, and [TUF](https://theupdateframework.io/). When uploading using the Rekor CLI, specify the `--pki-format` flag. For example, to upload an artifact signed with a PGP key:


```
gpg --armor -u user@example.com --output signature.asc --detach-sig package.tar.gz

gpg --export --armor "user@example.com" > public.key

rekor-cli upload --rekor_server https://rekor.sigstore.dev \
  --signature signature.asc \
  --public-key public.key \
  --pki-format=pgp \
  --artifact package.tar.gz
```

### Benefits

* Developers begin to publish signing events for auditability.
* Artifact consumers can create a verification policy that requires a signature be published to a transparency log.

## Self-managed keys in identity-based code signing certificate with auditability

When requesting a code signing certificate from the Sigstore certificate authority [Fulcio](https://github.com/sigstore/fulcio), Fulcio binds an OpenID Connect identity to a key, allowing for a verification policy based on identity rather than a key. Developers can request a code signing certificate from Fulcio with a self-managed long-lived key, sign an artifact with Cosign, and upload the artifact signature to the transparency log.

However, artifact consumers can still fail-open with verification (allow the artifact, while logging the failure) if they do not want to take a hard dependency on Sigstore (require that Sigstore services be used for signature generation). A developer can use their self-managed key to generate a signature. A verifier can simply extract the verification key from the certificate without verification of the certificate’s signature. (Note that verification can occur offline, since inclusion in a transparency log can be verified using a [persisted signed bundle from Rekor](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md#properties) and code signing certificates can be verified with the CA root certificate. See [Cosign’s verification code](https://github.com/sigstore/cosign/blob/29361993239764ae63c3017198cc80ff5816c08f/pkg/cosign/verify.go#L756) for an example of verifying the Rekor bundle.)

Once a consumer takes a hard dependency on Sigstore, a CI/CD pipeline can move to fail-closed (forbid the artifact if verification fails).

### Benefits

* A stronger verification policy that enforces both the presence of the signature in a transparency log and the identity of the signer.
* Verification policies can be enforced fail-closed.

## Identity-based (“keyless”) signing

This final step is added for completeness. Signing is done using code signing certificates, and signatures must be published to a transparency log for verification. With identity-based signing, fail-closed is the only option, since Sigstore services must be online to retrieve code signing certificates and append entries to the transparency log. Developers will no longer need to maintain signing keys.

## Conclusion

The Sigstore tooling and infrastructure can be used as a whole or modularly. Each separate integration can help to improve the security of artifact distribution while allowing for incremental updates and verifying each step of the integration.
