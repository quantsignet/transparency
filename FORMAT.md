# Format & verification

## `sth.jsonl`

Append-only. One JSON object per line, in publication order. Newlines are `\n`. Lines are never edited or removed — only appended.

Each line:

```json
{
  "size": 3,
  "rootHex": "67994518…",
  "timestampMs": "1786990571301",
  "proofHex": "c4a99d6a…",
  "publishedMs": "1786990600000"
}
```

| Field | Type | Meaning |
|---|---|---|
| `size` | integer | Number of leaves in the ownership log at this head. Monotonically non-decreasing down the file. |
| `rootHex` | hex | Merkle root over all `size` leaves. |
| `timestampMs` | string(int) | Unix ms when the **notary** signed this STH. |
| `proofHex` | hex | The notary's signature (chameleon-hash collision) over the STH preimage. |
| `publishedMs` | string(int) | Unix ms when this line was appended here. Convenience only — not covered by `proofHex`. |

Integers that may exceed 2^53 are encoded as strings.

## The signed preimage

`proofHex` covers the domain-separated STH preimage, big-endian throughout:

```
preimage = "AFCDQ_STH_V1" || u64(size) || rootHex(32 bytes) || u64(timestampMs)
```

`publishedMs` is **not** part of the preimage (it's added by this repository, not the notary).

## Verifying a single STH

Using the Quantsignet verifier core (`afcdq-core`) and the pinned notary public key `pk` in `notary-pubkey.txt`:

```
notaryGCH = SHA3-512("AFCDQ_GENESIS_V1" || SHA-256(pk))        // = notary-gch.txt
accept iff  ChamHash(pk, preimage(size, rootHex, timestampMs), proofHex) == notaryGCH
```

i.e. the STH is valid exactly when the chameleon hash of its preimage under the notary key recovers the notary's genesis hash. Any other result → not a genuine Quantsignet STH.

## Verifying append-only (consistency)

For any two lines `A` (size `m`) and `B` (size `n ≥ m`), a **Merkle consistency proof** must exist showing the tree at `A` is an exact prefix of the tree at `B`. The registry serves consistency proofs; an auditor can also recompute them from the raw leaves. If no valid consistency proof exists between two published STHs, the log was rewritten.

## Detecting equivocation

Take an STH `X` that a Quantsignet verifier received from the registry, and the line `Y` in this file with `Y.size == X.size` (or the nearest `≥`):

- If `X.rootHex == Y.rootHex` → same history. Good.
- If they differ **and** no consistency proof reconciles them → the operator showed two different histories. The pair `{X, Y}` — two valid notary signatures over contradictory heads — is a self-evidencing, portable proof of equivocation. Anyone can verify it with the pinned key alone.

## Mirrors

To reduce reliance on any single host, STHs may also be mirrored to additional public media (a second git host, IPFS, and/or a periodic on-chain anchor of `rootHex`). A verifier that finds disagreement between mirrors treats it as a warning.
