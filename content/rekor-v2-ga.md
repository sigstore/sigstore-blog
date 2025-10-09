+++
title = "Rekor v2 GA - Cheaper to run, simpler to maintain"
date = "2025-10-09"
tags = ["sigstore","rekor","transparency"]
draft = false
author = "Hayden Blauzvern, Google"
type = "post"
+++

We are very excited to announce the General Availability of Rekor v2!

[Rekor v2](https://github.com/sigstore/rekor-tiles) is a redesigned and modernized
[Rekor](https://github.com/sigstore/rekor), Sigstore's signature transparency log,
transitioning its backend to a
[tile-backed transparency log implementation](https://github.com/transparency-dev/tessera) to simplify
maintenance and lower operational costs. Learn more about Rekor v2 in our
[previous blog post announcing Alpha](https://blog.sigstore.dev/rekor-v2-alpha/).

We have added support for Rekor v2 upload and verification to
[Cosign v2.6.0](https://github.com/sigstore/cosign/releases/tag/v2.6.0), along with the 
[Go](https://github.com/sigstore/sigstore-go), [Python](https://github.com/sigstore/sigstore-python),
and [Java](https://github.com/sigstore/sigstore-java) clients.
We have also released a new version of the
[conformance test suite](https://github.com/sigstore/sigstore-conformance) for client
developers to test support for Rekor v2.

We've begun to roll out the 2025 Rekor v2 log public key via TUF so that upgraded clients
will automatically verify Rekor v2 entries. However, users signing artifacts will not
automatically switch to Rekor v2 yet. We will not distribute the Rekor v2 URL via TUF
until users have had adequate time to upgrade their verification clients. We will
publish a blog post in a couple of months when we distribute the new log URL.

If you would like to start using Rekor v2 today, you'll need to provide a
[SigningConfig](https://github.com/sigstore/protobuf-specs/blob/8b671553427e6415ad4777683c45c1b0f25b6ee8/protos/sigstore_trustroot.proto#L185)
that includes the 2025 Rekor v2 URL. Make a copy of the
[TUF-distributed production SigningConfig](https://github.com/sigstore/root-signing/blob/main/targets/signing_config.v0.2.json),
and change `rekorTlogUrls` to the following:

```
  "rekorTlogUrls": [
    {
      "url": "https://log2025-1.rekor.sigstore.dev",
      "majorApiVersion": 2,
      "validFor": {
        "start": "2025-10-06T00:00:00Z"
      },
      "operator": "sigstore.dev"
    },
    {
      "url": "https://rekor.sigstore.dev",
      "majorApiVersion": 1,
      "validFor": {
        "start": "2021-01-12T11:53:27.000Z"
      },
      "operator": "sigstore.dev"
    }
  ],
```

Save this file named `rekor_v2_signing_config.json`.

**Note**: We will eventually turn down this 2025 Rekor v2 instance when we deploy a 2026 instance. We strongly
advise against hardcoding this URL into any pipelines that cannot be easily updated.

Upgrade to the latest Cosign version v3.0.1+ or v2.6.0+:

```
# Sign an artifact, producing a bundle. The Rekor v2 URL is provided in the signing config. The trusted root to verify the signature will be fetched via TUF. 
cosign sign-blob --signing-config rekor_v2_signing_config.json --bundle sigstore.json --yes README.md

# Inspect the Rekor v2 response and the uploaded entry
cat sigstore.json| jq -r ".verificationMaterial.tlogEntries[0]"
cat sigstore.json| jq -r ".verificationMaterial.tlogEntries[0].canonicalizedBody" | base64 -d | jq .

# Verify the bundle. Replace $EMAIL and $ISSUER with your own identity. The trusted root to verify the signature will be fetched via TUF. 
cosign verify-blob --bundle sigstore.json --certificate-identity $EMAIL --certificate-oidc-issuer $ISSUER --use-signed-timestamps README.md
```

The [bundles](https://github.com/sigstore/protobuf-specs/blob/main/protos/sigstore_bundle.proto)
Cosign produces are verifiable by the latest Go, Java and Python clients as well. Support for JavaScript and Ruby will follow.

If you have any questions or feedback, please reach out on [Slack](https://www.sigstore.dev/community) on the
`#rekor` or `#clients` channels, and as you upgrade clients, please report any issues on
[GitHub](https://github.com/sigstore/rekor-tiles/issues).

## FAQ

### What's happening with Rekor v1?

Rekor v1 will continue to run in parallel with Rekor v2. Clients using the publicly distributed SigningConfig and TrustedRoot files
will automatically and seamlessly transition to using Rekor v2 and verifying entries.

We will eventually freeze Rekor v1, disallowing new entry uploads. This will be announced one year in advance as per our deprecation
guidelines.  Regardless, we strongly encourage users to upgrade to the latest clients as soon as possible.

### What's the new URL for the log?

Previously, Rekor was accessible via a single URL `rekor.sigstore.dev`. This URL abstracted
log shards, where a new tree is created to upload entries. This added complexity for log operation.

Going forward, each log shard will have a unique URL, e.g. `logYEAR-rev.rekor.sigstore.dev`, which matches
how Certificate Transparency handles log deployments.
Log shard URLs will be distributed via Sigstore's TUF repository. You can find the latest URL
by inspecting the [SigningConfig](https://github.com/sigstore/root-signing/blob/main/targets/signing_config.v0.2.json).

If you're building a client, you MUST NOT hardcode this URL. We will periodically rotate the log instance, and while
we'll publicly communicate when we spin up a new log, we will also freeze the previous log instance shortly afterwards.

### How do I upload entries to the public instance?

The easiest way is to use an updated Sigstore client or SDK.

If you want to experiment with the API, we've provided an
[example](https://github.com/sigstore/rekor-tiles/blob/main/CLIENTS.md#rekor-v2-the-bash-way). Note
that Rekor v2 exposes only a single endpoint, `/api/v2/log/entries`.

### How do I read entries from the public instance?

If you're monitoring the log, the easiest way is to use [rekor-monitor](https://github.com/sigstore/rekor-monitor).

Read requests are served by a standardized [read API](https://github.com/C2SP/C2SP/blob/main/tlog-tiles.md#merkle-tree)
that serves tiles and entries. Note that these APIs are significantly different than Rekor v1, namely
there are no Get-By-Log-Index or Get-By-Leaf-Hash APIs. The Sigstore clients handle the logic to fetch the set of tiles
needed to compute an inclusion proof given a log index.

Unless you're monitoring the log for entries, we discourage using the read API for computing inclusion proofs. The log
will return inclusion proofs on upload and these should be persisted alongside artifacts and their signatures.

### What are the benefits of Rekor v2 over v1 as a user?

Rekor v2 is more easily scalable so the public instance will support a higher QPS.

Rekor v2 also will provide stronger security guarantees that the log remains append-only
by integrating [witnessing](https://blog.transparency.dev/can-i-get-a-witness-network) directly into Rekor.
This will be implemented soon - reach out if you have more questions on this topic.

### What are the benefits of Rekor v2 over v1 as a private operator?

You will see both a reduction in storage costs from the [tile-based backend](https://transparency.dev/articles/tile-based-logs/)
and infrastructure cost as you can turn down Trillian log server and log signer instances. You will also see a reduction
in egress costs for any read requests since they're fully cacheable and can be served by a CDN.

### What's been removed between v1 and v2?

The log no longer returns signed timestamps with proofs. Sigstore clients will fetch
a signed timestamp from a dedicated service. We've deployed a public instance timestamp authority,
and users can configure their signing workflows to use other public or private timestamp authorities.

The search index has been removed, since this was a best-effort service. This will be
implemented as a dedicated service backed by a
[verifiable index](https://github.com/transparency-dev/incubator/tree/main/vindex)
to simplify monitoring. Reach out if you need specific indexes besides signing identity.

While this was already deprecated in v1, there is no attestation storage in v2. Users should persist
attestations alongside artifacts. Many package registries now support Sigstore-signed attestations.

Due to a lack of usage and to simplify the API, many of the entry types have been removed. The
artifact `hashedrekord` entry type and the attestation `dsse` entry type are the two supported
types in Rekor v2. `intoto`, `rekord`, `helm`, `tuf`, `rfc3161`, `jar`, `rpm`, `cose` and `alpine`
have been removed.

### Is there any reason to not use Rekor v2?

Nope! The transition should be seamless and invisible to users.

The only transition concern to be aware of is if you are producing signatures with an updated clients and verifying signatures
with an older client. In that case, make sure to update your verification pipeline before transitioning your signing pipeline.

### Why does Rekor v2 take a few seconds to return a response?

Rekor v2 batches requests, which enables the higher QPS and witnessing. We strongly believe this tradeoff
of waiting a few seconds for log integration for stronger security guarantees and reliability outweighs
the latency increase. When signing multiple artifacts, clients can parallelize generating signatures
to minimize the latency increase.

### How do I monitor Rekor v2 entries?

[rekor-monitor](https://github.com/sigstore/rekor-monitor) has been upgraded to support Rekor v2. Thank you
to the OpenSSF for funding this work and Trail of Bits for implementation!

### What should private operators do?

If you use GCP, begin transitioning over to Rekor v2 to reduce infrastructure costs and simplify maintenance.

We recommended using the
[SigningConfig](https://github.com/sigstore/protobuf-specs/blob/d4937b379b0c462b5ebe0be7c196e1b653027bb1/protos/sigstore_trustroot.proto#L185)
to distribute service URLs for signing. You can specify both a Rekor v1 and v2 service URL for a seamless transition.
We also recommend using a
[TrustedRoot](https://github.com/sigstore/protobuf-specs/blob/d4937b379b0c462b5ebe0be7c196e1b653027bb1/protos/sigstore_trustroot.proto#L155)
to distribute the log's public key for verification.

If you do not use GCP, we are actively working on adding support for other cloud providers. We depend on [Tessera's storage backends](https://github.com/transparency-dev/tessera),
which currently support [GCP](https://github.com/transparency-dev/tessera/tree/main/storage/gcp),
[AWS](https://github.com/transparency-dev/tessera/tree/main/storage/aws), [MySQL](https://github.com/transparency-dev/tessera/tree/main/storage/mysql),
and a [POSIX filesystem](https://github.com/transparency-dev/tessera/tree/main/storage/mysql).

### Where can I learn more about the technical details?

Join the [sigstore-dev Google group](https://groups.google.com/g/sigstore-dev) to access the design documents.

* [PRD](https://docs.google.com/document/d/1Mi9OhzrucIyt-UCLk_FxO2_xSQZW9ow9U3Lv0ZB_PpM/edit?resourcekey=0-4rPbZPyCS7QDj26Hk0UyvA&tab=t.0#heading=h.bjitqo6lwsmn)
* [Design doc](https://docs.google.com/document/d/1ZYlt_VFB-lxbZCcTZHN-6KVDox3h7-ePp85pNpOUF1U/edit?resourcekey=0-V3WqDB22nOJfI4lTs59RVQ&tab=t.0#heading=h.xzptrog8pyxf)
* [Repo](https://github.com/sigstore/rekor-tiles)

To learn more about the client changes, read our
[client documentation](https://github.com/sigstore/rekor-tiles/blob/main/CLIENTS.md).

### Is a hot dog a sandwich?

No.
