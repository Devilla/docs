# Deploying PNS on a Private Chain

If you’d like to deploy PNS on your own network, or deploy your own copy of PNS on a public network, this guide shows you how. If you want to use an existing PNS deployment, see [Resolving Names](dapp-developer-guide/resolving-names.md), [Managing Names](dapp-developer-guide/managing-names.md), and [Registering & Renewing Names](dapp-developer-guide/registering-and-renewing-names.md) instead.

On this page we will use Javascript, Web3, and [Hardhat](https://hardhat.org/) with npm for simplicity. You will find a complete migration file example [at the bottom of this page](deploying-pns-on-a-private-chain.md#migration-file-example).

Please be aware that existing frameworks such as [waffle](https://ethereum-waffle.readthedocs.io/en/latest/pns.html) and [embark](https://framework.embarklabs.io/docs/naming_configuration.html) have support for local PNS deployment as well.

## Importing contracts

The essential smart contracts are published [as npm modules](https://www.npmjs.com/package/@pnsdomains/pns-contracts). You can install them in your npm project with `npm install @pnsdomains/pns-contracts`. Now, you can require them in a migration script as follows \(see the [Truffle Documentation](https://truffleframework.com/docs/truffle/getting-started/package-management-via-npm) on working with contract artifacts and npm for details\)

```javascript
import {
  PNS, PNSRegistry, PublicResolver
} from '@pnsdomains/pns-contracts'
```

Including them within your smart contract is as follows

```javascript
import '@pnsdomains/pns-contracts/contracts/registry/PNS.sol'
```

`PNS` contains only an interface while `PNSRegistry` includes the actual implementation.

## Deploy the Registry

The registry is PNS’s central component and stores, among other things, who owns which domain. This is the example using ethers and hardhat.

```javascript
const PNSRegistry = await ethers.getContractFactory("PNSRegistry")
await PNSRegistry.deploy()
```

Once deployed, you will have a fresh PNS registry, whose root node is owned by the account that submitted the transaction. This account has total control over the PNS registry - it can create and replace any node in the entire tree.

From here, it's possible to create and manage names by directly interacting with the registry, as described in [Managing Names](dapp-developer-guide/managing-names.md). However, you will probably want to [deploy a resolver](deploying-pns-on-a-private-chain.md#deploy-a-resolver), and you may want to [deploy a registrar](deploying-pns-on-a-private-chain.md#deploy-a-registrar) so other users can register names.

## Deploy a Resolver

Records in the registry can point to resolver contracts which store additional domain information. The most common use-case is to store an address for a domain, but storing a contract ABI or text is also possible. For most purposes on private networks it's convenient to have an unrestricted general-purpose resolver available. Deploying one is straightforward:

```javascript
const ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";
const PNSRegistry = await ethers.getContractFactory("PNSRegistry")
const registry = await PNSRegistry.deploy()
await registry.deployed()
const PublicResolver = await ethers.getContractFactory("PublicResolver")
const resolver = await PublicResolver.deploy(registry.address, ZERO_ADDRESS);
await resolver.deployed()
```

The [`PublicResolver`](https://github.com/pnsdomains/pns-contracts/blob/master/contracts/resolvers/PublicResolver.sol) looks up ownership in the registry, which is why the registry's address is required at deployment.

For ease of use, we can give this resolver a name:

```javascript
const ethers = require('ethers');
const utils = ethers.utils;
const labelhash = (label) => utils.keccak256(utils.toUtf8Bytes(label))
const namehash = require('eth-ens-namehash');

async function setupResolver(pns, resolver, accounts) {
  const resolverNode = namehash.hash("resolver");
  const resolverLabel = labelhash("resolver");

  await pns.setSubnodeOwner("0x0000000000000000000000000000000000000000", resolverLabel, accounts[0]);
  await pns.setResolver(resolverNode, resolver.address);
  await resolver.setAddr(resolverNode, resolver.address);
}
```

Above, we first create a new top-level domain, "resolver", then set its resolver address to our newly deployed public resolver. Finally, we set up an address record for "resolver", pointing back to the resolver address. In effect, the resolver is answering queries about its own address. After this, anyone can find the public resolver at the special PNS name "resolver". We call this function after deploying the public resolver in a `.then()` block as we did with the resolver.

## Deploy a Registrar

So far, domains can only be registered manually by the owner of the registry's root node. Fortunately, contracts can also own nodes. This means we can set up a registrar contract as the owner of a node, e.g. "test", in the registry which enables it to distribute subdomains such as "mycontract.test". It allows us to have custom, on-chain logic which governs domain allocation. Once we own a \(sub-\)node we are free to repeat this process and set up another registrar. If you are part of the "myorg" organisation you could register "myorg.test" and let it point to your custom registrar which only allows certified members of your organisation to claim subdomains such as "bob.myorg.test". For our private network, we'll use the simple 'first come, first served' [FIFSRegistrar](https://github.com/pnsdomains/pns/blob/master/contracts/FIFSRegistrar.sol), and set it as the owner of the top-level domain "test" in our migration script:

```javascript
...
  const registrar = await FIFSRegistrar.deploy(pns.address, namehash.hash("test"));
  await registrar.deployed();
  await pns.setSubnodeOwner("0x0000000000000000000000000000000000000000", sha3("test"), registrar.address);
})
...
```

## Deploy the Reverse Registrar

Similarly, if you wish to enable reverse resolution on your deployment, you will need to deploy the reverse registrar:

```javascript
...
const reverseRegistrar = await ReverseRegistrar.deploy(pns.address, resolver.address);
await reverseRegistrar.deployed();
setupReverseRegistrar(pns, resolver, reverseRegistrar, accounts);
...

async function setupReverseRegistrar(pns, resolver, reverseRegistrar, accounts) {
  await pns.setSubnodeOwner("0x0000000000000000000000000000000000000000", utils.sha3("reverse"), accounts[0]);
  await pns.setSubnodeOwner(namehash.hash("reverse"), utils.sha3("addr"), reverseRegistrar.address);
}
```

## Migration File Example

We can combine the steps above in a single hardhat migration file. This allows us to deploy PNS in one go:

### contracts/deps.sol

```text
//SPDX-License-Identifier: MIT
// These imports are here to force Hardhat to compile contracts we depend on in our tests but don't need anywhere else.
import "@pnsdomains/pns-contracts/contracts/registry/PNSRegistry.sol";
import "@pnsdomains/pns-contracts/contracts/registry/FIFSRegistrar.sol";
import "@pnsdomains/pns-contracts/contracts/resolvers/PublicResolver.sol";
````

### script/deploy.js

```javascript
const hre = require("hardhat");
const namehash = require('eth-ens-namehash');
const tld = "test";
const ethers = hre.ethers;
const utils = ethers.utils;
const labelhash = (label) => utils.keccak256(utils.toUtf8Bytes(label))
const ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";
const ZERO_HASH = "0x0000000000000000000000000000000000000000000000000000000000000000";
async function main() {
  const PNSRegistry = await ethers.getContractFactory("PNSRegistry")
  const FIFSRegistrar = await ethers.getContractFactory("FIFSRegistrar")
  const ReverseRegistrar = await ethers.getContractFactory("ReverseRegistrar")
  const PublicResolver = await ethers.getContractFactory("PublicResolver")
  const signers = await ethers.getSigners();
  const accounts = signers.map(s => s.address)

  const pns = await PNSRegistry.deploy()
  await pns.deployed()
  const resolver = await PublicResolver.deploy(pns.address, ZERO_ADDRESS);
  await resolver.deployed()
  await setupResolver(pns, resolver, accounts)
  const registrar = await  FIFSRegistrar.deploy(pns.address, namehash.hash(tld));
  await registrar.deployed()
  await setupRegistrar(pns, registrar);
  const reverseRegistrar = await ReverseRegistrar.deploy(pns.address, resolver.address);
  await reverseRegistrar.deployed()
  await setupReverseRegistrar(pns, registrar, reverseRegistrar, accounts);
};

async function setupResolver(pns, resolver, accounts) {
  const resolverNode = namehash.hash("resolver");
  const resolverLabel = labelhash("resolver");
  await pns.setSubnodeOwner(ZERO_HASH, resolverLabel, accounts[0]);
  await pns.setResolver(resolverNode, resolver.address);
  await resolver['setAddr(bytes32,address)'](resolverNode, resolver.address);
}

async function setupRegistrar(pns, registrar) {
  await pns.setSubnodeOwner(ZERO_HASH, labelhash(tld), registrar.address);
}

async function setupReverseRegistrar(pns, registrar, reverseRegistrar, accounts) {
  await pns.setSubnodeOwner(ZERO_HASH, labelhash("reverse"), accounts[0]);
  await pns.setSubnodeOwner(namehash.hash("reverse"), labelhash("addr"), reverseRegistrar.address);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

To execute the migration file on hardhat, run the following command line.

```text
npx hardhat run scripts/deploy.js
```

### Deploying PNS in a single transaction

Alternately you may wish to deploy a test registrar and its dependencies with a single transaction. This is useful for example in unit tests where you wish to start from a clean slate in each test. In many cases it will also be faster than sending a series of separate transactions.

This can be done by deploying a new contract that creates and sets up all the other contracts in its constructor. The below code creates all the PNS contracts and assigns the eth TLD to the FIFS Registrar so that any eth domain may be registered in the unit tests.

```text
pragma solidity >=0.8.4;
import {INameWrapper, PublicResolver} from '@pnsdomains/pns-contracts/contracts/resolvers/PublicResolver.sol';
import '@pnsdomains/pns-contracts/contracts/registry/PNSRegistry.sol';
import '@pnsdomains/pns-contracts/contracts/registry/FIFSRegistrar.sol';
import {NameResolver, ReverseRegistrar} from '@pnsdomains/pns-contracts/contracts/registry/ReverseRegistrar.sol';

// Construct a set of test PNS contracts.
contract PNSDeployer {
  bytes32 public constant TLD_LABEL = keccak256('eth');
  bytes32 public constant RESOLVER_LABEL = keccak256('resolver');
  bytes32 public constant REVERSE_REGISTRAR_LABEL = keccak256('reverse');
  bytes32 public constant ADDR_LABEL = keccak256('addr');

  PNSRegistry public pns;
  FIFSRegistrar public fifsRegistrar;
  ReverseRegistrar public reverseRegistrar;
  PublicResolver public publicResolver;

  function namehash(bytes32 node, bytes32 label) public pure returns (bytes32) {
    return keccak256(abi.encodePacked(node, label));
  }

  constructor() public {
    pns = new PNSRegistry();
    publicResolver = new PublicResolver(pns, INameWrapper(address(0)));

    // Set up the resolver
    bytes32 resolverNode = namehash(bytes32(0), RESOLVER_LABEL);

    pns.setSubnodeOwner(bytes32(0), RESOLVER_LABEL, address(this));
    pns.setResolver(resolverNode, address(publicResolver));
    publicResolver.setAddr(resolverNode, address(publicResolver));

    // Create a FIFS registrar for the TLD
    fifsRegistrar = new FIFSRegistrar(pns, namehash(bytes32(0), TLD_LABEL));

    pns.setSubnodeOwner(bytes32(0), TLD_LABEL, address(fifsRegistrar));

    // Construct a new reverse registrar and point it at the public resolver
    reverseRegistrar = new ReverseRegistrar(
      pns,
      NameResolver(address(publicResolver))
    );

    // Set up the reverse registrar
    pns.setSubnodeOwner(bytes32(0), REVERSE_REGISTRAR_LABEL, address(this));
    pns.setSubnodeOwner(
      namehash(bytes32(0), REVERSE_REGISTRAR_LABEL),
      ADDR_LABEL,
      address(reverseRegistrar)
    );
  }
}
```



