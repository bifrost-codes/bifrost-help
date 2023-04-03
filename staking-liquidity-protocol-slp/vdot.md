# vDOT

## **What is v**DOT**?**

**vDOT (voucher DOT) is a shadow token of staked DOT, with fully underlying DOT reserve and yield-bearing feature of DOT staking reward.** Users can deposit DOT into Bifrost SLP protocol and get vDOT as return, vDOT can be traded in the open market or be redeemed back to DOT. Holding vDOT equals to holding the DOT staking position, staking rewards appreciate the exchange price of vDOT.

{% hint style="info" %}
Staking rewards automatically add to the vDOT exchange price, no manual claim. The longer vDOT postion holding, the greater amount of DOT will be exchanged back for redemption.
{% endhint %}

## Why vDOT?

{% hint style="success" %}
**Liquid Staking**
{% endhint %}

The product allows users to stake DOT for liquid vToken, (vDOT). vDOT will keep receiving **staking rewards** and can continue to be used in Bifrost and Polkadot-based DeFi for additional rewards.

{% hint style="success" %}
**Automatically Staking rewards capturing without scenario limitations**
{% endhint %}

SLP will issue Staking rewards to vDOT by adjusting the price of vDOT / DOT upwards. vDOT Rate = SLP Staking DOT (SUM) / vDOT Total Allocation.

{% hint style="success" %}
**Floating redemption period, vDOT redemption ≤ 28 days**
{% endhint %}

While Polkadot’s original chain Staking has a **fixed 28-day redemption period**, Bifrost SLP helps users to realize the possibility of early vDOT redemption by matching the real-time vDOT minting quantity with the redemption quantity at the protocol layer in the form of a queue. Theoretically, it can achieve second-level redemption.

{% hint style="success" %}
**Higher Staking Yield**
{% endhint %}

In the SLP protocol, the protocol screens more than **10 verified nodes** through governance (subsequently increasing with the overall staking volume) and adds multiple filters such as the **number of nominees**, **commission ratio**, and **history of blocks out to maximize the return of this verifier portfolio** while ensuring that none of the nodes have experienced slashes.

{% hint style="success" %}
**Multi-environment Compatibility**
{% endhint %}

