# Economic Model

## 5. Economics

Note: this is a _**preliminary outline**_ of the overall intent for the EBLA network's economics. These designs will be finalized at network launch via the governance process by stakeholders from the community, and they are subject to alterations via the same process.

The native digital cryptographically-secured utility token of EBLA (**EBLA token**) is a transferable representation of attributed functions specified in the protocol/code of EBLA, which is designed to play a major role in the functioning of the ecosystem on EBLA and intended to be used solely as the primary utility token on the network.

EBLA token is a non-refundable functional utility token which will be used as the medium of exchange between participants on EBLA. The goal of introducing EBLA token is to provide a convenient and secure mode of payment and settlement between participants who interact within the ecosystem on EBLA, and it is not, and not intended to be, a medium of exchange accepted by the public (or a section of the public) as payment for goods or services or for the discharge of a debt; nor is it designed or intended to be used by any person as payment for any goods or services whatsoever that are not exclusively provided by the issuer. EBLA token does not in any way represent any shareholding, participation, right, title, or interest in the Foundation, the Distributor, their respective affiliates, or any other company, enterprise or undertaking, nor will EBLA token entitle token holders to any promise of fees, dividends, revenue, profits or investment returns, and are not intended to constitute securities in Singapore or any relevant jurisdiction. EBLA token may only be utilized on EBLA, and ownership of EBLA token carries no rights, express or implied, other than the right to use EBLA token as a means to enable usage of and interaction within EBLA.

EBLA token would also provide the economic incentives which will be distributed to encourage users to contribute and maintain the ecosystem on EBLA, thereby creating a win-win system where every participant is fairly compensated for its efforts. EBLA token is an integral and indispensable part of EBLA, because without EBLA token, there would be no incentive for users to expend resources to participate in activities or provide services for the benefit of the entire ecosystem on EBLA. Given that additional EBLA token will be awarded to a user based only on its actual usage, activity and contribution on EBLA, users of EBLA and/or holders of EBLA token which did not actively participate will not receive any EBLA token incentives.

### 5.1 Token Supply

EBLA has a hard cap of 12 billion tokens. 5 billion are preminted at genesis; the remaining 7 billion are released over time as DPoS staking rewards.

