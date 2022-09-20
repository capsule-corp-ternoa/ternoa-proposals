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
  create_contract(nft_id: NFTId, duration: Duration, acceptance_type: AcceptanceType, renter_can_cancel: bool,  rent_fee: RentFee, renter_cancellation_fee: Option<CancellationFee>, rentee_cancellation_fee: Option<CancellationFee>);
  
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
#### Create Contract
- Provided NFT MUST be owned by the Caller.
- Provided NFT MUST NOT be in the following states: Capsule, ListedForSale, Delegated, Soulbound, Rented.
- Provided Rent NFT Fee MUST exist.
- Provided Renter Cancellation NFT Fee MUST be owned by the Caller.
- Provided Renter Cancellation Token Fee MUST be less then the free balance of Caller.
- Provided Rentee Cancellation NFT Fee MUST exist.
#### Revoke Contract
- There MUST be a contract for the provided NFT.
- The Contract MUST be running.
- The Caller MUST be a contract participant (either the renter or rentee).
#### Cancel Contract
- There MUST be a contract for the provided NFT.
- The Contract MUST NOT be running.
- The Caller MUST be the owner of the contract.
#### Rent
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
#### Make Rent Offer
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
#### Accept Rent Offer
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
#### Retract Rent Offer
- There MUST be a offer-queue for the provided NFT.
- The Caller MUST have an offer for that provided NFT.
#### Change Subscription Terms
- There MUST be a contract for the provided NFT.
- The Caller MUST be the owner of the contract.
- The Contract MUST be of type subscription.
- The Contract MUST allow for changes in subscription terms.
#### Accept Subscription Terms
- There MUST be a contract for the provided NFT.
- The Contract MUST be running.
- The Caller MUST be rentee of the contract.
- The Contract subscription terms MUST have been changed.

### Results
#### Create Contract
- Provided NFT MUST be in state `Rented`.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Caller and send to a escrow account.
#### Revoke Contract
- Provided NFT MUST NOT be anymore in state `Rented`.
- The existing contract MUST be burned.
- Provided Caller Cancellation Fee MUST be given to the the damaged party.
- Provided Damaged Party Cancellation Fee MUST be returned to the damaged party.
#### Cancel Contract
- Provided NFT MUST NOT be anymore in state `Rented`.
- The existing contract MUST be burned.
- Provided Cancellation Fee MUST be given back to the Caller.
#### Rent
- The existing contract MUST contain the Caller's address as Rentee.
- The existing contract MUST contain the start block.
- Provided Rent NFT Fee MUST change its ownership to the contract owner address.
- Provided Rent NFT Token Fee MUST be taken from Caller and send to contract owner address.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Caller and send to a escrow account.
#### Make Rent Offer
- Offers related to the contract MUST contain the Callers address.
#### Accept Rent Offer
- The existing contract MUST contain the Offer-Owner's address as Rentee.
- The existing contract MUST contain the start block.
- Provided Rent NFT Fee MUST change its ownership to the contract owner address.
- Provided Rent NFT Token Fee MUST be taken from Offer-Owner and send to contract owner address.
- Provided Renter Cancellation NFT Fee MUST change its ownership to escrow account.
- Provided Renter Cancellation Token Fee MUST be taken from the Offer-Owner and send to a escrow account.
#### Retract Rent Offer
- Offers related to the contract MUST NOT contain the Callers address.
#### Change Subscription Terms
- Contract MUST be updated with the new subscription and rent values
- Contract MUST be marked as changed.
#### Accept Subscription Terms
- Contract MUST be marked as not-changed.

## Metadata
No Metadata

## End-to-end workflow (Ternoa-specific)
The following is the workflow proposed for creating a contract: 
1. User already own an NFT and know its ID.
2. User create the Rent Contract for the NFT ID required by passing a Duration type (Fixed, Subscription or Infinite), an Acceptance type (Auto or Manual / optionnal whitelist), a Revocation type (NoRevocation, OnSubscriptionChange or Anytime), a RentFee (Tokens or NFT), Optionnal Renter Cancellation Fee (Fixed or Flexible tokens, or NFT), Optionnal Rentee Cancellation Fee (Fixed or Flexible tokens, or NFT).
3. Rental Contract is created, user can find it with the NFT ID. 

The following is the workflow proposed for making an offer on a contract: 
1. User (as Rentee) find a NFT Contract he wants to rent with a Manual Acceptance.
2. User is whitelisted in the Manual Acceptance List or there is no Manual Acceptance List at all.
3. User send a rent extrinsic with the NFT ID he wants to rent. 
4. Offer is sent. User can remove his offer until Renter accepte it. 
5. Renter can accepte this offer or any other offer received. 

The following is the workflow proposed for directly renting a contract : 
1. User (as Rentee) find a NFT Contract he wants to rent with an Automatic Acceptance.
2. User is whitelisted in the Automatic Acceptance List or there is no Automatic Acceptance List at all.
3. User send a rent extrinsic with the NFT ID he wants to rent. 
4. Offer is sent : Contract Started 
5. User can start benefit from the NFT properties.

The following is the workflow proposed for updating a subscription terms contract :
1. User already own an NFT and know its ID.
2. User create the Rent Contract for the NFT ID required by passing a Subscription Duration type, an Acceptance type (Auto or Manual / optionnal whitelist), an OnSubscriptionChange Revocation type a RentFee (Tokens or NFT), Optionnal Renter Cancellation Fee (Fixed or Flexible tokens, or NFT), Optionnal Rentee Cancellation Fee (Fixed or Flexible tokens, or NFT).
3. Rental Contract is created.
2. User (as Rentee) find the NFT Contract he wants to rent with a Subscription Duration.
4. User send a rent extrinsic with the NFT ID he wants to rent. 
5. Offer is sent : Contract Started if Auto Acceptance/ Contract needs to be aproved. 
6. Contract started.
7. Rentee can start benefit from the NFT properties.
8. After a certain time, Renter wants to update the contract terms. 
9. He sends a changeSubscriptionTerms extrinsic with the NFT ID, the subscription Duration type with the the new values (or the same if he wants the duration to stay as it is), the new amount (or the same if he wants the amount to stay as it is).
10. Rentee has until the end of the current duration periode to accept the new contract terms.
11. If Terms are accepted by the rentee before the end of the current duration periode. The contract is renewed with the new subscription values. 
12. if Terms are not accepted before the end of the current duration periode, the contract ended (contract ended event will retrun a revokedBy value equal to 'renter address')

The following is the workflow proposed for revoking a contract: 
1. A contract already started according to one of the following scenario below.
If Renter want to revoke the contract. 
3.Contract has not started yet : renter can send a revokeContract extrinsic with the NFT ID.
4. If contract has started, but Revocation Type is NoRevocation : Renter can not revoke contract. Only Rentee can revoke it.
5. Else, renter can send a revokeContract extrinsic with the NFT ID.
6. If RenterCancellationFee are set, when revoking contract, renter will pay the fees/nft.
If Rentee want to revoke the contract.
3.Rentee can send a revokeContract extrinsic with the NFT ID.
4.If RenteeCancellationFee are set, when revoking contract, rentee will pay the fees/nft.

## Test cases

* NFT Owner can create a Rental Contract
* Rentee can rent contract directly 
* Rentee can make an offer on a contract
* Rentee can remove the contract offer
* Renter can accept an offer
* Renter can update subscription terms contract
* Rentee can accept new subscritpion terms contract
* Renter or Rentee can revoke contract
 
## References
TBD

## Copyright
TBD