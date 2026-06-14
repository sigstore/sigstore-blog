+++
title = "Proving What an AI Agent Did: Sigstore-Shaped Audit Trails"
author = "Aleksy Siek (nolabs.ai)"
date = "2026-06-14"
tags = ["sigstore", "in-toto", "dsse", "rekor", "transparency", "ai-agents"]
type = "post"
draft = false
+++

Most of what you see across the Sigstore landscape today — npm provenance, PyPI attestations, Homebrew bottles, Maven Central, container image signing — runs on the `public-good` stack: Fulcio, Rekor, OIDC identity binding, DSSE, and in-toto bundles. Those flows answer two familiar questions: where did this artifact come from — which source and build produced it — and whether it has been tampered with since. The signed bundle links a tarball or image back to its origin and lets you verify integrity before you install or run it.

This post is about a different but closely related question, one that comes up the moment you let an AI coding agent run unfettered on a machine: "what did the agent actually do, and can I prove it later?"

[nono](https://github.com/always-further/nono), from the team that built Sigstore, is a capability-based sandbox for running agents like Claude Code, Opencode, Codex and more. The sandbox decides what an agent is *allowed* to do; sandboxing and recording are different roles. For incident review, compliance, or just debugging a weird session, you ideally want a tamper-evident record of what happened, one whose integrity you can check after the fact, provable with a signature.

The cleanest way to build that record is by using the exact primitives standardized by the Sigstore and in-toto communities: a Merkle tree formatted like a Rekor transparency log, an in-toto attestation, a DSSE envelope, and a Sigstore bundle.

This post walks through that audit trail end-to-end using real commands and output. The core, reusable concept here is applying Sigstore's foundational building blocks to a non-package artifact: the activity log of a process.

Everything below is captured output from `nono v0.62.0` — session IDs, hashes, timestamps, and the signed bundle are all genuine; your session IDs and hashes will differ per run. You'll need `nono` v0.62.0 or later, and the Claude pack installed once:

```
$ nono pull always-further/claude
```

## Recording a session

Auditing is on by default. We add one flag, `--audit-sign-key`, to have the trusted supervisor sign the finished session.

```
$ echo "# Demo project" > README.md
$ nono trust keygen --keyref "file://$PWD/audit-key.pem"
Signing key generated successfully.
  Key ID:      file:///Users/you/Work/nono-audit-demo/audit-key.pem
  Fingerprint: b5fa5659f70ba19fa699ead75c48630d3ec4557de4147543122e3a3c68956818
  Algorithm:   ECDSA P-256 (SHA-256)
  Stored in:   file (/Users/you/Work/nono-audit-demo/audit-key.pem)
  Public key:  file (/Users/you/Work/nono-audit-demo/audit-key.pem.pub)
```

Run Claude Code from that directory with the `always-further/claude` pack and `--allow-cwd`. We also pass `--dangerously-skip-permissions`; not because we want Claude running loose, but because nono is already the sandbox. On Linux it applies `Landlock`; on macOS, `Seatbelt`. Those are kernel-enforced boundaries: once applied, the agent cannot read, write, or connect outside what the profile grants, no matter what Claude thinks it is allowed to do. Claude Code ships its own application-level permission prompts on top of that; under nono those are redundant and block headless runs like `-p`. Skipping them lets the demo run non-interactively while the OS-level sandbox stays in charge.

```
$ nono run --audit-sign-key "file://$PWD/audit-key.pem" \
    --profile always-further/claude --allow-cwd -- \
    claude --dangerously-skip-permissions -p "Add a bullet to README.md: '- [x] tidy up the README'. Do not ask questions; just edit the file."

  nono v0.62.0
  Verified 1 pack(s)
  Capabilities:
  ────────────────────────────────────────────────────
   r+w  ~/Work/nono-audit-demo (dir)
       + Claude profile paths and system paths (-v to show)
   net  outbound allowed
  ────────────────────────────────────────────────────

  Applying sandbox...

Done.

Command exited with code 0.

$ cat README.md
# Demo project

- [x] tidy up the README
```

Notice `Verified 1 pack(s)` at the top of the run output. The profile references the `always-further/claude` pack — hooks and config installed from the nono registry. Before the sandbox starts, nono checks each artifact's SHA-256 against the lockfile and re-verifies the Sigstore bundles in `.nono-trust.bundle`, pinning the signer identity recorded at `nono pull` time. Same provenance pattern as npm or PyPI on the public good instance: if the pack was tampered with or the signature does not verify, the run aborts before Claude starts.

Claude adds the bullet and exits cleanly. nono's supervisor — which lives *outside* the sandbox — recorded what it observed into an append-only `audit-events.ndjson` under `~/.nono/audit/<session-id>/`, committed the event stream, and (because we passed a key) wrote a signature.

## Verifying it later

Let's take a look at what was recorded:

```
$ nono audit list --recent 1
nono 1 command(s)

  ~/Work/nono-audit-demo (1 command)
    20260614-230519-37057  just now  completed  claude --dangerously-skip-permis...
```

```
$ nono audit show 20260614-230519-37057
nono Audit trail for session: 20260614-230519-37057 completed
  Command:  claude --dangerously-skip-permissions -p "Add a bullet to README.md: '- [x] tidy up the README'. Do not ask questions; just edit the file."
  Started:  2026-06-14T23:05:19.957141+01:00
  Ended:    2026-06-14T23:05:27.020388+01:00
  Exit:     0
  Paths:    ~/Work/nono-audit-demo, …
  Audit:    integrity enabled (2 events)
  Chain:    97fc66bfa4ab0ca5
  Root:     48b9c38f499b2826
  Signed:   b5fa5659f70ba19fa699ead75c48630d3ec4557de4147543122e3a3c68956818
```

Two values there are the crux of everything that follows:

* `Chain` — the head of a hash chain over the ordered event stream.
* `Root` — a Merkle root over all recorded event leaves.

Together they're a compact commitment to "exactly these events, in exactly this order." Change, drop, or mess up a single event and both values change. This run recorded `session_started` and `session_ended` — two events covering a ~7-second Claude invocation. Longer sessions, or runs in supervised mode where the supervisor intercepts capability requests, also record capability decisions, URL opens, and network events in the same log.

The interesting command for our case is `nono audit verify`. It re-derives the integrity values from the on-disk events, checks the session's entry in nono's local audit ledger, and verifies the signature. We pass `--public-key-file` to pin the signer to the key we generated above:

```
$ nono audit verify 20260614-230519-37057 --public-key-file audit-key.pem.pub
nono Audit integrity for session 20260614-230519-37057 VERIFIED
  Events:   2
  Chain:    97fc66bfa4ab0ca5d3bc7bbddc66d295fe4569e412df636ecc3d3648b24c6e35
  Root:     48b9c38f499b2826c3bdf46161a471932bd8f8419fb51d169322e7387ddb4031
  Scheme:   alpha
  Stored:   events=2, chain=97fc66bfa4ab0ca5d3bc7bbddc66d295fe4569e412df636ecc3d3648b24c6e35, root=48b9c38f499b2826c3bdf46161a471932bd8f8419fb51d169322e7387ddb4031
  Ledger:   verified (entries=50, head=9f92ea86ad7bb7ca53bd920f7f8cbc60bc2dc265f037dcdc98dc2982a215920e)
  Signed:   verified (key=b5fa5659f70ba19fa699ead75c48630d3ec4557de4147543122e3a3c68956818)
  Pubkey:   matched
```

That single `VERIFIED` is the AND of several independent checks, and each one maps onto a concept Sigstore users will recognize:

1. Re-computed vs. stored. nono walks `audit-events.ndjson`, rebuilds the hash chain and Merkle root, and compares against what was stored (the `Stored:` line). This is the "does the data still hash to the committed value" check.
2. Ledger inclusion. The session is looked up in an append-only, hash-chained `ledger.ndjson` that spans *all* sessions on the host (here, `entries=50`), and its session digest and the ledger's chain are verified. This is an inclusion check against a local transparency-style log.
3. Signature. The keyed DSSE signature over the session's Merkle root is verified, the signer key id is checked, and — because we passed `--public-key-file` — the signer is *pinned* to the public key we chose to trust ahead of time (`Pubkey: matched`).

If you don't pin a key, nono is candid about it: it prints `self-attested` and tells you to "rely on ledger chain, or pass `--public-key-file` to pin signer."

### Tamper detection, for real real

The whole point is that editing the record breaks verification. Flip a single hex digit in a `chain_hash` field on the first line of `~/.nono/audit/<session-id>/audit-events.ndjson` and re-verify:

```
$ nono audit verify 20260614-230519-37057 --public-key-file audit-key.pem.pub
nono: Snapshot error: Audit event chain hash mismatch at line 1
```

## The attestation is a Sigstore bundle

So what actually is the signature? nono writes it to `~/.nono/audit/<session-id>/audit-attestation.bundle` — not in your project directory — and it is an ordinary [Sigstore bundle v0.3](https://github.com/sigstore/protobuf-specs) — the same `application/vnd.dev.sigstore.bundle.v0.3+json` media type cosign outputs. Inside, it nests three standards:

1. A Sigstore bundle carrying the verification material (here, a public-key hint; in the keyless flow, a Fulcio certificate + Rekor entry).
2. A [DSSE](https://github.com/secure-systems-lab/dsse) envelope whose signature is over the Pre-Authentication Encoding (PAE) of the payload, so the signature commits to *what kind of thing* was signed.
3. An [in-toto v1 Statement](https://github.com/in-toto/attestation) binding a subject to a predicate.

Here is the actual `audit-attestation.bundle` from the run above (the long `payload` truncated for the page):

```json
{
  "mediaType": "application/vnd.dev.sigstore.bundle.v0.3+json",
  "verificationMaterial": {
    "publicKey": {
      "hint": "b5fa5659f70ba19fa699ead75c48630d3ec4557de4147543122e3a3c68956818"
    },
    "tlogEntries": [],
    "timestampVerificationData": {}
  },
  "dsseEnvelope": {
    "payloadType": "application/vnd.in-toto+json",
    "payload": "eyJfdHlwZSI6Imh0dHBzOi8vaW4tdG90by5pby9TdGF0ZW1lbnQvdjEi…",
    "signatures": [
      { "sig": "MEQCIEy+AAtRZSyzKG8lMqHYQ9QfkQd+TpKpnKJsoNlohiKSAiBKD1D45Za/+RP3x+LdGNuPARrRjA+I3JyFUYZJk9hKHQ==" }
    ]
  }
}
```

`tlogEntries` is empty because this is a keyed bundle — the long-lived key is its own anchor, so there's no Rekor entry. (In nono's keyless flow, signing from CI via GitHub Actions OIDC, that array carries the Rekor inclusion proof and `verificationMaterial` carries a Fulcio certificate instead of a public-key hint.)

If you decode that `payload` (it's base64url over the in-toto Statement), here is the actual statement — and notice the subject digest is the session's Merkle root (`48b9c38f…`, the same `Root:` from `audit show`):

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "name": "audit-session:20260614-230519-37057",
      "digest": {
        "sha256": "48b9c38f499b2826c3bdf46161a471932bd8f8419fb51d169322e7387ddb4031"
      }
    }
  ],
  "predicateType": "https://nono.sh/attestation/audit-session/alpha",
  "predicate": {
    "version": 1,
    "session_id": "20260614-230519-37057",
    "started": "2026-06-14T23:05:19.957141+01:00",
    "ended": "2026-06-14T23:05:27.020388+01:00",
    "command": [
      "claude",
      "--dangerously-skip-permissions",
      "-p",
      "Add a bullet to README.md: '- [x] tidy up the README'. Do not ask questions; just edit the file."
    ],
    "audit_log": {
      "hash_algorithm": "sha256",
      "event_count": 2,
      "chain_head": "97fc66bfa4ab0ca5d3bc7bbddc66d295fe4569e412df636ecc3d3648b24c6e35",
      "merkle_root": "48b9c38f499b2826c3bdf46161a471932bd8f8419fb51d169322e7387ddb4031"
    },
    "signer": {
      "kind": "keyed",
      "key_id": "b5fa5659f70ba19fa699ead75c48630d3ec4557de4147543122e3a3c68956818"
    }
  }
}
```

This is a nice illustration of what in-toto's subject/predicate split is for. The `subject` is the thing we're making a claim about — a session, identified by its Merkle root. The `predicate` is the claim — session metadata under our own `predicateType` URI. The `/alpha` suffix is deliberate: the predicate schema is still pre-1.0 and we expect it to evolve before we stabilize the format. A verifier checks the predicate type before it trusts the contents, exactly the way it checks the DSSE payload type. We didn't have to invent a container; we just minted a predicate type and reused the stack. (One supply-chain win: the `command` array is run through a best-effort redactor before signing, so tokens and `Authorization:` headers don't get baked into a signed artifact.)

## The tree is Rekor-shaped

Nono's audit Merkle tree follows the same *shape* as Rekor's transparency log — a binary tree built bottom-up over ordered event leaves, with domain-separated hashing so leaf and interior digests cannot be swapped. The alpha audit scheme lives in `audit.rs`:

```
// crates/nono/src/audit.rs
//
// Domain separators for the alpha audit Merkle scheme.
pub const EVENT_DOMAIN_ALPHA: &[u8] = b"nono.audit.event.alpha\n";
pub const MERKLE_NODE_DOMAIN_ALPHA: &[u8] = b"nono.audit.merkle.alpha\n";

// Event leaves are SHA-256(EVENT_DOMAIN_ALPHA || canonical_event_json);
// internal nodes are SHA-256(MERKLE_NODE_DOMAIN_ALPHA || left || right).
```

Event leaves are hashed canonical JSON with a scheme-specific prefix; internal nodes pair siblings the same way Certificate Transparency and Rekor do. If you've read those internals, the tree walk is familiar — applied to audit-event leaves instead of log entries.

And the per-host `ledger.ndjson` is the other transparency-log primitive: an append-only, hash-chained log of *every* session, guarded by a file lock, where each new session commits its digest onto the chain. `nono audit verify` checks that a session is included in that chain and that the chain itself is intact — structurally, an inclusion proof against a local log.

The attestation itself is built by the same signing path nono uses for everything else — there's no special-case audit signer, just a Merkle root dropped into an in-toto subject:

```
// crates/nono-cli/src/audit_attestation.rs
let statement = trust::new_statement(
    &format!("audit-session:{}", metadata.session_id),
    &integrity.merkle_root.to_string(), // the subject digest IS the Merkle root
    predicate,
    AUDIT_ATTESTATION_PREDICATE_TYPE_ALPHA,
);
let bundle_json = trust::sign_statement_bundle(&statement, &signer.key_pair)?;
```

The same DSSE + in-toto + bundle machinery also backs nono's other trust feature — signing the instruction files an agent reads (`CLAUDE.md`, `SKILL.md`, and others), with keyless Fulcio + Rekor + OIDC signing from CI — but that's a story for another post. If you want the internals, the [nono docs](https://docs.nono.sh) cover the audit trail and the attestation format in detail.

If you're thinking about attesting something that isn't a conventional build artifact — a session, a config, a policy, an agent's instructions — our experience is that the Sigstore stack stretches to fit far better than you might expect. The hard parts (short-lived certs, transparency-tree math, offline verification, identity binding) are solved; what's left is deciding *what you're saying* about the artifact, which is exactly what in-toto predicates are for.

Thanks to everyone in the Sigstore, in-toto, and DSSE communities whose specs and libraries this is built on. The interoperability is the whole point <3
