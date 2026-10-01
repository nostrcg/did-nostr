# Parity and Key Representation in `did:nostr`

> The identifier is parity-agnostic; a resolver with only the identifier emits `0x02`; a document the controller publishes may carry `0x03`; verifiers accept both.

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

**2. Multikey — `0x02` by default, `0x03` when the controller knows better.**
The verification method's `publicKeyMultibase` is built as:

```
f      (multibase, base16-lower)
e701   (multicodec, secp256k1-pub)
02     (parity prefix)
<32-byte X>
```

A resolver that has only the identifier (minimal, offline resolution) emits `0x02` — the
even-y lift, per BIP-340 and the CCG verifier-side prepend-`02` agreement
([w3c-ccg/community#254](https://github.com/w3c-ccg/community/issues/254)). That is the
default every resolver computes from the 32 bytes alone.

A document the controller publishes (HTTP or relay resolution) MAY carry `0x03` instead,
when the controller holds the full key and its y-coordinate is odd — for example a key
derived by additive tweaking, whose parity is not predictable in advance. The parity byte is
the only place in the document that can carry the full point; the identifier cannot. The
specification's transformation step ("use `0x03` for odd y-coordinate, when applicable") and
its parity note permit this.

Example, for `x = 124c0f…fdd2`:

- Identifier: `did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`
- `publicKeyMultibase`: `fe70102124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`

**3. Verifiers and decoders — accept both.**
Verifiers and general secp256k1 / Multikey decoders MUST accept both prefixes. The
x-coordinate is the identifier either way, and a BIP-340 signature verifies against the same
x whichever prefix the document carries. The test vectors `decode_even_parity` and
`decode_odd_parity` pin this.

**Key arithmetic.** An implementation that derives keys by scalar addition (tweaking) works
on the full point. With a published document, it tweaks the point the document carries. With
only the identifier, it tweaks the `0x02` point; a holder whose secret `d` gives an odd-y point
uses `n − d` once, so that the `0x02` point is exactly theirs. Either way the published key
and the holder's arithmetic agree, with no further parity handling in the chain.

## Why this is worth stating separately

The `0x02`/`0x03` question reads like an identifier ambiguity, but it is not — it lives
entirely at the Multikey layer. Conflating the three layers is exactly what produces
"my decoder rejected the key" bugs. For downstream consumers (CID, Data Integrity, anything
reading the verification method): expect `0x02` from a resolver that had only the identifier,
accept `0x03` from a document the controller published, and never reject a key for its parity.

## References

- [did:nostr specification](https://nostrcg.github.io/did-nostr/) — *Multikey Verification Method*
- [w3c-ccg/community#254](https://github.com/w3c-ccg/community/issues/254) — Data Integrity BIP-340 cryptosuite (prepend-`0x02` agreement)
- [BIP-340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) — Schnorr signatures (x-only keys, even-y convention)
