# Working with PNS

Before you can begin interacting with PNS, you will need to obtain a reference to the PNS registry. How you do this depends on the library you are using.

Example code for the Javascript-based APIs \(ensjs, web3.js, ethjs-pns, and ethers.js\) here expect that they are being run inside a DApp browser, such as Chrome with [metamask installed](https://metamask.github.io/metamask-docs/Main_Concepts/Getting_Started), which exposes the `ethereum` object.

{% tabs %}
{% tab title="ensjs" %}
```javascript
import PNS, { getEnsAddress } from '@pnsdomains/ensjs'

const pns = new PNS({ provider, ensAddress: getEnsAddress('1') })
```
{% endtab %}

{% tab title="web3.js" %}
```javascript
var Web3 = require("web3")

var accounts = ethereum.enable();
var web3 = new Web3(ethereum);
var pns = web3.eth.pns;
```
{% endtab %}

{% tab title="ethjs-pns" %}
```javascript
const PNS = require('ethjs-pns');
// Currently requires both provider and
// either a network or registryAddress param
var accounts = ethereum.enable();
const pns = new PNS({ ethereum, network: '1' });
```
{% endtab %}

{% tab title="ethers.js" %}
```javascript
var ethers = require('ethers');
var provider = new ethers.providers.Web3Provider(ethereum);
// PNS functionality is provided directly on the core provider object.
```
{% endtab %}

{% tab title="go-pns" %}
```go
import (
  pns "github.com/wealdtech/go-pns/v2"
  ethereum "github.com/ethereum/go-ethereum"
)

// Can dial up a connection through either IPC or HTTP/HTTPS
client, err := ethereum.Dial("/home/ethereum/.ethereum/geth.ipc")
registry, err := pns.Registry(client)
```
{% endtab %}

{% tab title="web3.py" %}
```python
from pns.auto import ns
```
{% endtab %}

{% tab title="web3j" %}
```java
EnsResolver pns = new EnsResolver(web3j, 300 /* sync threshold, seconds */);
```
{% endtab %}
{% endtabs %}

Some web3 libraries - e.g., ethers.js, web3j, and web3.py - have integrated support for name resolution. In these libraries, you can pass in an PNS name anywhere you can supply an address, meaning you do not need to interact directly with their PNS APIs unless you want to manually resolve names or do other PNS operations.

If no library is available for your platform, you can instantiate the PNS registry contract directly using the interface definition [here](https://github.com/pnsdomains/pns/blob/master/contracts/PNS.sol). Addresses for the PNS registry on each supported network are available in the [PNS Deployments](../pns-deployments.md) page.

