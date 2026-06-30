# Parity and Key Representation in `did:nostr`

> The identifier is parity-agnostic; `did:nostr` emits `0x02`; decoders should accept `0x03` but will not see it from `did:nostr`.

The relationship between the x-only `did:nostr` identifier and the parity byte in the
Multikey verification method is a recurring source of implementer confusion — decoders
that reject odd-parity keys, or implementations that assume they must round-trip
arbitrary parity. This note states the model in one place. It is **informative** and
complements the normative text in the [specification](https://nostrcg.github.io/did-nostr/)
(*Multikey Verification Method*).

## Three layers

**1. Identifier — parity-agnostic.**
A `did:nostr` identifier is `did:nostr:<64-hex>`, the x-only BIP-340 public key (the
32-byte x-coordinate). There is no `0x02`/`0x03` at this layer, and BIP-340 parsing of the
64-hex key is unaffected by parity.

**2. Canonical Multikey — even by construction.**
The verification method's `publicKeyMultibase` is built as:

```
f      (multibase, base16-lower)
e701   (multicodec, secp256k1-pub)
02     (parity prefix)
<32-byte X>
```

A conformant `did:nostr` document always emits `0x02` — the even-y lift, per BIP-340 and
the CCG verifier-side prepend-`02` agreement ([w3c-ccg/community#254](https://github.com/w3c-ccg/community/issues/254)).

Example, for `x = 124c0f…fdd2`:

- Identifier: `did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`
- `publicKeyMultibase`: `fe70102124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`

**3. Decoders — tolerate `0x03`, but won't see it here.**
General secp256k1 / Multikey decoders may encounter `0x03` in the wild — tweaked or derived
keys, or keys from other sources — and SHOULD accept it. But a conformant `did:nostr`
document never *emits* `0x03`; its canonical form is unambiguous `0x02`.

## Why this is worth stating separately

The `0x02`/`0x03` question reads like an identifier ambiguity, but it is not — it lives
entirely at the Multikey layer. Conflating the three layers is exactly what produces
"my decoder rejected the key" bugs. For downstream consumers (CID, Data Integrity, anything
reading the verification method), `did:nostr`'s canonical form is unambiguous `0x02`.

## References

- [did:nostr specification](https://nostrcg.github.io/did-nostr/) — *Multikey Verification Method*
- [w3c-ccg/community#254](https://github.com/w3c-ccg/community/issues/254) — Data Integrity BIP-340 cryptosuite (prepend-`0x02` agreement)
- [BIP-340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) — Schnorr signatures (x-only keys, even-y convention)
