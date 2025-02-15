---
description: >-
  Defines a resolver profile for PNS that provides features for storage and
  lookup of DNS records (formerly EIP-1185).
---

# PNSIP-6: DNS-in-PNS

{% hint style="danger" %}
This standard is obsolete, and does not reflect the current implementation.
{% endhint %}

| **Author**  | Jim McDonald (@mcdee) |
| ----------- | --------------------- |
| **Status**  | Stagnant              |
| **Created** | 2018-06-26            |

### Abstract

This PNSIP defines a resolver profile for PNS that provides features for storage and lookup of DNS records. This allows PNS to be used as a store of authoritative DNS information.

### Motivation

PNS is a highly desirable store for DNS information. It provides the distributed authority of DNS without conflating ownership and authoritative serving of information. With PNS, the owner of a domain has full control over their own DNS records. Also, PNS has the ability (through smart contracts) for a domain's subdomains to be irrevocably assigned to another entity.

### Specification

The resolver profile to support DNS on PNS follows the resolver specification as defined in #137.

Traditionally, DNS is a zone-based system in that all of the records for a zone are kept together in the same file. This has the benefit of simplicity and atomicity of zone updates, but when transposed to PNS can result in significant gas costs for simple changes. As a result, the resolver works on the basis of record sets. A record set is uniquely defined by the tuple (domain, name, resource record type), for example the tuple (example.com, www.example.com, A) defines the record set of A records for the name www.example.com in the domain example.com. A record set can contain 0 or more values, for example if www.example.com has A records 1.2.3.4 and 5.6.7.8 then the aforementioned tuple will have two values.

The choice to work at the level of record sets rather than zones means that this specification cannot completely support some features of DNS, such as zone transfers and DNSSEC. It would be possible to build a different resolver profile that works at the zone level, however it would be very expensive to carry out updates and so is not considered further for this PNSIP.

The DNS resolver interface consists of two functions to set DNS information and two functions to query DNS information.

#### setDNSRecords(bytes32 node, bytes data)

`setDNSRecords()` sets, updates or clears 1 or more DNS records for a given node. It has function signature `0x0af179d7`.

The arguments for the function are as follows:

* node: the namehash of the fully-qualified domain in PNS for which to set the records. Namehashes are defined in #137
* data: 1 or more DNS records in DNS wire format. Any record that is supplied without a value will be cleared. Note that all records in the same RRset should be contiguous within the data; if not then the later RRsets will overwrite the earlier one(s)

#### clearDNSZone(bytes32 node)

`clearDNSZone()` removes all DNS records for the domain. It has function signature `0xad5780af`.

Although it is possible to clear records individually with `setDNSRecords()` as described above this requires the owner to know all of the records that have been set (as the resolver has no methods to iterate over the records for a given domain), and might require multiple transactions. `clearDNSZone()` removes all zone information in a single operation.

The arguments for the function is as follows:

* node: the namehash of the fully-qualified domain in PNS for which to clear the records. Namehashes are defined in #137

#### dnsRecords(bytes32 node, bytes32 name, uint16 resource) view returns (bytes)

`dnsRecords()` obtains the DNS records for a given node, name and resource. It has function signature `0x2461e851`.

The arguments for the function are as follows:

* node: the namehash of the fully-qualified domain in PNS for which to set the records. Namehashes are defined in #137
* name: the `keccak256()` hash of the name of the record in DNS wire format.
* resource: the resource record ID. Resource record IDs are defined in https://en.wikipedia.org/wiki/List\_of\_DNS\_record\_types

The function returns all matching records in DNS wire format. If there are no records present the function will return nothing.

#### hasDNSRecords(bytes32 node, bytes32 name) view returns (bool)

`hasDNSRecords()` reports if there are any records for the provided name in the domain. It has function signature `0x4cbf6ba4`.

This function is needed by DNS resolvers when working with wildcard resources as defined in https://tools.ietf.org/html/rfc4592

The arguments for the function are as follows:

* node: the namehash of the fully-qualified domain in PNS for which to set the records. Namehashes are defined in #137
* name: the `keccak256()` hash of the name of the record in DNS wire format.

The function returns `true` if there are any records for the provided node and name, otherwise `false`.

### Backwards compatibility

Not applicable.

### Implementation

The reference implementation of the DNS resolver is at https://github.com/wealdtech/wealdtech-solidity/blob/master/contracts/pns/DNSResolver.sol

https://github.com/wealdtech/ethereal.git can be used to test the functionality of the resolver with the "dns set", "dns get" and "dns clear" commands.

### Test Cases

Test cases for the DNS resolver are at https://github.com/wealdtech/wealdtech-solidity/blob/master/test/pns/DNSResolver.js

### Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
