# Ternoa Improvement Proposal - Soulbound NFT

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 15 Sep 2022       |
| TIP Number   | TIP104       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. An NFT can be defined as soulbound on creation. It means that once it has be received, it cannot do any more move operations (transfer, sell, rent, auction, ...) except burning.

## Abstract

In the objective of giving control over NFTs, creators can specify if an NFT is soulbound or not. It can have multiple usecases where it's necessary that the NFT kept on the wallet no matter what.

## Motivation

The main objective of soulbound NFT is to give a warranty that an NFT, after being issued won't move again.
An NFT can be issued by transfering it, putting on auction or selling it. After that the receiver account won't be able to change the ownership of the NFT except by burning it. This can be used in many usecases, eg. Issuing a license, a degree or even a game item.

## Specification

### Existing interface changed for  Soulbound NFT
Soulbound NFT support the following onchain interface:
```rust
interface {
  /// Interface Id: TIP104-01
  /// Description: User can create an NFT and specify the soulbound flag.
  /// Constraint(s): 
  ///		- None
  create_nft(owner: AccountId, offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);
}
```

## Constraints
 - None

## Additional Info

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating a soulbound NFT:

 1. User creates an NFT while specifying true for the soulbound flag.
 2. The NFT is now soulbound and can be transfered or sold only once.

The following is the workflow proposed for issuing a souldbound NFT by transferring it:

 1. User has already created a soulbound NFT and knows its id.
 2. User transfer the NFT to a recipient.
 3. The Soulbound NFT is now bound to the receiver address.

The following is the workflow proposed for issuing a souldbound NFT by selling it:

 1. User has already created a soulbound NFT and knows its id.
 2. User list the NFT for an amount of token.
 3. A user buys the NFT.
 4. The buyer is now bounded to the NFT.

The following is the workflow proposed for issuing a souldbound NFT by auctioning it:

 1. User has already created a soulbound NFT and knows its id.
 2. User put the NFT on auction.
 3. A user wins the auction.
 4. The winning bidder is now bounded to the NFT.

The following is the workflow proposed for burning a soulbound token:

 1. User has already received a soulbound NFT and knows its id.
 2. User calls the "burn_nft" function.
 3. The NFT does not exist anymore.

## Test cases

* User can create a soulbound NFT.
* User can issue a soulbound NFT by transferring it.
* User can issue a soulbound NFT by selling it.
* User can issue a soulbound NFT by auctioning it.
* User can burn a received soulbound NFT.
* User can't transfer a received soulbound NFT.
* User can't transfer a bought soulbound NFT.
* User can't transfer a soulbound NFT won at auction.
 
## References
TBD

## Copyright
TBD
