



# Ternoa Improvement Proposal - Royalties

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 14 Sep 2022       |
| TIP Number   | TIP102       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. Royalties gives creators a way to receive a certain percentage of the secondary sales of their created NFTs. Royalty rules are set by the creator and enforced by the blockchain.

## Abstract

In the objective of giving control over secondary sales to creators. Royalties are a way for them to profit from secondary sales for NFTs. The creator can set the royalty when creating an NFT. He can also update the royalty percentage IF the creator is the owner of the NFT.

## Motivation

The main objective of marketplaces is to have a support / media to exchange NFTs other than plain P2P transfer. Having marketplaces allows user to define price for their NFTs. Those NFTs can be art, gaming items, tickets or any asset that can be represneted by a digital entity.
Royalties are computed and transfered to the creator on the following actions : 
 - NFT is sold in direct sale
 - NFT is sold in auction

## Specification

### Royalties External Interfaces
Royalties support the following onchain interfaces:
```rust
interface {
  /// Interface Id: TIP102-01
  /// Description: User can change the royalty of a created NFT if he's the owner
  /// Constraint(s): 
  ///       - The user must be the creator of the NFT
  ///       - The user must be the owner of the NFT
  ///		- The NFT must be available (not listed, delegated, rented, ...)
  set_royalty(nft_id: NFTId, royalty: permill);
}
```
### Existing Interfaces changed for collections
```rust
interface {
  /// Interface Id: TIP102-02
  /// Description: User can create an NFT spcifying the royalty.
  /// Constraint(s): 
  ///		- none
  create_nft(owner: AccountId, offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);
}
```
## Constraints
 - User must be owner and creator of an NFT to set its royalty.

## Additional Info

 - In case of a sale (direct or auction) the royalty percentage is computed after taking the commission fee for the marketplace (if it exists).
 - Royalty is a percentage between 0% and 100%.
 - Ex: Royalty is 20%, NFT price is 100 caps, marketplace commission fee is 10%. The royalty is computed this way : (price - commission_fee) * royalty_percentage <=> (100 - (100 * 0.1)) * 0.2 <=> 18 CAPS.


## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating an NFT with a royalty percentage:

 1. User create the an NFT specifying the royalty percentage as a permill (20% = 200000).
 2. User retrieves the NFT Id.

The following is the workflow proposed for setting the royalty amount of an owned and created NFT:

 1. User has already created an NFT and knows its id.
 2. User calls the "set_royalty" function with the NFT id and the new royalty percentage as a permill (20% = 200000).
 3. The NFT has now the new royalty amount.

The following is the workflow proposed for getting the royalty amount after a direct sale:

 1. A user has bought an NFT with royalty from direct sale.
 2. The creator of the NFT will receive an amount corresponding to the percentage of royalty and the price.

The following is the workflow proposed for getting the royalty amount after an auction:

 1. A user has won an auction for an NFT with royalty.
 2. The creator of the NFT will receive an amount corresponding to the percentage of royalty and the winning bid amount.

## Test cases

* User can create an NFT with a royalty percentage.
* User can set the royalty of a created and owned NFT.
* User receive royalty amount in case of direct sale.
* User receive royalty amount in case of auction sale.
 
## References
TBD

## Copyright
TBD
