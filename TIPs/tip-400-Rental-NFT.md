# Ternoa Improvement Proposal - Rent

| Author(s)      | Victor Salomon |
| ----------- | ----------- |
| Created   | 14 Sep 2022       |
| TIP Number   | TIP-400 : NFT Rental      |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |

## Simple Summary
NFTs represent proof of ownership on the blockchain. Renting is a powerful tool for the NFT owners to make more out of their NFTs. An NFT rental agreement is an on chain rental contract, between the owner of an NFT and a renter (usually called rentee) who will have a temporary access to the NFT properties, against a rent fee (an amount or an NFT).

## Abstract
Rental Contract are a way to spread the use of an NFT and earning money without loosing its ownership. An NFT owner can set his NFT as a rental contract and start renting it to anyone. Rental contract are mostly customizable to cover most common rental usecases. 

## Motivation
Rental contracts have so many usecases in the real world that could be adapted to the NFTs. While the main purpose of renting an NFT is to easily empower any projects to get the best utility of an NFT by spreading its features to everyone, it is also a great way to cover many real world sucessful usecases. And if rental NFT are already popular in the gaming industry, the tractions it gains in many other filed is truely relevant to our web 3 ecosystem (events-tickets, music, sport, subscription...).

## Specification

### Lifecycle states

Rental NFTs will have the following lifecycle associated with them:
Contract Created -> Contract Rented -> *Contract Updated* * -> Contract Ended/Revoked.

*Optionnal in the Rental NFTs lifecycle.

### External interfaces

Rental NFT should support the following onchain interfaces:

```rust
interface {
  /// Interface Id: TIP400-01
  /// Description: The Caller creates a renting contract out of his NFT.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  create_contract(nft_id: NFTId, duration: Duration, acceptance_type: AcceptanceType, renter_can_revoke: bool,  rent_fee: RentFee, renter_cancellation_fee: Option<CancellationFee>, rentee_cancellation_fee: Option<CancellationFee>);
  
  /// Interface Id: TIP400-02
  /// Description: The Caller revokes a running contract.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  revoke_contract(nft_id: NFTId);

  /// Interface Id: TIP400-03
  /// Description: The Caller cancel a non-running contract.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  cancel_contract(nft_id: NFTId);

  /// Interface Id: TIP400-04
  /// Description: The Caller accepts a existing non-running contract.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  rent(nft_id: NFTId);

  /// Interface Id: TIP400-05
  /// Description: The Caller creates a offer for an existing non-running contract.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  make_rent_offer(nft_id: NFTId);

  /// Interface Id: TIP400-06
  /// Description: The Caller accepts a offer for an existing non-running contract.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  accept_rent_offer(nft_id: NFTId);

  /// Interface Id: TIP400-07
  /// Description: The Caller removes his offer.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  retract_rent_offer(nft_id: NFTId)

  /// Interface Id: TIP400-08
  /// Description: The Caller changes the subscription terms.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  change_subscription_terms(nft_id: NFTId, period: BlockNumber, max_duration: Option<BlockNumber>, rent_fee: Balance, changeable: bool)

  /// Interface Id: TIP400-09
  /// Description: The Caller accepts the changed subscription terms.
  /// Constraint(s): Refer to section 'Rules'
  /// Result(s): Refer to section 'Results'
  accept_subscription_terms(nft_id: NFTId)
}
```

