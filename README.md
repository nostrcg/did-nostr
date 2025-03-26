# did-nostr: Nostr DID Method Specification

This repository contains the unofficial draft specification for the Nostr Decentralized Identifier (DID) method.

## Overview

The Nostr DID method (`did:nostr`) enables decentralized identifiers on the [Nostr](https://nostr.com/) network. This specification defines how to:

- Create a DID based on a Nostr public key
- Resolve a Nostr DID to its DID Document
- Use the DID with Nostr relays
- Security and privacy considerations for this method

## DID Format

Nostr DIDs follow this format:

```
did:nostr:<64-character-lowercase-public-key>
```

Example:

```
did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2
```

## Example DID Document

When a Nostr DID is resolved, it produces a DID Document like this:

```json
{
  "@context": ["https://w3id.org/did", "https://w3id.org/nostr/context"],
  "id": "did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2",
  "publicKey": [
    {
      "id": "did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2#key1",
      "controller": "did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2",
      "type": "SchnorrVerification2023"
    }
  ],
  "authentication": ["#key1"],
  "assertionMethod": ["#key1"],
  "service": [
    {
      "id": "did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2#relay1",
      "type": "Relay",
      "serviceEndpoint": "wss://relay.example.org/"
    }
  ]
}
```

This document defines:

- The DID identifier
- A public key that can be used for authentication and assertions
- A Nostr relay service endpoint associated with this DID

## Key Features

- **Schnorr Signatures**: Uses the same cryptographic signature scheme as Nostr
- **Relay Declaration**: Service fields allow declaring associated Nostr relays
- **Simple Creation**: Generate a key pair and encode the public key as a 64-character string
- **Deterministic Resolution**: DID documents can be deterministically generated from the public key

## Specification Status

This is an unofficial draft specification under development. It is being created within the W3C Nostr Community Group.

## Relationship with Nostr npubs

In the Nostr ecosystem, public keys are often displayed as "npubs" (e.g., `npub1...`), which are Bech32 encoded versions of the raw public keys for human-friendly display. It's important to note:

- Nostr DIDs use the raw 64-character hexadecimal public key, not the npub format
- npubs are for display purposes only and improve readability in user interfaces
- For display in user interfaces, applications may choose to show the corresponding npub alongside the DID

Example mapping:

- Raw public key: `124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`
- DID: `did:nostr:124c0fa99407182ece5a24fad9b7f6674902fc422843d3128d38a0afbee0fdd2`
- Display npub: `npub1cpxejnc58zpcuyh0pt8gvkzpv34qxceu0sqp7jec2nk9nut7p5zs4zyx4c`

## License

MIT License

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Resources

View the full specification by opening the [index.html](index.html) file in your browser.