There will be inflation (see the section on [Rewards](economic-model.md#5-4-rewards)) which comes in the form of block rewards providing staking yield, but will only inflate upon stakes that are in circulation and have been delegated to consensus nodes. In other words, any tokens that are locked cannot be staked or delegated, therefore generates no inflation.&#x20;

### 5.2 Token Utility

**Transaction Fees**

EBLA token holders may use the token to initiate and pay for the cost of processing transactions, and these transaction fees are paid to the validators of the network. More on transaction fees in the Rewards section.

**Validator Eligibility**

EBLA token holders may choose to stake their tokens via bonding to become an eligible validator of the network, responsible for processing the network’s transactions and maintaining the network’s overall security and integrity. As an indication of commitment to the system and service standard assurance, users would be required to place an amount of EBLA token as security deposit before it may participate in mining for the benefit of the network.

Specific rules for validator eligibility will be set at network launch and are subject to change per stakeholders’ voting. In general, we want there to be a minimum amount staked and/or delegated to a specific node to participate in consensus, but we want to keep that threshold low to maximize participation. We also want to make sure there's a cap for the amount of tokens staked and/or delegated to a single consensus node, so that there isn't excessive concentration (centralization) of stake into any particular node.

### 5.3 Staking

Staking is a mechanism for stakeholders to signal their commitment to help maintain and grow the EBLA network, and they are rewarded (see Rewards section) accordingly. To stake, the stakeholder needs to bond EBLA tokens for a period, in return gaining additional weight in voting in governance-related decisions and become eligible to participate in the EBLA network’s consensus process and earning rewards.\
\
**Staking Period**

For simplicity, the staking period will be based on a fixed timeframe to be determined at network launch. The actual timeframe (as all time-related metrics) are governed by the number of Periods elapsed on the EBLA network.

At network launch there may a special set of launch nodes that have a longer staking period with greater rewards. Details are to be determined.\
\
**Staking via Bonding**

Bonding of EBLA tokens is needed in staking in order to provide stable expectations for eligibility. To bond a token is to lock a token into a fixed period of time (as determined by the number of Periods on chain), during which the bonded token is not tradeable or usable – besides granting the privileges in validation eligibility and access to current state storage on the network.\
\
**Stake Delegation**

Not everyone who wishes to help secure the EBLA network wishes to deal with the actual operational details of running a node. These stakeholders can choose to bond their stake and then delegate it to another entity to operate a node on its behalf. How the proceeds from the stake are divide may be negotiated between the stakeholder and the node operator, via secondary smart contracts on the EBLA network.\
\
**Stake Alterations**

When a stakeholder wishes to bond or withdraw (at the end of the staking period) his / her stake, there is a buffer period (e.g., a few days) before such changes take effect. Since the staking amount is fixed, fractional stake alterations aren’t possible. The reasoning behind this buffer period is two-fold.

The first consideration is security. Because staking endows critical privileges on the network, namely the right to validate transactions, it is critical that the network has full understanding for eligibility. The network and the community need to be given ample time to note any such changes and be confident in their determination of any changes in eligibility resulting from the staking alterations.

The second consideration is economic. When large quantities of tokens are either added or removed from the staking pool, it could have an impact on the secondary markets for both the tokens themselves as well as the storage rentals (i.e., those with bonded stakes could rent out the storage they’re entitled to DApp developers). Having a buffer period will give the community ample notice to react to the change.

**Target Staking Rate**

The network will have a target staking rate the entire token supply to be determined at network launch. This is the “desired” number of tokens in the overall supply that is locked into stake, leaving the remaining tokens used for fees on the network. Having a target staking rate is meant to encourage the community to stake their coins and help secure the network.

The target staking rate is tentatively set at 67%. EBLA finalizes blocks at a 5/8 (62.5%) quorum, so a high staking rate raises the cost of acquiring enough stake to threaten consensus.

### 5.4 Rewards

Rewards in EBLA are made up of block rewards and transaction fees. \\

**Block Rewards**

Block rewards (EBLA tokens) will be distributed to incentivize stakeholders to help secure the network by participating in validation by driving up the overall network’s staking rate. Computational resources are required for validation and verification of blockchain information, so providers of these services / resources would require payment for the consumption of these resources to maintain network security, and EBLA token will be used as the network currency to quantify and pay the costs of the consumed computational resources. This is especially important in the early days of the network where distributed fees may not be sufficient to attract node operators to secure the network.

Block rewards come in the form of inflation on top of the staked tokens. Since only nodes with sufficient staking or have received sufficient delegated stake are eligible to produce blocks, the amount of block rewards can be thought of as a direct yield which will be added to the total staked token supply.

Staking yield in EBLA follows a fixed schedule set at genesis: it starts at 7% and decays epoch-by-epoch toward a 1% floor.

![](<../.gitbook/assets/staking reward.png>)

The initial yield is 7%.

Block rewards are dispersed to staked validators the successful completion of these activities,

* Proposing valid blocks in the Block DAG
* Proposing valid consensus blocks on the PBFT chain

**Transaction Fees**

EBLA will make use of the same first-price auction mechanism currently being used in BTC, ETH, and almost all other blockchain systems.

The primary criticism against the fees in classic systems such as BTC and ETH are that, they are 1) too high, and 2) too volatile. However, these symptoms are primarily driven not by the pricing mechanism itself, but rather by the throughput limitations of these networks. To put it simply, if the system is not at its maximum throughput limit, the block proposer has no incentive to prioritize between which pending transactions to include in a block – they’d simply place all available (and eligible) pending transactions into the next block.

As a sidebar, this isn’t entirely true in practice, as the network propagation speed consideration during competitive mining could adversely impact inclusion as well. In the EBLA network however, there is no competition between the staking nodes so this adverse incentive should not exist.

Because of the very low maximum throughput in systems such as BTC and ETH, these networks are almost always operating at their maximum throughput, forcing users into constantly paying premiums to be included in blocks earlier, driving up both fee prices as well as fee volatility. In a far higher-throughput system such as EBLA, we expect this to be far less of a consideration.

Just like block rewards, fees are going to be dispersed to staked validators for the successful completion of same activities,

* Proposing valid blocks in the Block DAG
* Proposing valid consensus blocks on the PBFT chain

***

**Inactivity Slashing**

EBLA applies inactivity slashing. A validator's effective voting power decays while it fails to produce blocks, and it is force-undelegated (evicted) if its effective stake falls below the 5,000 EBLA eligibility threshold. Producing a block restores the validator's full voting power.

This keeps consensus weighted toward validators that are actively participating, without touching the underlying delegated balances.

##
