# PNS as NFT

When PNS .eth registrar migrated in May 2019, the .eth registrar became an [ERC721](https://github.com/pnsdomains/pns/blob/master/docs/ethregistrar.rst#id3) compliant non-fungible token contract, meaning that .eth registrations can be transferred in the same fashion as other NFTs.

## Deriving tokenId from PNS name

The tokenId of PNS name is simply the uint256 representation of the hash of the label (`vitalik` for `vitalik.eth`).

```javascript
const ethers = require('ethers')
const BigNumber = ethers.BigNumber
const utils = ethers.utils
const name = 'vitalik'
const labelHash = utils.keccak256(utils.toUtf8Bytes(name))
const tokenId = BigNumber.from(labelHash).toString()
```

In the example above,[`79233663829379634837589865448569342784712482819484549289560981379859480642508`](https://opensea.io/assets/0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85/79233663829379634837589865448569342784712482819484549289560981379859480642508) is the tokenId of `vitalik.eth`

## Deriving PNS name from tokenId

Unlike deriving tokenId, deriving PNS name from tokenId is not as easy. This is because all PNS names are stored as fixed-length hash to allow registering infinite length of names. The downside of this architecture is that you cannot directly query PNS smart contracts to return PNS name using tokenId.

Our recommended way is to query via [https://thegraph.com](https://thegraph.com) PNS subgraph. The graph decodes the hash to name as it indexes. The example code to query is as follows.

```javascript
const ethers = require('ethers')
const BigNumber = ethers.BigNumber
const gr = require('graphql-request')
const { request, gql } = gr
const tokenId = '79233663829379634837589865448569342784712482819484549289560981379859480642508'
// Should return 0xaf2caa1c2ca1d027f1ac823b529d0a67cd144264b2789fa2ea4d63a67c7103cc
const labelHash = BigNumber.from(tokenId).toHexString()

const url = 'https://api.thegraph.com/subgraphs/name/pnsdomains/pns'
const GET_LABEL_NAME = gql`
query{
  domains(first:1, where:{labelhash:"${labelHash}"}){
    labelName
  }
}`

request(url, GET_LABEL_NAME).then((data) => console.log(data))
// { domains: [ { labelName: 'vitalik' } ] }
```

If you prefer not to rely on a third party like TheGraph, the team open-sourced [pns-rainbow](https://github.com/graphprotocol/pns-rainbow) containing a link to the original dataset (6GB with 133 million entities) so that you can host your own PNS name decoding service.

## Turning subdomain into NFT

Currently, all the subdomains nor non `.eth` domains are not NFT, unless the domain registrar itself supports NFT such as (`dcl.eth`, and `.kred`). If you want to turn all subdomains which you own, you have to create a registrar

1. Create a registrar contract as ERC721 compliant
2. Set PNS registry address (mostly when you deploy the registrar)
3. Create `register` function which calls `registry.setSubnodeOwner` then mint the token making the subdomain label hash as tokenId

```
contract DCLRegistrar is ERC721Full, Ownable {
    constructor(
        IPNSRegistry _registry,
    ) public ERC721Full("DCL Registrar", "DCLPNS") {
        // PNS registry
        updateRegistry(_registry);
    }

    function register(
        string memory _subdomain,
        bytes32 subdomainLabelHash,
        address _beneficiary,
        uint256 _createdDate
    ) internal {
        // Create new subdomain and assign the _beneficiary as the owner
        registry.setSubnodeOwner(domainNameHash, subdomainLabelHash, _beneficiary);
        // Mint an ERC721 token with the subdomain label hash as its id
        _mint(_beneficiary, uint256(subdomainLabelHash));
    }
}
```

Once deployed, then you have to transfer the controller address to the contract.

For non-technical users, we are currently working on upgrading our `SubdomainRegistrar` which allows you to turn your subdomain into NFT without any coding.

## Metadata

.eth does not have `.tokenURI` . However, we created a separate metadata service which NFT marketplaces like OpenSea can fetch metadata for PNS such as registration data, expiration date, name length, etc. For more detail, please refer to the metadata documentation site.

{% embed url="https://metadata.pulse.domains/docs" %}
