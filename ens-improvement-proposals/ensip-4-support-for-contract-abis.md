---
description: >-
  A mechanism for storing ABI definitions in PNS, for easy lookup of contract
  interfaces by callers (formerly EIP-205).
---

# PNSIP-4: Support for contract ABIs

| **Author**    | Nick Johnson \<nick@pulse.domains> |
| ------------- | -------------------------------- |
| **Status**    | Final                            |
| **Submitted** | 2017-02-06                       |

### Abstract

ABIs are important metadata required for interacting with most contracts. At present, they are typically supplied out-of-band, which adds an additional burden to interacting with contracts, particularly on a one-off basis or where the ABI may be updated over time. The small size of ABIs permits an alternative solution, storing them in PNS, permitting name lookup and ABI discovery via the same process.

ABIs are typically quite compact; the largest in-use ABI we could find, that for the DAO, is 9450 bytes uncompressed JSON, 6920 bytes uncompressed CBOR, and 1128 bytes when the JSON form is compressed with zlib. Further gains on CBOR encoding are possible using a CBOR extension that permits eliminating repeated strings, which feature extensively in ABIs. Most ABIs, however, are far shorter than this, consisting of only a few hundred bytes of uncompressed JSON.

This PNSIP defines a resolver profile for retrieving contract ABIs, as well as encoding standards for storing ABIs for different applications, allowing the user to select between different representations based on their need for compactness and other considerations such as onchain access.

### Specification

#### ABI encodings

In order to allow for different tradeoffs between onchain size and accessibility, several ABI encodings are defined. Each ABI encoding is defined by a unique constant with only a single bit set, allowing for the specification of 256 unique encodings in a single uint.

The currently recognised encodings are:

| ID | Description          |
| -- | -------------------- |
| 1  | JSON                 |
| 2  | zlib-compressed JSON |
| 4  | CBOR                 |
| 8  | URI                  |

This table may be extended in future through the PNSIP process.

Encoding type 1 specifies plaintext JSON, uncompressed; this is the standard format in which ABIs are typically encoded, but also the bulkiest, and is not easily parseable onchain.

Encoding type 2 specifies zlib-compressed JSON. This is significantly smaller than uncompressed JSON, and is straightforward to decode offchain. However, it is impractical for onchain consumers to use.

Encoding type 4 is [CBOR](https://cbor.io). CBOR is a binary encoding format that is a superset of JSON, and is both more compact and easier to parse in limited environments such as the EVM. Consumers that support CBOR are strongly encouraged to also support the [stringref extension](http://cbor.schmorp.de/stringref) to CBOR, which provides significant additional reduction in encoded size.

Encoding type 8 indicates that the ABI can be found elsewhere, at the specified URI. This is typically the most compact of the supported forms, but also adds external dependencies for implementers. The specified URI may use any schema, but HTTP, IPFS, and Swarm are expected to be the most common.

#### Resolver profile

A new resolver interface is defined, consisting of the following method:

```
function ABI(bytes32 node, uint256 contentType) constant returns (uint256, bytes);
```

The interface ID of this interface is 0x2203ab56.

contentType is a bitfield, and is the bitwise OR of all the encoding types the caller will accept. Resolvers that implement this interface must return an ABI encoded using one of the requested formats, or `(0, "")` if they do not have an ABI for this function, or do not support any of the requested formats.

The `abi` resolver profile is valid on both forward and reverse records.

#### ABI lookup process

When attempting to fetch an ABI based on an PNS name, implementers should first attempt an ABI lookup on the name itself. If that lookup returns no results, they should attempt a reverse lookup on the Pulsechain address the name resolves to.

Implementers should support as many of the ABI encoding formats as practical.

### Rationale

Storing ABIs onchain avoids the need to introduce additional dependencies for applications wishing to fetch them, such as swarm or HTTP access. Given the typical compactness of ABIs, we believe this is a worthwhile tradeoff in many cases.

The two-step resolution process permits different names to provide different ABIs for the same contract, such as in the case where it's useful to provide a minimal ABI to some callers, as well as specifying ABIs for contracts that did not specify one of their own. The fallback to looking up an ABI on the reverse record permits contracts to specify their own canonical ABI, and prevents the need for duplication when multiple names reference the same contract without the need for different ABIs.

### Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
