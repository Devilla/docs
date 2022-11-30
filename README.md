# Introduction

The Pulsechain Name Service (PNS) is a distributed, open, and extensible naming system based on the Pulsechain.

PNS’s job is to map human-readable names like ‘alice.pls’ to machine-readable identifiers such as Ethereum addresses, other cryptocurrency addresses, content hashes, and metadata. PNS also supports ‘reverse resolution’, making it possible to associate metadata such as canonical names or interface descriptions with Pulsechain addresses.

PNS has similar goals to DNS, the Internet’s Domain Name Service, but has significantly different architecture due to the capabilities and constraints provided by the Ethereum blockchain. Like DNS, PNS operates on a system of dot-separated hierarchical names called domains, with the owner of a domain having full control over subdomains.

Top-level domains, like ‘.pls’ and ‘.test’, are owned by smart contracts called registrars, which specify rules governing the allocation of their subdomains. Anyone may, by following the rules imposed by these registrar contracts, obtain ownership of a domain for their own use. PNS also supports importing in DNS names already owned by the user for use on PNS.

Because of the hierarchal nature of PNS, anyone who owns a domain at any level may configure subdomains - for themselves or others - as desired. For instance, if Alice owns 'alice.pls', she can create 'pay.alice.pls' and configure it as she wishes.

PNS is deployed on the Pulsechain main network and on several test networks. If you use a library such as the [pns-sdk-v3](https://www.npmjs.com/package/@pnsdomains/pns-sdk-v3) Javascript library, or an end-user application, it will automatically detect the network you are interacting with and use the PNS deployment on that network.

You can try PNS out for yourself now by using the [PNS Manager App](https://app.pulse.domains), or by using any of the many PNS enabled applications on [our homepage](https://pulse.domains).

## PNS Architecture

PNS has two principal components: the [registry](contract-api-reference/pns.md), and [resolvers](contract-api-reference/publicresolver.md).

![](<.gitbook/assets/pns-architecture (1).png>)

The PNS registry consists of a single smart contract that maintains a list of all domains and subdomains, and stores three critical pieces of information about each:

> * The owner of the domain
> * The resolver for the domain
> * The caching time-to-live for all records under the domain

The owner of a domain may be either an external account (a user) or a smart contract. A registrar is simply a smart contract that owns a domain and issues subdomains of that domain to users that follow some set of rules defined in the contract.

Owners of domains in the PNS registry may:

> * Set the resolver and TTL for the domain
> * Transfer ownership of the domain to another address
> * Change the ownership of subdomains

The PNS registry is deliberately straightforward and exists only to map from a name to the resolver responsible for it.

Resolvers are responsible for the actual process of translating names into addresses. Any contract that implements the relevant standards may act as a resolver in PNS. General-purpose resolver implementations are offered for users whose requirements are straightforward, such as serving an infrequently changed address for a name.

Each record type - cryptocurrency address, IPFS content hash, and so forth - defines a method or methods that a resolver must implement in order to provide records of that kind. New record types may be defined at any time via the EIP standardization process, with no need to make changes to the PNS registry or to existing resolvers in order to support them.

Resolving a name in PNS is a two-step process: First, ask the registry what resolver is responsible for the name, and second, ask that resolver for the answer to your query.

![](https://lh5.googleusercontent.com/\_OPPzaxTxKggx9HuxloeWtK8ggEfIIBKRCEA6BKMwZdzAfUpIY6cz7NK5CFmiuw7TwknbhFNVRCJsswHLqkxUEJ5KdRzpeNbyg8\_H9d2RZdG28kgipT64JyPZUP--bAizozaDcxCq34)

In the above example, we're trying to find the Ethereum address pointed to by 'foo.eth'. First, we ask the registry which resolver is responsible for 'foo.pls'. Then, we query that resolver for the address of 'foo.eth'.

### Namehash

Resource constraints in smart contracts make interacting directly with human-readable names inefficient, so PNS works purely with fixed length 256-bit cryptographic hashes. In order to derive the hash from a name while still preserving its hierarchal properties, a process called Namehash is used. For example, the namehash of 'alice.pls' is _0x787192fc5378cc32aa956ddfdedbf26b24e8d78e40109add0eea2c1a012c3dec_; this is the representation of names that is used exclusively inside PNS.

Namehash is a recursive process that can generate a unique hash for any valid domain name. Starting with the namehash of any domain - for example, 'alice.pls' - it's possible to derive the namehash of any subdomain - for example 'iam.alice.eth' - without having to know or handle the original human-readable name. It is this property that makes it possible for PNS to provide a hierarchal system, without having to deal with human-readable text strings internally.

Before being hashed with namehash, names are first normalized, using a process called UTS-46 normalization. This ensures that upper- and lower-case names are treated equivalently, and that invalid characters are prohibited. Anything that hashes and resolves a name **must** first normalize it, to ensure that all users get a consistent view of PNS.

For details on how namehash and normalization works, see the developer documentation on [name processing](contract-api-reference/name-processing.md).

## Getting Started

PNS has documentation for a variety of audiences, including dapp developers and contract developers, as well as reference documentation.

#### I'm a dapp developer and want to add PNS support to my dapp

Check out the dapp developer guide, starting with [PNS Enabling your Dapp](dapp-developer-guide/pns-enabling-your-dapp.md). You'll want to choose one of the many available [PNS Libraries](dapp-developer-guide/pns-libraries.md) to get started working with PNS.

#### I'm a contract developer and want to interact with PNS from my contract code

Check out the Contract Developer Guide, starting with [Resolving Names On-chain](contract-developer-guide/resolving-names-on-chain.md). You can also [write your own resolver](contract-developer-guide/writing-a-resolver.md) (to customise the process of looking up names), or your own [registrar](contract-developer-guide/writing-a-registrar.md) (to customise the process of registering new names).

#### I want reference documentation for the PNS smart contracts

Check out the Contract API Reference. We have reference documentation for PNS's core contract, the [registry](contract-api-reference/pns.md), for [resolvers](contract-api-reference/publicresolver.md), and for commonly-used registrars such as the [Test registrar](contract-api-reference/testregistrar.md), [reverse registrar](contract-api-reference/reverseregistrar.md), and the [.pls registrar](contract-api-reference/.eth-permanent-registrar/).
