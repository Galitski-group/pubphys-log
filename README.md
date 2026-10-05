# PubPhys public log

This repository is the public mirror of the PubPhys transparency log. PubPhys writes to it, but
nothing here has to be taken on trust: every file can be checked against Sigstore Rekor, Bitcoin
and keys you pin yourself. The plain-language explanation is at https://pubphys.com/docs/trust.

| Path | Content |
|---|---|
| `checkpoints/<size>.note` | Signed checkpoints of the log (C2SP signed notes, origin `pubphys.com/log/v1`) |
| `rekor/<size>/<n>.json` | The Sigstore Rekor v2 entries that logged each checkpoint's SHA-256 |
| `ots/<size>.note.ots` | OpenTimestamps proofs of the checkpoints, once they are in Bitcoin |
| `keys/<record hash>.json` | Bundles of the records that introduce or revoke PubPhys keys |
| `jwks/<record hash>.json` | Bundles of the records of ORCID's public keys, with the Wayback Machine captures that show them |
| `trust.json` | A convenience copy of the keys and Rekor shards; never the authority |
| `verifier/` | A copy of the verifier, written by PubPhys; prefer the signed release at https://github.com/Galitski-group/pubphys-verifier/releases |
| `SPEC.md` | The protocol specification, version 1 |

Nothing else is here: no personal data and no documents. Records of people, problems, solutions and
uploaded files stay on pubphys.com; the log carries only their fingerprints. Line endings are never
converted (`.gitattributes`), because checkpoints are hashed byte for byte.

## Check a record yourself

1. Download the proof of a topic or problem ("Download proof" on its page).
2. Take the recovery key fingerprint from DNS:
   `dig +short TXT _pubphys.pubphys.com` (the value after `recovery=`).
3. Run, in a clone of this repository:

```
node verifier/fetch-trust.mjs --recovery-key-id <fingerprint> --bitcoin > my-trust.json   # not trust.json: that is the file it reads
node verifier/fetch-witness.mjs --bundle <bundle.json> --trust my-trust.json --sigstore-trusted-root trusted_root.json --site https://pubphys.com > trust-witness.json
node verifier/fetch-orcid-keys.mjs --trust trust-witness.json --live > trust-full.json   # records signed with ORCID
node verifier/pubphys-verify.mjs <bundle.json> --trust trust-full.json --bitcoin
```

`fetch-trust` accepts a PubPhys key only under the rule in `verifier/TRUST.md`: its key record
verifies, it descends from the recovery key you pinned, and no key it descends from is revoked
(unless the recovery key introduced it again). The ORCID client id and the witness keys come from
`verifier/defaults.json` or your own `--orcid-client-id` and `--witness-key`, never from this
repository's `trust.json`. `fetch-witness` takes the checkpoint of the bundle's size
from this repository and accepts it only if a Rekor entry for it verifies against a Rekor shard key
taken from Sigstore's own trusted root, never from this repository: `trusted_root.json` obtained
from Sigstore through its TUF repository (for example with cosign or sigstore-python).
`--fetch-sigstore-root-unverified` downloads it from https://github.com/sigstore/root-signing
instead, without TUF verification, and says so. `--bitcoin` checks
the OpenTimestamps proofs against Bitcoin block headers from blockstream.info; pass the URL of your
own Esplora instance to avoid trusting it.

`fetch-witness` also collects the evidence for a record's inclusion promise: a Rekor checkpoint
covering a PubPhys checkpoint that includes the record, cosigned by a witness whose key is pinned.
`pubphys-verify` then reports the promise `kept`, or `not_judged` (it never reports a promise
missed: this repository is written by PubPhys and could omit an earlier checkpoint).
`fetch-orcid-keys` trusts an ORCID key only as the Wayback Machine itself shows it (its index and raw capture), and `--live` adds the keys ORCID serves now.

What a verified result means, and what it does not, is explained at https://pubphys.com/docs/trust.