vDOT is one of Substrate assets in Bifrost parachain, by using the [HRMP](https://wiki.polkadot.network/docs/learn-crosschain#xcmp-cross-chain-message-passing) channels between Bifrost and others, it can be easily utilized in EVM, WASM and Substarte competiable parachains.

## How it works?

vDOT is minted by Bifrost SLP pallet, so firstly users have to call XCM cross chain transfer DOT from Polkadot Relay to Bifrost Parachain.

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption><p>vDOT workflow</p></figcaption></figure>

### Mint vDOT

1. Users initiate a vDOT mint order, SLP protocol transfers DOT to DOT Ready Pool (which is an order pool accumulates all mint and redeem orders), SLP mints vDOT for users;
2. DOT Ready Pool matches Mint amount and Redeem amount;
3. Oracle monitor matching results from DOT Ready Pool and send messages to Polkadot Bifrost parachain sub-addresses;
4. Parachain sub-addresses execute Staking to SLP Polkadot validators set, Oracle queries the successful messages and send them back to Bifrost DOT Ready Pool, get ready for the next matching.

### Redeem vDOT

1. Users initiate a vDOT redeem orders to DOT Ready Pool;
2. DOT Ready Pool matches Redeem amount and Mint amount, and dispatches the remaining DOTs to vDOT redeem orders, SLP destroys the redeemed vDOT amount;
3. Oracle monitor redeem orders from Bifrost chain and send messages to Polkadot Bifrost parachain sub-addresses;
4. Parachain sub-addresses execute Redeem to SLP Polkadot validators set and send redeemed DOT back to Bifrost parachain, Oracle queries all these messages and send them back to Bifrost DOT Ready Pool, get ready for the next matching.

### DOT Reward

The DOT reward will be reinvested on the relay chain, and the Oracle will transmit the reward data to the DOT Ready Pool to adjust the vDOT exchange rate.

$$
vDOTrate=(StakedDOT+Staking Reward)/vDOT Allocation
$$

💡 Read more detailed info in the following sections.

***

## SLP-vDOT Architecture

### **Glossary**

#### DOT Ready Pool

The operation of staking and the settlement of related operations are generally not subject to real-time settlement, and there will be a cycle, which is [Era in Polkadot](https://wiki.polkadot.network/docs/maintain-polkadot-parameters). In order to improve resource efficiency and utilization, the **DOT Mint and Redeem requirements with related requirements are stored through the DOT Ready Poo**l. Wait until the appropriate time for unified processing.

#### Multi-sign Oracle

What actions are performed by the SLP module on the Bifrost chain, the settlement of exchange rates, etc., all need to obtain accurate relay chain data. Through multisig oracles, Polkadot's on-chain data can be fed to the Bifrost chain on a regular basis or under certain circumstances, so as to keep the SLP module running normally. The accuracy and authenticity of the data is the key to the security of the SLP protocol. In the future, **this part of the function should be implemented through XCM query**. At present, a more mature multi-signature scheme is used for the time being.

<details>

<summary>Oracle Multisig Address</summary>

1. bjbwPTJoPdtF96S2FEquuHoNv72T6KPzhayntu17Cr5g4BX
2. cEDk5LZS5UQtEEJnv1Sm1pDoTCDvv6rmChyF4A81sT9eCUZ
3. djJJCvG3s2Y4qZeTDS6RmmWdFSBHvcfTyJEDb2ARZKma9zA
4. eznovZYddE1KAvMc3ooBhvSS3JY6xFbii3QUCqpVEdnx5eg
5. gQCMN389RSi4gZwbG25agpyMPp1KQfPetteRU8TRd8tiQHu

Multisig synthetic address: emmdmYNhx3PMyuaQZi98ZSNPw6PhFnhGnXyX2NqLwWazck2

</details>

#### Aggregator Module

In order to improve the user experience of Redeem (more use of Swap for lightning swaps), this module is used to AMM (automatic market-maker) behavior and **simulate arbitrage to lower vDOT's Swap discount rate to a level similar to Redeem**. The profits generated in the process continue to carry out relevant market making operations/return to users. Through this module, the system can generate additional value to promote the healthy development of the SLP module.

#### Polkadot Staking Era

The settlement cycle of Polkadot staking operation is every 24 hours. At the end of the cycle, the **bonding / Unbonding operation** performed in the current cycle will be confirmed, and the **staking reward will be settled**. Committing operations at the end of the cycle is most efficient because the efficient execution of most operations in the cycle is processed uniformly at the end of the cycle.

#### Bifrost SLP Address

The DOT pledged by SLP and the related state are managed through the **sub-account of the parachain address**. Since there is an upper limit on the amount of DOT that can be pledged by an address under the condition of maximizing revenue, it should have the ability to configure multiple sub-addresses through on-chain governance.

#### vDOT Rate

SLP needs to issue Staking rewards to vDOT by raising the price of vDOT / DOT, **vDOT Rate = Staking total DOT / vDOT total issuance**. This exchange rate is the basis for a large number of operations of the system.

### How DOT Ready Pool works? (Adjusting vDOT Ratio)

DOT Ready Pool communicate with Oracle by receiving update status and data on Polkadot relay-chain in each Era.

#### In a New Era

When a new Era message is received by SLP, the system mainly performs the following operations:

* Send the cross-chain message withdraw Unbonded DOT and cross-chain transfer it to the DOT Ready Pool
* Get the new staking reward data in current era, a part of which is directly transferred to the Aggregator module across the chain according to the Bifrost service commission rate.
* After confirming that all current operations are successful, end the operation and issue a completion event

#### At the End of an Era

After receiving the Era End message, the system performs the following operations in sequence:

* Match Redeem demand orders one by one for DOT redemption in the order of submission, delete the redeemed demand orders, and transfer DOT back to the corresponding address (Redeem orders support partial redemption)
* When the Redeem match is over, judge the current DOT Ready Pool status:

{% hint style="info" %}
**Redeem:** vDOT revoking request on Bifrost side. \
**Unbond:** DOT revoking request on Polkadot side. \
**Rebond:** cancel DOT unbonding.
{% endhint %}

| DOT has leftovers                                                                                   | Initiate a cross-chain request to bond more funds with the remaining DOT on the relay chain          |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| All DOT have been matched, and the relay chain Unbonding state DOT ≥ Redeem demand order DOT total  | Initiate a cross-chain request to Rebond funds (Unbonding state DOT - Redeem demand list sum DOT)    |
| All DOTs have been matched, and the relay chain Unbonding state DOT < Redeem demand order DOT total | Initiate a cross-chain request to Unbond funds (Redeem demand list total DOT - Unbonding status DOT) |

* Calculate the current vDOT Rate = Staking DOT (SUM) / vDOT total issuance and update the record
* After confirming that the above operations are successfully executed, end the current Era operation and wait for the next cycle to start

### The Workflow of Aggregator Module

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>Aggregator Workflow</p></figcaption></figure>

#### 1. Check the DOT/vDOT exchange rate in Zenlink DEX

Get the exchange rate of DOT/vDOTin the DEX by query between modules at the end of the block.

#### 2. Calculate and construct the transaction

Based on the comparison between the queried DEX exchange rate and vDOT Rate, when the exchange rate in DEX < vDOT Rate, a transaction is triggered, and the transaction is constructed based on the number of DOTs existing in the current Aggregator Module, so that the DEX exchange rate after the transaction ≈ vDOT Rate. Initiate a transaction to get a discounted vDOT.

#### 3. Determine whether to redeem vDOT

The remaining DOT in the current module + the Redeem value requested by the current module < the set arbitrage loop maximum value (the arbitrage loop maximum value can be set and modified by governance). Then perform the Redeem operation on the newly acquired vDOT in the module (DOT Ready Pool will immediately process the Redeem order submitted by the Aggregator first)

#### 4. Excess vDOT is flushed into the liquidity pool

When the remaining DOT in the current module + the Redeem value requested by the current module > the set arbitrage cycle maximum value, vDOT will no longer be redeemed, but half of the balance of vDOT and DOT will be directly flushed into DEX's DOT - vDOT In the liquidity pool of the trading pair, obtain LP Token.

## vDOT Validator Set

## Selecting Conditions

There are several filters to select vDOT Validators:

* Validator address with on-chain identity
* No experienced Slashes
* Low commission
* Total Stake below average number
* Average Return Rate in 28 days
* Available nominator slot

## Selecting Organization

The selecting organization will evolute in 3 stages:

1. Bifrost Team&#x20;
2. Bifrost Council&#x20;
3. SLP automatic selection (current stage)

Validator selecting or replacement is a automatic process, it depends on a score ranking which is socred by a script:

****[**Validator Score Ranking**](https://docs.google.com/spreadsheets/d/13YdCMc\_bVdmy5My22mWRTC5ITwVEaRZQi4fIZGr8tSw/edit#gid=628365504)****

****[**The Script**](https://bifrost-api-test.vercel.app/api/dapp/staking/\_refresh)****

## Current Validator Set

Check for live Validator Set:

vDOT: [https://bifrost.app/vstaking/vDOT?tab=validators](https://bifrost.app/vstaking/vDOT?tab=validators)