### Rules and constraints
#### Create Contract Interface
- Provided NFT MUST be owned by the Caller.
- Provided NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- Provided Rent NFT Fee MUST exist.
- Provided Renter Cancellation NFT Fee MUST be owned by the Caller.
- Provided Renter Cancellation Token Fee MUST be less then the free balance of Caller.
- Provided Rentee Cancellation NFT Fee MUST exist.
#### Revoke Contract Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST be running.
- The Caller MUST be a contract participant (either the renter or rentee).
- If the Caller is the Contract owner, the Contract MUST allow for renter to revoke contracts.
#### Cancel Contract Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST NOT be running.
- The Caller MUST be the owner of the contract.
#### Rent Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST NOT be running.
- The Contract MUST have the acceptance_type set to Automatic
- The Caller MUST NOT be the owner of the contract.
- If a whitelist exists, the Caller MUST be whitelisted.
- If the Rent Fee is an NFT, the Caller MUST own that NFT.
- If the Rent Fee is an NFT, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If the Rent Fee is of Token type, the Caller MUST have enough balance to pay for it.
- If a Cancellation NFT Fee exists, the Caller MUST own that NFT.
- If a Cancellation NFT Fee exists, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If a Cancellation Token Fee exists, the Caller MUST have enough balance to pay for it.
#### Make Rent Offer Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST NOT be running.
- The Contract MUST have the acceptance_type set to Manual
- The Caller MUST NOT be the owner of the contract.
- If a whitelist exists, the Caller MUST be whitelisted.
- If the Rent Fee is an NFT, the Caller MUST own that NFT.
- If the Rent Fee is an NFT, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If the Rent Fee is of Token type, the Caller MUST have enough balance to pay for it.
- If a Cancellation NFT Fee exists, the Caller MUST own that NFT.
- If a Cancellation NFT Fee exists, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If a Cancellation Token Fee exists, the Caller MUST have enough balance to pay for it.
#### Accept Rent Offer Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST NOT be running.
- The Contract MUST have the acceptance_type set to Manual
- The Caller MUST be the owner of the contract.
- If the Rent Fee is an NFT, the Offer-Owner MUST own that NFT.
- If the Rent Fee is an NFT, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If the Rent Fee is of Token type, the Offer-Owner MUST have enough balance to pay for it.
- If a Cancellation NFT Fee exists, the Offer-Owner MUST own that NFT.
- If a Cancellation NFT Fee exists, the NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- If a Cancellation Token Fee exists, the Offer-Owner MUST have enough balance to pay for it.
#### Retract Rent Offer Interface
- There MUST be a offer-queue for the provided NFT.
- The Caller MUST have an offer for that provided NFT.
#### Change Subscription Terms Interface
- There MUST be a contract for the provided NFT.
- The Caller MUST be the owner of the contract.
- The Contract MUST be of type subscription.
- The Contract MUST allow for changes in subscription terms.
#### Accept Subscription Terms Interface
- There MUST be a contract for the provided NFT.
- The Contract MUST be running.
- The Caller MUST be rentee of the contract.
- The Contract subscription terms MUST have been changed.

### Duration, Rent Fee and Cancellation Fee
- Duration of type Subscription MUST NOT use a NFT Rent Fee.
- Only Duration of type Fixed MAY use the Flexible Token Cancellation Fee

### Results
#### Create Contract Interface
- Provided NFT MUST be in state `Rented`.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Caller and send to a escrow account.
#### Revoke Contract Interface
- Provided NFT MUST NOT be anymore in state `Rented`.
- The existing contract MUST be burned.
- Provided Caller Cancellation Fee MUST be given to the the damaged party.
- Provided Damaged Party Cancellation Fee MUST be returned to the damaged party.
#### Cancel Contract Interface
- Provided NFT MUST NOT be anymore in state `Rented`.
- The existing contract MUST be burned.
- Provided Cancellation Fee MUST be given back to the Caller.
#### Rent Interface
- The existing contract MUST contain the Caller's address as Rentee.
- The existing contract MUST contain the start block.
- Provided Rent NFT Fee MUST change its ownership to the contract owner address.
- Provided Rent NFT Token Fee MUST be taken from Caller and send to contract owner address.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Caller and send to a escrow account.
#### Make Rent Offer Interface
- Offers related to the contract MUST contain the Callers address.
#### Accept Rent Offer Interface
- The existing contract MUST contain the Offer-Owner's address as Rentee.
- The existing contract MUST contain the start block.
- Provided Rent NFT Fee MUST change its ownership to the contract owner address.
- Provided Rent NFT Token Fee MUST be taken from Offer-Owner and send to contract owner address.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Offer-Owner and send to a escrow account.
#### Retract Rent Offer Interface
- Offers related to the contract MUST NOT contain the Callers address.
#### Change Subscription Terms Interface
- Contract MUST be updated with the new subscription and rent values
- Contract MUST be marked as changed.
#### Accept Subscription Terms Interface
- Contract MUST be marked as not-changed.

