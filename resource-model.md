# Resource Model in `did:nostr`

> The **subject** is the identity; the **document** is one HTTP representation of it. Change-time is a property of the subject (in the graph); cache validators are properties of the representation (in HTTP headers).

`did:nostr` resolution returns a JSON-LD document, but that document is *not* the
identity — it is a representation of it. Keeping the two apart (the httpRange-14
distinction) is what makes provenance mirror-independent and is the reasoning behind
several spec decisions. This note states the model in one place. It is **informative**
and complements the normative text in the [specification](https://nostrcg.github.io/did-nostr/).

## Two resources

| Resource | What it is | Identified by |
| --- | --- | --- |
| **Subject** | the identity itself (a person, agent, service) | `did:nostr:<hex>` |
| **Document** | a JSON-LD representation returned on resolution | an HTTP URL (mirror-specific) |

The same subject can be served by many mirrors, each returning a byte-for-byte different
document (formatting, which optional parts are included, which mirror answered). They all
describe the *one* subject.

## Provenance lives on the layer it describes

| Concern | Property | Layer |
| --- | --- | --- |
| "Did the subject's signed state change?" | [`modified`](https://nostrcg.github.io/did-nostr/) (`dcterms:modified`) | **in the graph** (signed, mirror-independent) |
| "Is this exact representation current?" | `ETag`, `Last-Modified`, `Cache-Control` | **HTTP** (node-local, per-mirror) |

`modified` is **source-derived** — computed as `max(created_at)` over the signed source
events — so every mirror produces the same value for the same state. `ETag` is the hash of
the canonical document bytes.

The two are related but not equal: every change to signed subject data bumps `modified`
*and* (since `modified` is in the document) the `ETag`; but a representation-only change
(reformatting, a different mirror) bumps the `ETag` and **not** `modified`. So `ETag` is a
*strict refinement* of `modified` — exact representation version vs. advisory subject-change
hint.

## Why provenance is in the graph, not in HTTP headers

In-graph fields are part of the content-addressed, signable payload: they are covered by the
`ETag`, can be carried into a Bitcoin-anchored proof, and are identical across mirrors.
HTTP headers are out-of-band and node-local. This is why the earlier `Nostr-Timestamp`
header was removed in favour of the in-graph `modified` field — a header cannot be signed,
cannot be content-addressed, and differs per mirror.

For the same reason, `did:nostr` keeps change-time **in the document graph** rather than in
DID Core's `didDocumentMetadata`: that metadata lives in the resolution *result*, outside the
signable document, and its `created`/`updated` "operation" semantics do not map onto a method
whose document is a *projection* of signed events with no single update operation.

## Related conventions

- **`type: "DIDNostr"`** — DID Core leaves the document's root node typeless under JSON-LD;
  `did:nostr` types it explicitly so the document representation is unambiguous (httpRange-14).
- **Content-addressing** — because `modified` and the rest of the signed payload are in the
  graph and covered by the `ETag` (hash of the canonical document), a document version can be
  anchored and proven over time (future Bitcoin-anchored / blocktrails work).

## References

- [did:nostr specification](https://nostrcg.github.io/did-nostr/) — provenance and HTTP resolution
- [`parity-model.md`](./parity-model.md) — companion explainer (x-only / Multikey parity)
- [httpRange-14](https://www.w3.org/2001/tag/issues.html#httpRange-14) — TAG finding on resources vs representations
- [DCAT](https://www.w3.org/TR/vocab-dcat-3/) — `dcterms:modified` for dataset change time
