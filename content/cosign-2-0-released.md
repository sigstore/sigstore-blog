+++
title = "Cosign 2.0 Released!"
date = "2023-02-23"
tags = ["sigstore","cosign","security","opensource"]
draft = false
author = "Hayden Blauzvern"
type = "post"
+++

Cosign 2.0 has arrived! Cosign 2.0 follows [Sigstore's General Availability launch](https://blog.sigstore.dev/sigstore-ga-ddd6ba67894d), which offers production grade stable services for artifact signing and verification.

Cosign's most significant change is to no longer require `COSIGN_EXPERIMENTAL=1`, since the Sigstore services are now stable! By default, Cosign will fetch an identity-based certificate from Fulcio when a signing key is not provided, and upload the signature and signing key to Rekor to provide transparency.

The following is the list of breaking changes:

* `COSIGN_EXPERIMENTAL=1` is no longer required to have identity-based ("keyless") signing and transparency.
* By default, artifact signatures will be uploaded to Rekor, for both key-based and identity-based signing. To not upload to Rekor, include `--tlog-upload=false`.
    * You must also include `--insecure-ignore-tlog=true` when verifying an artifact that was not uploaded to Rekor.
    * Examples of when you may want to skip uploading to the transparency log are if you have a private Sigstore deployment that does not use transparency or a private artifact.
    * We strongly encourage all other use-cases to upload artifact signatures to Rekor. Transparency is a critical component of supply chain security, to allow artifact maintainers and consumers to monitor a public log for their artifacts and signing identities.
* Verification now requires identity flags, `--certificate-identity` and `--certificate-oidc-issuer`. Like verifying a signature with a public key, it's critical to specify who you trust to generate a signature for identity-based signing. See https://github.com/sigstore/cosign/issues/2056 for more discussion on this change.
* `--certificate-email` has been removed. Use `--certificate-identity`, which supports not only email verification but also any identity specified in a certificate, including SPIFFE, GitHub Actions, or service account identities. 
* Cosign no longer supports providing a certificate that does not conform to the Fulcio certificate profile, which includes setting the `SubjectAlternativeName` and OIDC Issuer OID. To verify with a non-conformant certificate, extract the public key from the certificate and verify with `cosign verify --key <key.pem>`.  We are actively working on more support for custom certificates for those who want to bring their existing PKI.
* Signing OCI images by tag prints a warning and is strongly discouraged, e.g. `cosign sign container.registry.io/foo:tag`. This is considered insecure since tags are mutable. If you want to specify a particular image, you are recommended to do so by digest.
* SCT verification, a proof of inclusion in a certificate transparency log, is now on by default for verifying Fulcio certificates. For private deployments without certificate transparency, use `--insecure-ignore-sct=true` to skip this check.
* DSSE support in `verify-blob` has been removed. You can now verify attestations using `verify-blob-attestation`.
* Environment variable `SIGSTORE_TRUST_REKOR_API_PUBLIC_KEY` has been removed. For private deployments, if you would like to set the Rekor public key to verify transparency log entries, use either a [TUF setup](https://blog.sigstore.dev/sigstore-bring-your-own-stuf-with-tuf-40febfd2badd) or set `SIGSTORE_REKOR_PUBLIC_KEY` with the PEM of the custom Rekor public key..
* `verify-blob` no longer searches for a certificate. You must provide one with either `--certificate` or `--bundle`.
* `cosign attest --type {custom|vuln}` (and `cosign verify-attestation`) will now use the RFC 3986 compliant URIs, adding `https://`, so that these predicate types are compliant with the in-toto specification.
* The `CosignPredicate` envelope that wraps the predicates of SPDX and CycloneDX attestations has been removed, which was a violation of the schema specified via the `predicateType` field ([more information](https://github.com/sigstore/cosign/pull/2718)).

Additionally, we've made the following improvements:

* Blob attestation and verification is now supported with `cosign attest-blob` and `cosign verify-blob-attestation`.
* You can now set flags via environment variables, for example instead of `--certificate-identity=email`, you can set an environment variable for `COSIGN_CERTIFICATE_IDENTITY=email`.
* `--offline=true` removes the fallback to the Rekor log when verifying an artifact. Previously, if you did not provide a bundle (a persisted response from Rekor), Cosign would fallback to querying Rekor. You can now skip this fallback for offline environments. Note that if the bundle fails to verify, Cosign will not fallback and will fail early.
* A Fulcio certificate can now be issued for self-managed keys by providing `--issue-certificate=true` with a key, `--key`, or security key, `--sk`. This is useful when [adopting Sigstore incrementally](https://blog.sigstore.dev/adopting-sigstore-incrementally-1b56a69b8c15/).
* Experimental support for trusted timestamping has been added. Timestamping leverages a third party to provide the timestamp that will be used to verify short-lived Fulcio certificates, which distributes trust. We will be writing more about this in an upcoming blog post!
   * To use a timestamp when signing a container, use `cosign sign --timestamp-server-url=<url> <container>`, such as `https://freetsa.org/tsr`, and to verify, `cosign verify --timestamp-certificate-chain=<path-to-PEM-encodeded-chain> <other flags> <artifact>`.
   * To use a timestamp when signing a blob, use `cosign sign-blob --timestamp-server-url=<url> --rfc3161-timestamp=<output-path> --bundle=<output-path> <blob>`, and to verify, `cosign verify-blob --rfc3161-timestamp=<output-path> --timestamp-certificate-chain=<path-to-PEM-encoded-chain> --bundle=<output-path> <other flags> <blob>`.

There are also numerous improvements to the Cosign API. One improvement is to the [`CheckOpts`](https://github.com/sigstore/cosign/blob/d55ed969c0c9a5ffcb46d509cab6629adea31609/pkg/cosign/verify.go#L73) structure. We have removed the requirement for TUF providing trusted keys, as now you can provide the set of trusted keys for Rekor, Fulcio, and the CT log to the respective fields in `CheckOpts`. The API for retrieving the trusted key for Rekor (`GetRekorPubs`) now only takes the context, and removes the option for fetching the Rekor public key directly from the API. This function will still respect the `SIGSTORE_REKOR_PUBLIC_KEY` custom environment variable.

See https://github.com/sigstore/cosign/compare/v1.13.1...main for all code changes.

Thank you to Priya Wadhwa, Asra Ali, Hector Fernandez, Hayden Blauzvern, and Zachary Newman for their contributions to Cosign 2.0, and thank you to the entire community for your continued support and feedback! If you have any questions about Cosign 2.0, come chat with us on [Slack](https://sigstore.slack.com/archives/C01PZKDL4DP) or post an issue on [GitHub](https://github.com/sigstore/cosign/issues).
