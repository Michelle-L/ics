---
ics: 15
title: Cosmos Signed Messages
stage: draft
category: misc
author: Aleksandr Bezobchuk <alex@tendermint.com>
created: 2019-03-07
modified: 2019-03-07
---

## Synopsis

Having the ability to sign messages off-chain has proven to be a fundamental aspect
of nearly any blockchain. The notion of signing messages off-chain has many
added benefits such as saving on computational costs and reducing transaction
throughput and overhead. Within the context of the Cosmos, some of the major
applications of signing such data includes, but is not limited to, providing a
cryptographic secure and verifiable means of proving validator identity and
possibly associating it with some other framework or organization. In addition,
having the ability to sign Cosmos messages with a Ledger or similar HSM device.

A standardized protocol for hashing, signing, and verifying messages that can be
implemented by the Cosmos SDK and other third-party organizations is needed.

## Specification

### Desired Properties

The Cosmos message signing standardized protocol subscribes to the following:

* Use of a secure cryptographic hash function (e.g. resistance to collision and second
pre-image attacks)
* Hash and sign over human-readable and machine-parsable messages
* Is invulnerable to chosen ciphertext attacks
* Allow for signing over structured data
* Contains a framework for deterministic and injective encoding of structured data
* Have builtin framework and support for domain separation and replay protection
* Has protection against potentially signing transactions a user did not intend to

### Technical Specification

The Cosmos message signing protocol will be parameterized over a secure
cryptographic hash function `y ← H(x)` and a public key DSA `(sk,pk) ← S`, where
`H` satisfies the desired properties such as having resistance to collision and
second pre-image attacks, as well as being
[deterministic](https://en.wikipedia.org/wiki/Hash_function#Determinism) and
[uniform](https://en.wikipedia.org/wiki/Hash_function#Uniformity) and where
`S` contains the operations <code>sign<sub>sk</sub>(x) → z</code> and
<code>verify<sub>pk</sub>(x,z,H) → true|false</code> which provide digital
signatures over a set of bytes and verification of signatures respectively.

Tendermint has a well established protocol for signing messages using a canonical
JSON representation as defined [here](https://github.com/tendermint/tendermint/blob/master/types/canonical.go). With the given canonical JSON structure, the specification requires
that they include the meta fields `@chain_id` and `@type`, both of which are strings.
These meta fields are **reserved** and **must** be included. In addition, the fields
must be ordered in lexicographically ascending order.

For the purposes of signing Cosmos messages, the `@chain_id` field must correspond
to the Cosmos chain identifier. The user-agent should **refuse** signing if the
`@chain_id` field does not match the currently active chain! The `@type` field
corresponds to the type of structure the user will be signing in an application.
The protocol allows for signing valid ASCII text and application-specific objects.
In the former case, the `@type` must be `"message"` and the latter case `@type`
must be `"object"`.

Having the ability to support domain separation of messages is also be vital as
just simply encoding messages is not sufficient. For example, some applications
may produce identical messages or structures and when signed can be valid on
both applications. Thus an optional field `domain_separator` may be provided which
is intended to include data that is specific to the application. In addition,
client may provide optional replay protection data via the fields `nonce`,
`block_height`, and `timestamp`.

Finally, the JSON representation must also include a `data` field which is the
application-specific user supplied message and where the type corresponds to the
value defined by the `@type` field. This must be valid ASCII text or
an application-specific object.

Thus, we can have a canonical JSON structure for signing Cosmos messages using
the [JSON schema](http://json-schema.org/) specification:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "$id": "cosmos/signing/typeData/schema",
  "title": "The Cosmos signed message typed data schema.",
  "type": "object",
  "properties": {
    "@chain_id": {
      "type": "string",
      "description": "The corresponding Cosmos chain identifier.",
      "minLength": 1
    },
    "@type": {
      "type": "string",
      "description": "The message type.",
      "enum": [
        "message",
        "object"
      ]
    },
    "data": {
      "type": ["string", "object"],
      "description": "The application message.",
      "pattern": "^[\\x20-\\x7E]+$",
      "minLength": 1
    },
    "domain_separator": {
      "type": "string",
      "description": "The application unique domain separator.",
      "pattern": "^[\\x20-\\x7E]+$",
      "minLength": 1
    },
    "nonce": {
      "type": "integer",
      "description": "The account nonce.",
      "minimum": 0
    },
    "block_height": {
      "type": "integer",
      "description": "The chain block height.",
      "minimum": 0
    },
    "timestamp": {
      "type": "integer",
      "minimum": 0
    }
  },
  "required": [
    "@chain_id",
    "@type",
    "data"
  ]
}
```

We can formally specify the Cosmos message signing protocol as follows.
Given a message `m` that adheres to the JSON schema defined and `M`, the set of
all possible valid messages: <code>∀m ∈ M, z ← sign<sub>sk</sub>(H(m))</code>.

## History

2019-03-07: Initial ICS 1 draft finished and submitted as a PR

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).