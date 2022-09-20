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
  /// Description: A NFT owner can create a rental contract with his NFT.
  /// Constraint(s): Refer to section 'Rules'
  
  create_contract(nft_id: NFTId, duration: Duration, acceptance_type: AcceptanceType, rent_fee: RentFee, renter_cancellation_fee: Option<CancellationFee>, rentee_cancellation_fee: Option<CancellationFee>);
  
  /// Interface Id: TIP400-02
  /// Description: The NFT Renter or NFT Rentee can revoke a rental contract.
  /// Constraint(s): Refer to section 'Rules'

  revoke_contract(nft_id: NFTId);


  /// Interface Id: TIP400-03
  /// Description: Depending on the type of acceptance and the acceptance list, the NFT Rentee will provide a rent offer, if the rental contract is in manual acceptance, or directly rent the NFT if the rental contract is in automatic acceptance.
  /// Constraint(s): None
  
  rent(nft_id: NFTId);

  /// Interface Id: TIP400-04
  /// Description: The NFT Renter can accept one of the rent offers if contract is set as manual acceptance.
  /// Constraint(s): Refer to section 'Rules'

  accept_rent_offer(nft_id: NFTId, rentee: AccountId)

  /// Interface Id: TIP400-05
  /// Description: The NFT Rentee can retract an offer made for a Rental NFT.
  /// Constraint(s): Refer to section 'Rules'

  retract_rent_offer(nft_id: NFTId)

  /// Interface Id: TIP400-06
  /// Description: The NFT Renter can change the subscription terms (duration and amount)
  /// Constraint(s): Refer to section 'Rules'

  change_subscription_terms(nft_id: NFTId, duration: Duration.Subscription, amount: Balance)

  /// Interface Id: TIP400-07
  /// Description: The NFT Rentee can accept the new subscription terms.
  /// Constraint(s): None

  accept_subscription_terms(nft_id: NFTId)
}

```

### Rules and constraints
#### Create_contract
- The NFT can not be listed in a marketplace, delegated or on auction to be a rental contract.
- A rental contract can only be created by the NFT owner.
- Rented NFTs cannot be delegated, transfered, burnned, listed or aucitoned.
- Royalties can not be set for rented NFTs.

#### Revoke_contract
- Cancellation fee (fixed/flexible amount or NFT) is charged on revocation if mentioned in the contract.
- Contract can only be revoked by NFT Renter or NFT Rentee
#### Additionnal Rules of revocation :
- If Contract Duration Type is Fixed and Contract Revocation Type is NoRevocation: Only rentee can revoke 
- If Contract Duration Type is Fixed and Contract Revocation Type is AnyTime: Both NFT Renter or NFT Rentee can revoke
- Contract Duration Type Fixed with Contract Revocation Type OnSubscriptionChange is not possible.
- If Contract Duration Type is Infinite and Contract Revocation Type is NoRevocation: Only rentee can revoke 
- If Contract Duration Type is Infinite and Contract Revocation Type is AnyTime: Both NFT Renter or NFT Rentee can revoke
- Contract Duration Type Infinite with Contract Revocation Type OnSubscriptionChange is not possible.
- If Contract Duration Type is Subscription and Contract Revocation Type is NoRevocation: Only rentee can revoke 
- If Contract Duration Type is Subscription and Contract Revocation Type is AnyTime: Both NFT Renter or NFT Rentee can revoke
- If Contract Duration Type is Subscription and Contract Revocation Type is OnSubscriptionChange: Both NFT Renter or NFT Rentee can revoke
#### Additionnal Rules about Cancellation Fees : 
- Flexible Tokens can only be applied for Fixed Contract Duration:  Fees will be calculated on a Pro-Rata basis.

#### Rent
- NFT renter can't rent the NFT

#### Retract_rent_offer
- Rentee can retract the offer he made until it has been accepted.

#### Change_subscription_terms
- Subscription terms can only be changed for a contract with a subscription duration and a revocation type set as OnSubscriptionChange only.
- Both the subscription duration and amount must be updated : They should be written again with the same initial duration/amount value if one of them do not changed. 
- New terms start at the next renewal subscription periode.
- Duration can't be changed to infinite or fixed. 
- rentFee can only be a tokens type, as an NFT can be used for subscription fee.

### Private interfaces

Rental NFT should support the following onchain private interfaces: These private interfaces are considered as native/root from the chain, and are automaticaly triggered by the chain itself during the lifecycle. User can't trigger them. 

```rust
interface {

  /// Interface Id: TIP400-08
  /// Description: ROOT - Triggered by the chain only,  when a contract ended.
  /// Constraint(s): None
  
  end_contract(nft_id: NFTId);
  
  /// Interface Id: TIP400-09
  /// Description: ROOT - Triggered by the chain only,  when a subscription contract periode is renewed.
  /// Constraint(s): None

  renew_contract(nft_id: NFTId);


  /// Interface Id: TIP400-10
  /// Description: ROOT - Triggered by the chain only,  when a contract expired.
  /// Constraint(s): None
  
  remove_expired_contract(nft_id: NFTId);
}

```

## Metadata
No  meta data (proper to the rent contract itself) are needed during the Rent Contract Lifecycle. 

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