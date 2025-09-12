+++
title = "Rekor v2 Beta - Cheaper to run, simpler to maintain"
date = "2025-09-12"
tags = ["sigstore","rekor","transparency"]
draft = false
author = "Hayden Blauzvern, Google"
type = "post"
+++

We are excited to announce the beta release of Rekor v2!

[Rekor v2](https://github.com/sigstore/rekor-tiles) is a redesigned and modernized
[Rekor](https://github.com/sigstore/rekor), Sigstore's signature transparency log,
transitioning its backend to a modern, tile-backed transparency log implementation to simplify
maintenance and lower operational costs. Learn more about Rekor v2 in our
[previous blog post announcing alpha](https://blog.sigstore.dev/rekor-v2-alpha/).

With the beta release, we have added support for Rekor v2 upload and verification to the
[Go](https://github.com/sigstore/sigstore-go), [Python](https://github.com/sigstore/sigstore-python),
and [Java](https://github.com/sigstore/sigstore-java) clients, and released a new version of
[Cosign](https://github.com/sigstore/cosign/releases/tag/v2.6.0) that supports Rekor v2 upload and verification.
We have also released a new version of the
[conformance test suite](https://github.com/sigstore/sigstore-conformance/releases/tag/v0.0.20) for client
developers to test support for Rekor v2.

To see Rekor v2 in action, follow this example, which will initialize Cosign to use the public instance's staging environment
where we have deployed Rekor v2. Be sure to update to the latest Cosign version v2.6.0 or greater!

```
# Trust the staging instance
curl -LO https://raw.githubusercontent.com/sigstore/root-signing-staging/refs/heads/main/metadata/root_history/1.root.json
cosign initialize --mirror="https://tuf-repo-cdn.sigstage.dev" --root=1.root.json

# Sign an artifact, producing a bundle. The Rekor v2 URL is retrieved using a TUF-distributed SigningConfig, which is a list of service URLs
cosign sign-blob --use-signing-config --bundle sigstore.json --yes README.md

# Inspect the Rekor v2 response and the uploaded entry
cat sigstore.json| jq -r ".verificationMaterial.tlogEntries[0]"
cat sigstore.json| jq -r ".verificationMaterial.tlogEntries[0].canonicalizedBody" | base64 -d | jq .

# Verify the bundle. Replace $EMAIL and $ISSUER with your own identity
cosign verify-blob --new-bundle-format --bundle sigstore.json --certificate-identity $EMAIL --certificate-oidc-issuer $ISSUER --use-signed-timestamps README.md

# To revert back to only trusting the production public instance
rm -r ~/.sigstore
cosign initialize
```

To learn more about the client changes, read our
[client documentation](https://github.com/sigstore/rekor-tiles/blob/main/CLIENTS.md).

Look forward to a GA launch later this year!
If you have any questions, please reach out on Slack on the `#rekor` channel.
