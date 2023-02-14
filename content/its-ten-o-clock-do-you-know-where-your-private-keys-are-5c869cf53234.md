+++
title = "It’s ten o’clock, do you know where your private keys are?"
date = "2021-08-03"
tags = ["sigstore","security","Open Source","Supply Chain"]
draft = false
author = "Asra Ali, Appu Goundan"
type = "post"

+++

Short-lived certificates are [great](https://smallstep.com/blog/passive-revocation/) — a short lifetime removes the need for complicated revocation policies and reduces an attacker’s window of opportunity. Yet using short-lived certificates in the software supply chain brings a lifetime problem: how can users trust artifacts after the certificate’s expiration? Repeatedly signing artifacts and requesting certificates is tedious. Really, distributors only need to *prove* that artifacts were signed when the certificate was valid… with timestamps! Enter [SigStore’s](https://sigstore.dev/) new, free, open-source RFC 3161 timestamping service on the transparency log [Rekor](https://github.com/sigstore/rekor)! Using short-lived certificates (from [Fulcio](https://github.com/sigstore/fulcio), for example) with trusted, publicly verifiable timestamps allows distributors to prove that their artifacts were signed at a valid time, thereby removing the need for long-term secret management or revocation policies.

RFC 3161 is a protocol that allows for assigning tamper resistant timestamps to a hashed payload. Timestamp authorities (TSA) are trusted entities that generate timestamp response (TSR) files which contain the information necessary to verify the timestamp. In our case, we can use a timestamp response over a signed artifact to prove that a short-lived certificate signed an artifact during its validity window.

![](/images/cert.png)

To use Rekor’s timestamping service, you can generate a timestamp response with a simple request to the server.

```
$ rekor-cli timestamp --artifact file.asc --out file.tsr
```

Given SigStore’s commitment to transparency, our timestamping service is also auditable. When you use the Rekor TSA, timestamps are also automatically uploaded to the transparency log (for example, [here](https://rekor.sigstore.dev/api/v1/log/entries/431e169050150a2336c52ec0abe346c6d45930887dc656c9385c033818397a89)).

Nevertheless, we want users to be able to use any TSA that they trust; this may be your own Rekor instance, the [public Rekor instance](https://github.com/sigstore/rekor#public-instance), or even an existing private or public TSA (like [freeTSA.org](https://www.freetsa.org/)). Rekor is designed to accept any valid RFC 3161 timestamp response in its transparency logs.

```
$ rekor-cli upload --type rfc3161 --artifact file.tsr
```

### Trust me, it’s (roughly) ten o’clock

Going back to validating short-lived certificates with timestamps, a user can only trust the signature validity as much as it trusts the timestamper’s clock. Turns out, that’s non-trivial: a [Chrome study](https://acmccs.github.io/papers/p1407-acerA.pdf) shows that the biggest individual cause of certificate errors for Android and Windows clients are incorrect client clocks. So how do you know that Rekor’s clock is accurate?

To secure Rekor’s system clock against authenticated servers, we can leverage Rekor’s ability to store attestations. A designated time-agent can upload (for example every hour) attestations about the current time based on a *rough time* from other trusted servers. These fence posts can attest to or correct any Rekor generated timestamps. This can be used retroactively; if a certificate validation error occurs, the [Roughtime](https://datatracker.ietf.org/doc/html/draft-roughtime-aanchal) fence posts could be used to determine whether Rekor’s clock skew was the root cause (rather than the certificate being invalid). Rekor could also use this proactively to synchronize its clock when issuing timestamps.

![](/images/cert2.png)

The Roughtime protocol aims to achieve a rough time synchronization across servers. The protocol chains timestamp responses from a list of Roughtime [servers](https://datatracker.ietf.org/doc/html/draft-roughtime-aanchal#section-10)) by incorporating hashes of previous replies into the nonces of subsequent requests. After running the Roughtime chained protocol the time agent will upload a fence post containing the average current time computed from the chain and the cryptographic proof from the servers it used to calculate this. When Rekor generates a timestamp response between the two fence posts, a user can validate that the timestamp must be between the rough times.

![](/images/cert3.png)

To this end, we’ll be adding a searchable log entry type for Roughtime chains. To prevent log spam, only the designated time-agent that Rekor trusts will be able to upload chains. If a use case requires more accurate time or a certificate validity error was found, users can check the surrounding rough time posts when validating a timestamp type in the log.

```
$ rekor-cli verify --artifact file.tsr --type rfc3161 --check-roughtimeCurrent Root Hash: ff0c8f22a58d941b435d8826d18bea45ad6f93addb823f097787e5ce8fb9d2a2
Entry Hash: aa3f749245216a438003ded156219c452e4c6323e6fb22f925ac2df57f4546cc
Entry Index: 4721
Current Tree Size: 5299Inclusion Proof: […]
Roughtime fence posts: 
2021–06–30 15:28:08 +0000 UTC (index 4712)
2021–06–30 16:30:22 +0000 UTC (index 4777)
```

### For a later time…

Using trusted timestamps is a key part of the [Zero Trust Supply Chain](https://github.com/sigstore/community/blob/main/docs/zero-trust-supply-chains.pdf) efforts. Rekor’s timestamping service can now be used to verify the time a signature was created to facilitate the use of short-lived certificates. With the addition of public Roughtime fence posts to secure Rekor’s clock, users will be able to audit and account for errors in Rekor’s system clock when validating the signing time.

Soon, we’ll also be adding support for a more modern timestamping format. While RFC 3161 timestamps are great for compliance with existing protocols, the binary format isn’t too friendly! The new signature format will be based on the [Signed Note](https://pkg.go.dev/golang.org/x/mod/sumdb/note) format. Rekor currently uses a variant of this, the [checkpoint format](https://github.com/google/trillian-examples/tree/master/formats/log), for its Signed Tree Heads.

### Fun

As a proof of concept, we timestamped the contents of this blog post above to prove we wrote it before posting on August 3rd, 2021! Check it out by verifying against Rekor’s log entry [here](https://rekor.sigstore.dev/api/v1/log/entries/f22468276f1b023c49bc983e5c598b68c4ca2c2fe5e5d3c530e069fab6715e75).

To do this, hash the blog [contents](https://gist.githubusercontent.com/asraa/7459f474dfb447db05e21026248e675e/raw/3afa39712c6ef6cfac22e8118e2ca488ea4305bb/timestamp.md) posted as a public Gist.

```
$ curl -fs0 https://gist.githubusercontent.com/asraa/4a0c380579e0adebad09eba6c94c4a52/raw/51551d487ab381799840597df6c4cffb5fa0523f/timestamp.md -o timestamp.md$ sha256sum timestamp.md
```

Now retrieve the log entry from Rekor and base64 decode the `body`. The body contains a TSR log entry with [this](https://github.com/sigstore/rekor/blob/main/pkg/types/rfc3161/v0.0.1/rfc3161_v0_0_1_schema.json) schema.

```
$ curl -fs0 https://rekor.sigstore.dev/api/v1/log/entries/f22468276f1b023c49bc983e5c598b68c4ca2c2fe5e5d3c530e069fab6715e75 -o logEntry.json$ jq '."f22468276f1b023c49bc983e5c598b68c4ca2c2fe5e5d3c530e069fab6715e75".body' logEntry.json | tr -d \" | base64 -d > body.json
```

Extracting the `tsr.content` field will give the base64-encoded RFC 3161 timestamp response. Now, use OpenSSL to inspect the timestamp response and match the “Message data” against the hash of `timestamp.md`.

```
$ jq '.spec.tsr.content' body.json | tr -d \" | base64 -d > entry.tsr$ openssl ts -reply -text -in entry.tsr
```

You can also verify against Rekor’s timestamping certificate chain! You must extract the chain from the `certchain` endpoint.

```
$ curl -fs0 https://rekor.sigstore.dev/api/v1/timestamp/certchain -o certchain.pem$ csplit -s -z -f cert- certchain.pem '/---BEGIN CERTIFICATE---/' '{1}'$ openssl ts -verify -in entry.tsr -data timestamp.md -CAfile cert-01
```