# vGLMR & vMOVR

{% hint style="info" %}
vMOVR and vGLMR are implemented by the same logic and the same functions, but deploy in BIfrost Kusama and Polkadot parachain. Thus, this mechanism explanation takes vMOVR and vGLMR as totally the same thing to demonstrate.
{% endhint %}

## **What is vMOVR?**

vMOVR (voucher MOVR) is a shadow token of staked MOVR, with fully underlying MOVR reserve and yield-bearing feature of MOVR staking reward. Users can deposit MOVR into Bifrost SLP protocol and get vMOVR as return, vMOVR can be traded in the open market or be redeemed back to MOVR. Holding vMOVR equals to holding the MOVR staking position, delegating rewards appreciate the exchange price of vMOVR.

{% hint style="info" %}
Delegating rewards automatically add to the vMOVR exchange price, no manual claim. The longer vMOVR postion holding, the greater amount of MOVR will be exchanged back for redemption.
{% endhint %}

## Why vMOVR?

{% hint style="success" %}
**Liquid Staking**
{% endhint %}

The product allows users to delegate MOVR for liquid vToken, (vMOVR). vMOVR will keep receiving staking rewards and can continue to be used in Bifrost and Kusama-based DeFi for additional rewards.

{% hint style="success" %}
**Automatically Staking rewards capturing without scenario limitations**
{% endhint %}

SLP will issue Staking rewards to vMOVR by adjusting the price of vMOVR / MOVR upwards. vMOVR Rate = SLP Staking MOVR (SUM) / vMOVR Total Issuance.

{% hint style="success" %}
**Floating redemption period, vMOVR redemption ≤ 48 hours**
{% endhint %}

While Moonriver’s original chain Staking has a fixed 24 rounds (about 48 hours) revoke period, Bifrost SLP helps users to realize the possibility of **early vMOVR redemption** by matching the real-time vMOVR minting quantity with the redemption quantity at the protocol layer in the form of a queue.

{% hint style="success" %}
**Higher Staking Yield**
{% endhint %}

In the SLP protocol, the protocol screens more than **10 verified nodes** through governance (subsequently increasing with the overall staking volume) and adds multiple filters such as the **number of nominees and** **history of blocks out to maximize the return of this verifier portfolio** while ensuring that none of the nodes have experienced slashes.

{% hint style="success" %}
**Multi-environment Compatibility**
{% endhint %}

