# 🚩 Register node via community site

You can register a validator node from the EBLA staking site by connecting your wallet — no account or KYC required. You may also [register a node by directly interacting with the on-chain DPoS contract](register-node-directly-on-chain.md).

### 0.  IMPORTANT: the wallet used to register a node is the owner of that node

Validator node registration is an on-chain transaction. The wallet you use to register a node is the "owner" of that node. This means once a node is registered, the "owner wallet" is now required to,&#x20;

* Change the validator node's commission rate
* Claim commission rewards earned by the validator node

A single owner wallet can register multiple validator nodes and become their owner, this makes it easy to manage multiple validator nodes in aggregate.&#x20;

**Please safeguard your validator nodes' owner wallet!**&#x20;

### 1.  Connect your wallet and open "Run a node"

Navigate to the [Run a node](https://staking.eblanetwork.com/node) section of the EBLA staking site, connect your wallet, and select "Register a node".&#x20;

<figure><img src="../.gitbook/assets/3. register a validator.png" alt=""><figcaption></figcaption></figure>

You'll see a node registration prompt asking for several pieces of node-specific information.&#x20;

<figure><img src="../.gitbook/assets/3. enter node details.png" alt=""><figcaption></figcaption></figure>

Here are instructions on how to find the node's,&#x20;

* [Public address](../node-setup/node\_address.md)
* [Proof of ownership](../node-setup/proof\_owership.md)
* [VRF key](../node-setup/vrf\_key.md)

Note that the container name may differ between environments (for example `ebla_compose_node_1` vs `mainnet_node_1`). To be sure you've got the right container name, just execute `docker ps`.&#x20;

### 2.  Setting the commission

The next entry to registering a node is setting its commission, which is the portion of the staking yield that goes to the node operator. Enter a whole number between 10 and 100 (the minimum commission is 10%).&#x20;

Here is some [information on commissions](../faq/mainnet.md). This is a purely economic decision on the part of the validator operators; it's advisable to review how much commission other validators are charging first by going to the [Staking section](https://staking.eblanetwork.com/staking) of the staking site.&#x20;

### 3.  Self-delegation requirement

Once you click "Submit" in the node registration pop-up, the on-chain transaction will require that the operator self-delegate 100 EBLA to their own validator node — so make sure you have enough in the wallet to complete the on-chain registration of the validator node.&#x20;

This self-delegation requirement is not meant to be a financial burden, but purely to guard against spamming.&#x20;