### Rented State
If an NFT is in Rented state it means that the NFT renter cannot call any extrinsic (this includes but is not limited to: Transfer, Burn, List, Auction,...) on it besides the `cancel_contract` or `revoke_contract`.
If an NFT is in Rented state it means that the NFT rentee cannot call any extrinsic (this includes but is not limited to: Transfer, Burn, List, Auction,...) on it besides the `revoke_contract`.

## Metadata
No Metadata

## End-to-end workflow (Ternoa-specific)
### Create Contract
Prerequisites: The User owns an NFT that is in the right state (NOT Capsule, ListedForSale,  Delegated, Soulbound, Rented).
#### Flow
1. The User decides if he wants to have a Fixed Term or a Subscription Term contract.
2. The user decides if he wants to use auto acceptance (the first offer is immediately accepted) or manual acceptance (the users chooses which offer to accept).
3. The User decides if he wants to introduce a whitelist where only certain accounts can send offers or rent.
4. The User decides if he wants to have a contract where he can or cannot cancel it.
5. The User decides if he wants for the Subscription Term to be changeable or not.
6. The User decides if he wants to specify a Token based or NFT based Rent Fee.
7. The User decides if he wants to specify the Renter cancellation fee. It can be either Token based, NFT based or Token-Flexible based.
8. The User decides if he wants to specify the Rentee cancellation fee. It can be either Token based, NFT based or Token-Flexible based.
9. The User calls the `create_contract` interface and then the contract is created.

### Revoke Contract
Prerequisites: A running contract already exists for the observed NFT.
#### Flow
1. The User decides if it is acceptable to lose the cancellation fees.
2. The User calls the `revoke_contract` interface and then the contract is revoked.

### Cancel Contract
Prerequisites: A non-running contract already exists for the observed NFT.
#### Flow
1. The User decides if he doesn't want to make his NFT available for rent.
2. The User calls the `cancel_contract` interface and then the contract is canceled.

### Rent Contract
Prerequisites: A non-running contract already exists for the observed NFT.
#### Flow
1. The User decides if he wants to rent the observed NFT.
2. The User decides if the rent fee requirement is acceptable.
3. The User decides if the cancellation fee requirement is acceptable.
4. The User decides if the contract as a whole is acceptable.
5. The User calls the `rent` interface and then the contract is signed and active and he "get's" the NFT.

### Make Rent Offer
Prerequisites: A non-running contract already exists for the observed NFT.
#### Flow
1. The User decides if he wants to rent the observed NFT.
2. The User decides if the rent fee requirement is acceptable.
3. The User decides if the cancellation fee requirement is acceptable.
4. The User decides if the contract as a whole is acceptable.
5. The User calls the `make_rent_offer` interface and then waits for the offer to be accepted or declined.

### Accept Rent Offer
Prerequisites: A non-running contract already exists for the observed NFT. Offers exists for that contract.
#### Flow
1. The User decides if he wants to accept a existing offer.
2. The User calls the `accept_rent_offer` interface and then the NFT becomes rented.

### Retract Rent Offer
Prerequisites: A non-running contract already exists for the observed NFT. Offers exists for that contract.
#### Flow
1. The User decides if he wants to retract his existing offer.
2. The User calls the `retract_rent_offer` interface and then his offer is removed.

### Change Subscription Terms
Prerequisites: A Contract already exists for the observed NFT. 
#### Flow
1. The User decides if he wants to change the current subscription terms.
2. The User calls the `change_subscription_terms` interface and then waits for the rentee to either accept it or decline it.

### Accept Subscription Terms
Prerequisites: A Contract already exists for the observed NFT. 
#### Flow
1. The User decides if he wants to accept the changes to the subscription terms.
2. The User calls the `accept_subscription_terms` interface and then he will pay the new price starting from the next period.

## Test cases

* NFT Owner can create a Rental Contract
* Rentee can rent contract directly 
* Rentee can make an offer on a contract
* Rentee can remove the contract offer
* Renter can accept an offer
* Renter can update subscription terms contract
* Rentee can accept new subscritpion terms contract
* Rentee can revoke contract
* Renter can revoke contract (if allowed and if not allowed)
* Renter can cancel contract
 
## References
TBD

## Copyright
TBD