vMOVR is one of Substrate assets in Bifrost parachain, by using the [HRMP](https://wiki.polkadot.network/docs/learn-crosschain#xcmp-cross-chain-message-passing) channels between Bifrost and others, it can be easily utilized in EVM, WASM and Substarte competiable parachains.

## How it works?

vMOVR is minted by Bifrost SLP pallet, so firstly users have to call XCM cross chain transfer MOVR from Moonriver to Bifrost Parachain.

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>vMOVR workflow</p></figcaption></figure>

### Mint vMOVR

1. Users initiate a vMOVR mint order, SLP protocol transfers MOVR to MOVR Ready Pool (which is an order pool accumulates all mint and redeem orders), SLP mints vMOVR for users;&#x20;
2. MOVR Ready Pool matches Mint amount and Redeem amount;&#x20;
3. Oracle monitor matching results from MOVR Ready Pool and send messages to Moonriver-Bifrost SLP addresses;&#x20;
4. Bifrost SLP addresses execute Staking to SLP Moonriver Collator set, Oracle queries the successful messages and send them back to Bifrost MOVR Ready Pool, get ready for the next matching.

**About SLP Oracle/backend service**: The SLP backend service is using ⅘ multisig nodes, responsible for adjusting the exchange rate in each round, returning the due amount to the current redeem due user, checking the ledger, summarizing the pledge amount in a new round, and calling the corresponding SLP module function. In order to more flexibly handle the increase or decrease of users' pledge in each round, we will adopt a decentralized delegation relationship and a small-amount pledge strategy to increase the efficiency of the use of funds.

### Redeem vMOVR

1. Users initiate a vMOVR redeem orders to MOVR Ready Pool;&#x20;
2. MOVR Ready Pool matches Redeem amount and Mint amount, and dispatches the remaining MOVR to vMOVR redeem orders, SLP destroys the redeemed vMOVR amount;&#x20;
3. Oracle monitor redeem orders from Bifrost chain and send messages to Moonriver-Bifrost SLP addresses;&#x20;
4. Bifrost SLP addresses execute Redeem to SLP Moonriver collators set and send redeemed MOVR back to Bifrost parachain, Oracle queries all these messages and send them back to Bifrost MOVR Ready Pool, get ready for the next round matching.

### MOVR Reward

The MOVR reward will be reinvested on Moonriver, and the Oracle will transmit the reward data to the MOVR Ready Pool to adjust the vMOVR exchange rate.

$$
vMOVR rate=(StakedMOVR+StakingReward)/vMOVR Allocation
$$

💡 Read more detailed info in the following sections.

***

## SLP-vMOVR Technology Implementation

### Overall operation process

The entire SLP protocol includes three parts, the Vtoken-Minting module, the SLP module, and the backend service.

The user can call the mint/redeem/rebond method of the VtokenMinting module through the front end to mint MOVR into vMOVR, or exchange vMOVR back to MOVR. During the period of holding vMOVR, users can enjoy staking benefits, which are reflected in the exchange rate of vMOVR to MOVR. If the user has new pledge amounts in the pool during the redeem period, the user will be able to experience the process of fast redemption, and the new pledge amount will be returned to the users in front of the redeem queue first.

The SLP module of Bifrost is responsible for communicating with the ParachainStaking module of the Moonriver chain by sending cross-chain messages to perform operations such as delegate, delegate\_bond\_more, schedule\_delegator\_bond\_less, schedule\_revoke\_delegation, execute\_delegation\_request, cancel\_delegation\_request, schedule\_leave\_delegators, execute\_leave\_delegators, cancel\_leave\_delegators. The delegator account is generated by Bifrost's parachain address on the Moonriver chain, and the corresponding sub-account is generated by calling the as\_derivative function of the utility module. All delegator and validator addresses are stored and used in the format of MultiLocation in the module.

### Backend service function

The SLP backend service is responsible for adjusting the exchange rate in each round, returning the due amount to the current redeem due user, checking the ledger, summarizing the pledge amount in a new round, and calling the corresponding SLP module function. In order to more flexibly handle the increase or decrease of users' pledge in each round, we will adopt a decentralized delegation relationship and a small-amount pledge strategy to increase the efficiency of the use of funds.

#### At the start of Round

1. Update round numbers
2. Charge the custody fee, add interest (according to the dividend event, dividends before n+1 period), modify the exchange rate, and transfer the interest amount back to the exit account.
3. Withdrawal of redemption due amount
4. Repay the current due debt and transfer the remaining amount to the entry account for quick redemption.
5. Query and compensate the reserve amount of handling fees for each operating account.

#### Before Round ends

1. Transfer the excess amount in the entry account to a different delegate, and perform the pledge operation. When pledging, pay attention to check if the ranking is in the bottom delegation. If so, add the delegating amount, or transfer the amount to another collator.
2. Check the ledger, warn and adjust accordingly if there are discrepancies.

#### Attention for staking operations:

1. The staking amount is distributed among multiple delegation relationships of multiple delegates in a relatively even manner to facilitate each round of bond and redeem fund operations and improve the efficiency of fund use.
2. Don't let the delegates fall into the bottom delegation group of collators. It is necessary to maintain a real-time top lowest value of collator. In theory, each account should try to keep a certain percentage higher than this value, such as 15%.
3. When comparing the difference in the number of redeems between the Bifrost chain and the Moonriver chain, the number on the Moonriver side should be greater than or equal to the number on the Bifrost side

### Vtoken-Minting module

The user can call the mint/redeem/rebond method of the VtokenMinting module through the front end to mint MOVR into vMOVR, or exchange vMOVR back to MOVR. During the period of holding vMOVR, users can enjoy staking benefits, which are reflected in the exchange rate of vMOVR to MOVR. If the user has new pledge amounts in the pool during the redeem period, the user will be able to experience the process of fast redemption, and the new pledge amount will be returned to the users in front of the redeem queue first.

### Storages

**pub type TokenPool = StorageMap\<CurrencyIdOf, BalanceOf>**

Record the total amount of MOVR participating in staking.

**pub type OngoingTimeUnit = StorageMap\<CurrencyIdOf, TimeUnit>**

The current round of the Moonriver ParachainStaking module in progress.

**pub type TokenUnlockLedger = StorageDoubleMap\<CurrencyIdOf, UnlockId, (T::AccountId, BalanceOf, TimeUnit)>**

Record user redeem information for each CurrencyId.

**pub type UserUnlockLedger = StorageDoubleMap\<AccountIdOf, CurrencyIdOf, (BalanceOf, BoundedVec\<UnlockId, T::MaximumUnlockIdOfUser>)>**

Record the MOVR information of each user redeem.

**pub type TimeUnitUnlockLedger = StorageDoubleMap\<TimeUnit, Blake2\_128Concat, CurrencyIdOf, (BalanceOf, BoundedVec\<UnlockId, T::MaximumUnlockIdOfTimeUnit>, CurrencyIdOf)>**

Record all the user redeemed MOVR information of the for each round.

**pub type CurrencyUnlockingTotal = StorageValue\<BalanceOf>**

Record the total number of redeems for MOVR.

### Functions

**fn mint(origin: OriginFor, token\_id: CurrencyIdOf, token\_amount: BalanceOf)**

Users mint vMOVR from MOVR according to the real-time exchange rate.

**fn redeem(origin: OriginFor, vtoken\_id: CurrencyIdOf, mut vtoken\_amount: BalanceOf)**

Users redeem vMOVR for MOVR according to the real-time exchange rate.

**pub fn rebond(origin: OriginFor, token\_id: CurrencyIdOf, token\_amount: BalanceOf)**

The user can rebond into vMOVR any amount but no more than the submitted redeeming value of MOVR according to the real-time exchange rate

**pub fn rebond\_by\_unlock\_id(origin: OriginFor, token\_id: CurrencyIdOf, unlock\_id: UnlockId)**

The user rebonds the full redeeming amount under an unlock id into vMOVR according to the real-time exchange rate.

#### Notice

1. Add a query interface (from the SLP module), if all delegates have already requested bond less, the round is not allowed to receive user unbonds.

## SLP module

The SLP module is responsible for communicating with the ParachainStaking module of the Moonriver chain by sending cross-chain messages to perform operations such as delegate, delegate\_bond\_more, schedule\_delegator\_bond\_less, schedule\_revoke\_delegation, execute\_delegation\_request, cancel\_delegation\_request, schedule\_leave\_delegators, execute\_leave\_delegators, cancel\_leave\_delegators. The delegator account is generated by Bifrost's parachain address on the Moonriver chain, and the corresponding sub-account is generated by calling the as\_derivative function of the utility module. All delegator and validator addresses are stored and used in the format of MultiLocation in the module.

### Data structure

`pub struct OneToManyLedger<DelegatorId, ValidatorId, Balance> {`\
&#x20; `pub account: DelegatorId,`\
&#x20; `pub delegations: BTreeMap<ValidatorId, Balance>,`\
&#x20; `pub total: Balance,` \
&#x20; `pub less_total: Balance,` \
&#x20; `// request details.` \
&#x20; `pub requests: Vec<OneToManyScheduledRequest<ValidatorId, Balance>>,` \
&#x20; `// fast check if request exists` \
&#x20; `pub request_briefs: BTreeMap<ValidatorId, (TimeUnit, Balance)>,` \
&#x20; `pub status: OneToManyDelegatorStatus,` \
&#x20;`}`

delegator ledger format

`pub enum OneToManyDelegatorStatus {`\
&#x20; `Active,`\
&#x20; `Leaving(TimeUnit),`\
`}`

delegator status format

`pub enum OneToManyDelegationAction {` \
&#x20; `Revoke(Balance),` \
&#x20; `Decrease(Balance),` \
`}`

delegation action format

`pub enum TimeUnit {` \
&#x20; `// Kusama staking time unit` \
&#x20; `Era( u32),` \
&#x20; `// Moonriver staking time unit` \
&#x20; `Round(u32),` \
`}`

Moonriver’s round format

### Storages

**pub type DelegatorsIndex2Multilocation = StorageDoubleMap\<CurrencyId, u16, MultiLocation>**

Delegator index ⇒ delegator multilocaition

**pub type DelegatorsMultilocation2Index = StorageDoubleMap\<CurrencyId, MultiLocation, u16>**

Delegator multilocaition ⇒ delegator index

**pub type DelegatorNextIndex = StorageMap\<CurrencyId, u16>**

The next delegator index of CurrencyId

**pub type Validators = StorageMap\<CurrencyId, Vec<(MultiLocation, Hash)>>**

Validators whitelist set

**pub type DelegatorLedgers = StorageDoubleMap\<CurrencyId, MultiLocation, Ledger\<MultiLocation, BalanceOf, MultiLocation>>**

Delegator's ledger details

**pub type DelegatorLedgerXcmUpdateQueue = StorageMap\<QueryId， (LedgerUpdateEntry\<BalanceOf, MultiLocation, MultiLocation>, BlockNumberFor)>**

After waiting for the cross-chain message receipt to be obtained, it needs to be updated to the information queue of the delegate ledger.

### Functions

**fn initialize\_delegator(origin: OriginFor,currency\_id: CurrencyId)**

Function: Initialize the delegate sub-account

**fn bond( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, amount: BalanceOf, validator: Option )**

Function: Add the delegation relationship between delegate ↔ validator, and call the delegate function on Moonriver across the chain.

**fn bond\_extra( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, validator: Option, amount: BalanceOf )**

Function: Increase the pledge amount on the existing delegation relationship, and call the delegate\_bond\_more function on Moonriver across the chain.

**fn unbond( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, validator: Option, amount: BalanceOf)**

Function: Reduce the pledge amount on the existing delegation relationship, and call the schedule\_delegator\_bond\_less function on Moonriver across the chain.

**fn unbond\_all( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation)**

Function: Leave the delegator set and call the schedule\_leave\_delegators function on Moonriver across the chain.

**fn rebond( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, validator: Option, amount: BalanceOf )**

Function: cancel the request of the schedule, and call the cancel\_delegation\_request function on the Moonriver across the chain.

**fn undelegate( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, targets: Vec )**

Function: withdraw the delegation relationship with a validator, and call the schedule\_revoke\_delegation function on Moonriver across the chain.

**fn redelegate( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, targets: Vec )**

Function: cancel the request to leave the delegator set, and call the cancel\_leave\_delegator function on Moonriver across the chain.

**fn liquidize( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation, when: Option, validator: Option )**

Function: Execute the expired request or the request of leave from the delegate set, and call the execute\_leave\_delegators or execute\_delegation\_requests function on the Moonriver across the chain.

**fn chill( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation )**

Function: same as unbond\_all. Leave the delegator set and call the schedule\_leave\_delegators function on Moonriver across the chain.

**fn transfer\_back( origin: OriginFor, currency\_id: CurrencyId, from: Box, to: Box, amount: BalanceOf )**

Function: transfer MOVR from the Moonriver chain, and call the xTokens module on Moonriver across the chain.

**fn transfer\_to( origin: OriginFor, currency\_id: CurrencyId, from: Box, to: Box, amount: BalanceOf )**

Function: transfer MOVR from the designated account of Bifrost to the delegator on the Moonriver chain, and call the xToken module on the Bifrost chain.

**fn update\_ongoing\_time\_unit( origin: OriginFor, currency\_id: CurrencyId, time\_unit: TimeUnit)**

Function: Synchronize the round information on the Moonriver chain.

**fn refund\_currency\_due\_unbond( origin: OriginFor, currency\_id: CurrencyId )**

Function: Repay the expired redeem in current round.

**fn supplement\_fee\_reserve( origin: OriginFor, currency\_id: CurrencyId, dest: MultiLocation )**

Function: Supplement the handling fee reserve of the operating account

**fn charge\_host\_fee\_and\_tune\_vtoken\_exchange\_rate( origin: OriginFor, currency\_id: CurrencyId, value: BalanceOf, who: MultiLocation )**

Function: collect pledge escrow fees from interest, and adjust the exchange rate between VMOVR and MOVR

**fn add\_delegator( origin: OriginFor, currency\_id: CurrencyId, index: u16, who: MultiLocation )**

Function: Add a delegate.

**fn remove\_delegator( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation )**

Function: Reduce a delegate.

**fn add\_validator( origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation )**

Function: Add a validator to the whitelist.

**fn remove\_validator(origin: OriginFor, currency\_id: CurrencyId, who: MultiLocation )**

Function: Reduce a validator to the whitelist.

**fn confirm\_delegator\_ledger\_query\_response( origin: OriginFor, currency\_id: CurrencyId, query\_id: QueryId )**

Function: After receiving the receipt of the successful execution of the cross-chain message, call this function to change the ledger of the corresponding delegator, and delete the change information in the temporary queue.

**fn fail\_delegator\_ledger\_query\_response( origin: OriginFor, currency\_id: CurrencyId, query\_id: QueryId )**

Function: After receiving the receipt of the failure of cross-chain message execution, no corresponding processing is performed, and the change information in the temporary queue is deleted.
