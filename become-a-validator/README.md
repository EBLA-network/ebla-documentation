---
description: Join the EBLA network as a Validator!
---

# 🌱 Become a Validator

## Becoming a Validator on the Mainnet

We've launched our Mainnet network! You can now join the Mainnet as a validator.&#x20;

To become a validator, you need to do and be aware of the following,&#x20;

* [Set up a validator node](set-up-validator-node.md)&#x20;
* [Register your validator node on the community site](register-node-via-community-site.md), OR
* [Register your validator node by directly interacting with the on-chain DPoS contract](register-node-directly-on-chain.md)
* [Solicit delegation](solicit-delegation.md)
* [Node upgrade & reset operations](node-upgrade-and-reset.md)
* [Monitor your node for inactivity slashing](../how-to/check-validator-slashing.md)



## Quick FAQs

### Do Validator operators need to put up collateral or self-stake?&#x20;

There is a 100 EBLA self-staking requirement to register with the on-chain DPoS contract. At the time of this writing 100 EBLA isn't a lot of money, and this is not meant to be collateral, but rather to deter spamming attacks on the DPoS contract.&#x20;



### How do Validators produce blocks?&#x20;

Validators receive delegation of EBLA tokens. A minimum total stake of 5,000 EBLA is required to start producing blocks, and each Validator can take on a maximum of 1,000,000 EBLA of delegated stake. The more delegated EBLA a Validator has, the more likely and more frequently it will produce blocks.&#x20;



### How are Validators rewarded?&#x20;

Validators charge a commission, which is a percentage of the total staking yields that come from the EBLA that has been delegated to it.&#x20;



### Where can I see a list of current Validators?&#x20;

You can see a list of current Validators, their delegation, yields efficiency and commissions on the [EBLA staking site's Staking section](https://staking.eblanetwork.com/staking).&#x20;



### How do I know if my node is being slashed for inactivity?&#x20;

If your validator stops producing blocks for ~10,000 blocks (~10 hours) it loses 5% of its voting power per window, and is force-undelegated if its effective stake falls below 5,000 EBLA. You can check your node's live status, spot any slashing, and project an eviction time with the [Check validator slashing status](../how-to/check-validator-slashing.md) guide.&#x20;

