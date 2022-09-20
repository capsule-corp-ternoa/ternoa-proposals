# Ternoa Improvement Proposal - NFT Dealing

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 19 Sep 2022       |
| TIP Number   | TIP-700       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. Creating an NFT Deal allows users to do some P2P exchange without the need of using a marketplace.

## Abstract

An NFT deal is a P2P exchange or NFTs and / or Tokens. A deal is proposed from a dealer to a receiver.
The terms are set by the dealer and accepted by the receiver. There are two side on a deal. Each side can specify one or multiple NFTs and optionnaly some tokens. At least one of the deal side must contain an NFT.
This will allow P2P trading but it also represent a system where offer can be made on-chain.
If I propose some tokens for an NFT, it can be assimilated to an on-chain offer which is a powerful feature.

## Motivation

The main objective of NFT dealing is to give more control over the exchange of NFTs / Tokens. It's a feature commonly used in video game and it can be extended to offers and to enhance marketplaces. It also gives assurance over trading NFTs.

## Specification

### NFT Dealing External Interfaces
NFT Dealing types:
```rust
pub struct DealData {
  offerer: AccountId, 
  offeree: AccountId,
  offererSide: DealSide,
  offereeSide: DealSide,
}

pub struct DealSide {
  NFTs: Option<NFTList>,
  Amount: Option<Balance>,
}
```
NFT Dealing support the following onchain interfaces:
```rust
interface { 
  /// Interface Id: TIP700-01
  /// Description: User can create a deal.
  /// Constraint(s): 
  ///		- At least one DealSide must contains at least one NFT
  ///		- A DealSide must not be empty
  ///		- To create a deal, offerer and offeree must possess their respective DealSide
  ///		- Offeree must not be the caller
  create_deal(to: AccountId, offererSide: DealSide, offereeSide: DealSide);

  /// Interface Id: TIP700-02
  /// Description: User can cancel a created deal if it was not accepted yet.
  /// Constraint(s): 
  ///		- Deal must not have been accepted
  ///		- Deal must be created by user
  cancel_deal(deal_id: DealId);
  
  /// Interface Id: TIP700-03
  /// Description: User can accept a received deal.
  /// Constraint(s): 
  ///		- Receiver must still possess his respective DealSide
  accept_deal(deal_id: DealId);

  /// Interface Id: TIP700-04
  /// Description: User can decline a received deal.
  /// Constraint(s): 
  ///		- None
  decline_deal(deal_id: DealId); 
}
```

## Constraints
 - At least one DealSide must contains at least one NFT
 - A DealSide must not be empty
 - To create a deal, offerer and offeree must possess their respective DealSide
 - Receiver must still possess his respective DealSide to accept it

## Additional Info

## Metadata

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating a deal:
 1. User has some NFTs and / or Caps, user knows the offer recipient and the recipient NFTs and / or Caps.
 2. User creates a trade specifying the recipient, his side with one or multiples NFTs + Tokens, the recipient side with one or multiples NFTs + Tokens.
 3. Trade is created waiting to be accepted, declined, canceled or expired. User can retrieve the deal_id

The following is the workflow proposed for cancelling a deal:
 1. A deal has already been created and user knows the deal_id.
 2. User calls the "cancel_deal" function specifying the deal_id.
 3. The deal is know canceled.

The following is the workflow proposed for accepting a deal:
 1. A deal has already been created and the recipient knows the deal_id.
 2. The recipient calls the "accept_deal" function specifying the deal_id.
 3. The deal is know accepted, the NFTs are traded (with or without tokens).

The following is the workflow proposed for declining a deal:
 1. A deal has already been created and the recipient knows the deal_id.
 2. The recipient calls the "decline_deal" function specifying the deal_id.
 3. The deal is know declined.

## Test cases

* User can create a deal
* User can cancel a deal
* User can accept a received deal
* User can decline a received deal
* User cannot create a deal without at least 1 NFT in at least 1 side
* User cannot create a deal with unowned resources
* User cannot accept a deal with unowned resources
* A deal must expire if not accepted
 
## References
TBD

## Copyright
TBD
