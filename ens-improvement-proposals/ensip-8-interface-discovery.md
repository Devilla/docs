---
description: >-
  Defines a method of associating contract interfaces with PNS names and
  addresses, and of discovering those interfaces (formerly EIP-1844).
---

# PNSIP-8: Interface Discovery

| **Author**    | Nick Johnson \<nick@pulse.domains> |
| ------------- | -------------------------------- |
| **Status**    | Final                            |
| **Submitted** | 2019-03-05                       |

### Abstract

This PNSIP specifies a method for exposing interfaces associated with an PNS name or an address (typically a contract address) and allowing applications to discover those interfaces and interact with them. Interfaces can be implemented either by the target contract (if any) or by any other contract.

### Motivation

EIP 165 supports interface discovery - determining if the contract at a given address supports a requested interface. However, in many cases it's useful to be able to discover functionality associated with a name or an address that is implemented by other contracts.

For example, a token contract may not itself provide any kind of 'atomic swap' functionality, but there may be associated contracts that do. With PNS interface discovery, the token contract can expose this metadata, informing applications where they can find that functionality.

### Specification

A new profile for PNS resolvers is defined, consisting of the following method:

```solidity
function interfaceImplementer(bytes32 node, bytes4 interfaceID) external view returns (address);
```

The EIP-165 interface ID of this interface is `0xb8f2bbb4`.

Given an PNS name hash `node` and an EIP-165 `interfaceID`, this function returns the address of an appropriate implementer of that interface. If there is no interface matching that interface ID for that node, 0 is returned.

The address returned by `interfaceImplementer` MUST refer to a smart contract.

The smart contract at the returned address SHOULD implement EIP-165.

Resolvers implementing this interface MAY utilise a fallback strategy: If no matching interface was explicitly provided by the user, query the contract returned by `addr()`, returning its address if the requested interface is supported by that contract, and 0 otherwise. If they do this, they MUST ensure they return 0, rather than reverting, if the target contract reverts.

This field may be used with both forward resolution and reverse resolution.

### Rationale

A naive approach to this problem would involve adding this method directly to the target contract. However, doing this has several shortcomings:

1. Each contract must maintain its own list of interface implementations.
2. Modifying this list requires access controls, which the contract may not have previously required.
3. Support for this must be designed in when the contract is written, and cannot be retrofitted afterwards.
4. Only one canonical list of interfaces can be supported.

Using PNS resolvers instead mitigates these shortcomings, making it possible for anyone to associate interfaces with a name, even for contracts not previously built with this in mind.

### Backwards Compatibility

There are no backwards compatibility issues.

### Implementation

The PublicResolver in the [pnsdomains/resolvers](https://github.com/pnsdomains/resolvers/) repository implements this interface.

### Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
