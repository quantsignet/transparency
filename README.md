# Quantsignet Transparency Log

This repository is a **public, append-only transparency log** for the Quantsignet ownership registry.

Quantsignet issues post-quantum authenticity and ownership proofs for physical goods. The registry that records ownership is operated by Quantsignet — and you should **not have to trust that operator.** This repository is how anyone, at any time, can independently check that the operator has behaved honestly.

## What is published here

Periodically, and on every change to the registry, the Quantsignet **notary** signs a **Signed Tree Head (STH)** — a compact, cryptographic fingerprint of the *entire* ownership history at that moment:

- `size` — the number of entries in the log.
- `rootHex` — the Merkle root hash summarizing all `size` entries.
- `timestampMs` — when the notary signed it.
- `proofHex` — the notary's signature over the above, verifiable under the pinned notary public key.

Each new STH is **appended** to [`sth.jsonl`](./sth.jsonl) (one JSON object per line). The notary public key is pinned in [`notary-pubkey.txt`](./notary-pubkey.txt).

See [`FORMAT.md`](./FORMAT.md) for the exact record format and verification procedure.

## Why this makes the operator honest

A Merkle tree has two properties that this log turns into public guarantees:

1. **Append-only / no rewriting.** You cannot change any past entry without changing `rootHex`. Any two STHs published here must be *consistent* — one an exact extension of the other. If they ever aren't, the operator rewrote history, and the two signed STHs are the proof.
2. **No equivocation / no split-view.** A dishonest operator's most powerful trick is to show one history to Alice and a different one to Bob. That works only as long as the two views never meet. **This public log is where they meet.** If the STH a Quantsignet app receives from the registry does not match the STH published here, the app rejects the result — and the two conflicting, notary-signed STHs are a portable, self-evidencing proof of misbehavior that anyone can check.

Because this repository lives on infrastructure Quantsignet does **not** control (GitHub), with a world-readable, timestamped commit history, the operator cannot quietly publish different heads to different people.

## How to audit

- **Verify a single STH:** check `proofHex` against the pinned notary key (see [`FORMAT.md`](./FORMAT.md)).
- **Verify append-only:** confirm each STH in [`sth.jsonl`](./sth.jsonl) is Merkle-consistent with the ones before it.
- **Detect equivocation:** compare an STH you received from a Quantsignet verifier against the STH of the same (or greater) `size` here. Different roots with no valid consistency proof = proof of a split-view.
- **Watch the history:** GitHub's own commit log is a second, independent record that this file was only ever appended to.

## Trust anchors

- **Notary public key** — pinned in [`notary-pubkey.txt`](./notary-pubkey.txt) and independently baked into every Quantsignet verifier and app. A key that doesn't match the pin is not the Quantsignet notary.
- **The physical QR** — the ultimate anchor for any individual item is the code printed on the item itself. This log anchors the *registry as a whole*.

---

*This log records tree heads only. It does not publish item contents, owner identities, or any personal data — those never leave the signed envelopes and are not needed to verify the log's integrity.*
