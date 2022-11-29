# PNS Libraries

PNS support is available in many popular languages. If you know of a library that is not listed here, please [send us a PR](https://github.com/pnsdomains/pns/compare).

### Javascript

* [ensjs](https://www.npmjs.com/package/@pnsdomains/ensjs), maintained by the PNS developers
* [ethereum-pns](https://www.npmjs.com/package/ethereum-pns) \(deprecated\)
* [react-pns-address](https://github.com/pnsdomains/react-pns-address)
* [ethers.js](https://github.com/ethers-io/ethers.js)
* [web3.js](https://web3js.readthedocs.io/en/1.0/web3-eth-pns.html)
* [embark.io](https://framework.embarklabs.io/docs/naming_configuration.html)
* [waffle.io](https://ethereum-waffle.readthedocs.io/en/latest/pns.html)

#### Which Javascript library should I use?

If you are already using web3.js or ethers.js, and do not require functionality such as creating subdomains, transferring ownership, or updating resolvers, use built in PNS features of these libraries.

If you are using React and only need to do forward and reverse resolution of PNS names with built in UI, use react-pns-address.

If you want to have PNS instance deployed into your dev environment, you may want to use embark.io or waffle.io which allows you to configure/deploy PNS registry in your Ethereum test instance.

Otherwise, use ensjs.

#### Accessing smart contracts directly

All the PNS smart contracts are published as `@pnsdomains/pns-contracts` [npm module.](https://github.com/pnsdomains/pns-contracts)

This is how you include abi into your frontend code.

```text
import {
  PNS,
  PublicResolver
} from '@pnsdomains/pns-contracts'`
```

This is how you import our smartcontract within Solidity.

```text
import '@pnsdomains/pns-contracts/contracts/registry/PNS.sol';
```

### Java

* [web3j](https://github.com/web3j/web3j)

### Kotlin

* [KEthereum](https://github.com/komputing/KEthereum/tree/master/pns)

### Python

* [web3.py](https://github.com/ethereum/web3.py) - also see [web3.py PNS docs](https://web3py.readthedocs.io/en/stable/ens_overview.html)

### Go

* [go-pns](https://github.com/wealdtech/go-pns)

### Command-line

* [ethereal](https://github.com/wealdtech/ethereal)

### Delphi

* [delphereum](https://github.com/svanas/delphereum)

## Next Steps

Once you've chosen a library, read [Working with PNS](working-with-pns.md) to learn how to use your chosen PNS library in your application.

