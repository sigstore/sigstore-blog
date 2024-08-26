+++
title = "cosign Verification of npm Provenance, GitHub Artifact Attestations, and Homebrew Provenance"
author = "Zach Steindler (GitHub)"
date = "2024-08-09"
tags = ["sigstore", "cosign", "softwaresupplychain", "security"]
type = "post"
draft = false
+++

One of the features of the [cosign v2.4.0 release](https://github.com/sigstore/cosign/releases/tag/v2.4.0) allows you to verify attestations in the [bundle format](https://github.com/sigstore/protobuf-specs/blob/main/protos/sigstore_bundle.proto) used by npm provenance, GitHub Artifact Attestations, and Homebrew provenance.

This is part of all Sigstore clients supporting the bundle format as outlined in the [community roadmap](https://github.com/sigstore/community/blob/main/ROADMAP.md#client-sdks).

We'll show how to perform that verification for each ecosystem, and explain some of the details involved. You'll notice these examples follow the same general pattern of getting an artifact to verify, getting the bundle that contains the signed attestation about that artifact, and then providing a verification policy to cosign via command line flags.

To run these examples you'll need up-to-date `curl`, `jq`, `gh`, `brew`, and (of course) `cosign`.

### npm Provenance

[npm provenance](https://docs.npmjs.com/generating-provenance-statements) connects npm packages back to their source code and build instructions. Over 16,000 unique packages have published with provenance, so to start we'll pick one and get an artifact to verify:

```
$ curl https://registry.npmjs.org/semver/-/semver-7.6.3.tgz > semver-7.6.3.tgz
```

Then we need the bundle associated with that artifact. npm returns the publish attestations along with the provenance attestation, so we have to pull out just the SLSA provenance (I promise you, I didn't get this right on the first try):

```
$ curl https://registry.npmjs.org/-/npm/v1/attestations/semver@7.6.3 | jq '.attestations[]|select(.predicateType=="https://slsa.dev/provenance/v1").bundle' > npm-provenance.sigstore.json
```

Now we're ready to verify! cosign assumes we're using the Sigstore public good instance, which is what npm provenance uses as well. We'll tell cosign we're using the new bundle format with `--new-bundle-format`, what CI/CD system we're expecting built this artifact via `--certificate-oidc-issuer`, as well as which repository we expect the artifact to come from (and in this case, also which workflow) via `--certificate-identity-regexp`:

```
$ cosign verify-blob-attestation --bundle npm-provenance.sigstore.json --new-bundle-format --certificate-oidc-issuer="https://token.actions.githubusercontent.com" --certificate-identity-regexp="^https://github.com/npm/node-semver/.github/workflows/release-integration.yml.?" semver-7.6.3.tgz
Verified OK
```

There are many checks that take place as part of this verification, but broadly speaking this ensures the signed material came from the Sigstore instance we're expecting, refers to the artifact provided, and that the included provenance has the properties that we're expecting.

### GitHub Artifact Attestations

[GitHub Artifact Attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds) came after npm provenance, and it lets you publish SLSA provenance and/or a signed SBOM for anything built in GitHub Actions. Public repositories can use Artifact Attestations for free, and also use the Sigstore public good instance.

Just like before, we start by getting the artifact we want to verify:

```
$ curl -L https://github.com/cli/cli/releases/download/v2.54.0/gh_2.54.0_linux_armv6.tar.gz > gh_2.54.0_linux_armv6.tar.gz
```

Then we need the bundle containing SLSA provenance. Usually with Artifact Attestation you would use `gh attestation verify` to easily retrieve and verify the attestation at the same time, but here we're doing this the hard way to show how it's compatible with cosign.

This release only publishes SLSA provenance, (i.e. it doesn't also publish a signed SBOM), so we can just get all the attestations with:

```
$ gh attestation download --repo cli/cli gh_2.54.0_linux_armv6.tar.gz
Wrote attestations to file sha256:cd53809273ad6011fdd98e0244c5c2276b15f3dd1294e4715627ebd4f9c6e0f1.jsonl.
```

The verification command for public repositories is very similar to npm provenance, except of course we need to update the repository (and workflow) we expect the artifact to come from:

```
$ cosign verify-blob-attestation --bundle sha256:cd53809273ad6011fdd98e0244c5c2276b15f3dd1294e4715627ebd4f9c6e0f1.jsonl --new-bundle-format --certificate-oidc-issuer="https://token.actions.githubusercontent.com" --certificate-identity-regexp="^https://github.com/cli/cli/.github/workflows/deployment.yml.?" gh_2.54.0_linux_armv6.tar.gz
Verified OK
```

Of course, if you're using a private GitHub repository things are a bit different. Instead of using the public good Sigstore instance (which includes the public transparency log Rekor, which you might not want to use for your private repository) we use a private Sigstore instance internal to GitHub. This instance has its own signing material (often called the trusted root) and instead of using the public Rekor instance it provides an independently observed time with a signed timestamp.

So first we get the trusted root information for GitHub's internal Sigstore instance (I also didn't get this right on the first try):

```
$ gh attestation trusted-root | jq '.|select(.certificateAuthorities[0].uri=="fulcio.githubapp.com")' > github-trusted-root.json
```

And then we need to modify our verification command. We'll supply the correct trusted root, and we'll also tell cosign that we're using a signed timestamp instead of Rekor's Signed Certificate Timestamp (SCT). The flag is called `--insecure-ignore-sct` because it would be insecure if we weren't independently verifying the timestamp some other way. Here's what that verification command would look like:

```
$ cosign verify-blob-attestation --trusted-root github-trusted-root.json --use-signed-timestamps --insecure-ignore-sct --bundle ...
```

Note that you need to be a GitHub Enterprise Cloud customer to use Artifact Attestations with private repositories.

### Homebrew Provenance

[Homebrew provenance](https://blog.trailofbits.com/2024/05/14/a-peek-into-build-provenance-for-homebrew/) is now in public beta and like npm provenance shows that its packages ("bottles") have been built in an isolated environment with links back to the source code and build instructions.

By now you know we start by getting the artifact we want to verify:

```
$ brew fetch jq
Already downloaded: ... 33fa6f63828d3fdfa086250cef295785aff4290b792819148ea101a71a95fc91--jq--1.7.1.arm64_sonoma.bottle.1.tar.gz
```

Under the hood Homebrew is using GitHub Artifact Attestations, so we can get the attestations the same way:

```
$ gh attestation download -o homebrew 33fa6f63828d3fdfa086250cef295785aff4290b792819148ea101a71a95fc91--jq--1.7.1.arm64_sonoma.bottle.1.tar.gz
Wrote attestations to file sha256:7d01bc414859db57e055c814daa10e9c586626381ea329862ad4300f9fee78ce.jsonl.
```

Since Homebrew provenance uses the public good instance, our verification command should by now look quite familiar:

```
$ cosign verify-blob-attestation --bundle sha256:7d01bc414859db57e055c814daa10e9c586626381ea329862ad4300f9fee78ce.jsonl --new-bundle-format --certificate-oidc-issuer="https://token.actions.githubusercontent.com" --certificate-identity="https://github.com/Homebrew/homebrew-core/.github/workflows/dispatch-rebottle.yml@refs/heads/master" 33fa6f63828d3fdfa086250cef295785aff4290b792819148ea101a71a95fc91--jq--1.7.1.arm64_sonoma.bottle.1.tar.gz
Verified OK
```

### What's Next?

As more people use Sigstore, we want to ensure the tooling remains compatible across the different users of the public good instance and also private Sigstore deployments. Many users of cosign today generate detached signatures and certificates instead of bundles, or provide certificate chains via the command line instead of using a trusted root file. We think it's much easier to deal with a single bundle file and single trusted root file, and if you'd like to make that transition but need help with the tooling please let us know!
