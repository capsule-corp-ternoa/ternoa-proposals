# Ternoa Improvement Proposal - Marketplaces

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 13 Sep 2022       |
| TIP Number   | TIP200       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. Creators need a way to group NFTs in a higher order entity. This entity is a collection. If a collection is created, NFTs can be put in it at its creation or after.
NFTs represent proof of ownership on the blockchain. Marketplaces are entities used, primarily, to sell and buy NFTs. A marketplace's ownership and rules are also defined by the chain.

## Abstract

In the objective of seeling and buying NFTs, we created the marketplace entity. Each user can create his own marketplace and define rules that apply to who can put on sale, what is the listing (putting on sale) cost and what is the commission fee. This document descrive the motivation and specification of the marketplace entity.

## Motivation

The main objective of marketplaces is to have a support / media to exchange NFTs other than plain P2P transfer. Having marketplaces allows user to define price for their NFTs. Those NFTs can be art, gaming items, tickets or any asset that can be represneted by a digital entity.

## Specification

### Marketplace Types
```rust
types {
	/// Possible operations.
	pub enum ConfigOp {
		Noop, // Don't change.
		Set(T), // Set the given value.
		Remove, // Remove the value.
	}

	/// Multiple form of fees
	pub enum CompoundFee<Balance> {
		Flat(Balance) // A flat amount of token,
		Percentage(Permill) // A percentage of token,
	}
}
```
### Marketplace External Interfaces
Marketplaces support the following onchain interfaces:
```rust
interface {
  /// Interface Id: TIP200-01
  /// Description: User can create a marketplace specifying the kind.
  /// 	The kind is the type of marketplace, it can be public or private. 
  /// 	Public mean that anyone can list except the addresses put in the account_list (ban_list).
  /// 	Private mean that no one can list except the addresses put in the account_list (allow_list).
  /// Constraint(s): 
  ///       - The user must have enough funds to cover for the marketplace mint fee
  create_marketplace(kind: MarketplaceType);
  
  /// Interface Id: TIP200-02
  /// Description: User can transfer ownership of a marketplace.
  /// Constraint(s): 
  ///       - User needs to be the owner of the marketplace
  set_marketplace_owner(marketplace_id: MarketplaceId, recipient: AccountId);
  
  /// Interface Id: TIP200-03
  /// Description: User can change the kind (type) of a marketplace.
  /// 	The kind is the type of marketplace, it can be public or private. 
  /// 	Public mean that anyone can list except the addresses put in the account_list (ban_list).
  /// 	Private mean that no one can list except the addresses put in the account_list (allow_list).
  /// Constraint(s): 
  ///       - User needs to be the owner of the marketplace
  ///		- If public -> private, accout_list becomes an allow list instead of a ban list
 ///		- If private -> public, accout_list becomes a ban list instead of an allow list
  set_marketplace_kind(marketplace_id: MarketplaceId, recipient: AccountId);

  /// Interface Id: TIP200-04
  /// Description: User can change the configuration option of the marketplace. He can change the commission_fee, the listing_fee, the account_list and the offchain data.
  /// Constraint(s): 
  ///       - User needs to be the owner of the marketplace
  set_marketplace_configuration(marketplace_id: MarketplaceId, commission_fee: ConfigOp<CompoundFee<Balance>>, listing_fee: ConfigOp<CompoundFee<Balance>>, account_list: ConfigOp<BoundedVec<AccountId, AccountSizeLimit>>, offchain_data: ConfigOp<BoundedVec<u8, offchainDataLimit>>);

  /// Interface Id: TIP200-05
  /// Description: User can list an NFT for direct sale if authorized in the marketplace.
  /// Constraint(s): 
  ///		- NFT must be available
  ///       - User must be the owner of the NFT
  ///		- User must be authorized
  ///		- User must have enough funds to cover for listing fee in case it exists
  ///		- User must have enough funds to cover for the flat commission fee if it exists
  list_nft(nft_id: NFTId, marketplace_id: MarketplaceId, price: Balance);

  /// Interface Id: TIP200-06
  /// Description: User can unlist a listed NFT.
  /// Constraint(s): 
  ///		- NFT must to be on sale
  ///       - User must to be the owner of the NFT
  unlist_nft(nft_id: NFTId);
  
  /// Interface Id: TIP200-07
  /// Description: User can buy a listed NFT.
  /// Constraint(s): 
  ///		- NFT needs to be on sale
  ///       - User must not be the owner of the NFT
  ///		- User must have enough funds to pay the NFT price
  buy_nft(nft_id: NFTId);
}
```
## Constraints
 - User must have enough funds to cover for marketplace creation fee to create a marketplace.
 - User must be owner of marketplace to change marketplace configuration.
 - User must be owner of marketplace to change kind.
 - User must be owner of marketplace to change marketplace ownership.
 - User must be authorized to list an NFT.
 - User must be the owner of the NFT to list it for sale.
 - User must have enough funds to cover for the listing fee in case it exists.
 - User must have enough funds to cover for the flat commission fee if it exists.
 - User must be the owner of the NFT to unlist it
 - NFT must be on sale to be unlisted.
 - NFT must be on sale to be bought.
 - User must not be the owner of the NFT to buy it.
 - User must have enough caps to pay for the price to buy an NFT.

## Additional Info

## Metadata

Marketplaces, like NFTs and collections, have their own metadata that are stored in json in either IPFS, a private server, any other type of storage. Metadatas cannot be enforced but we suggest this format:
```json
{
	"name":"This the title of the collection",
	"logo":"Hash / Link to the logo",
}
```
This metadata is stored offchain and only the CID (in case of IPFS), the link or the string corresponding to offchain data, will be stored on chain.

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating a marketplace:

 1. User create the marketplace specifying the kind.
 2. User retrieves the marketplace id.

The following is the workflow proposed for transferring a marketplace ownership:

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_owner" function with the address of the recipient.
 3. The recipient is now the owner of the marketplace.

The following is the workflow proposed for changin the kind / type of a marketplace:

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_kind" specifying private of public.
 3. The marketplace is now private or public. The account_list is respectively an allow list or ban list.

The following is the workflow proposed for changing the commission fee to a percentage :

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_configuration" function with the parameters (Set(Percentage(200000)), NoOp, NoOp, NoOp).
 3. The commission fee is now changed to 20% while other values did not.

The following is the workflow proposed for changing the listing_fee to a flat amount of token:

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_configuration" function with the parameters (NoOp, Set(Flat(10_000_000_000_000_000_000)), NoOp, NoOp).
 3. The listing_fee is now changed to 10 CAPS while other values did not change.

The following is the workflow proposed for removing the listing_fee:

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_configuration" function with the parameters (NoOp, Remove, NoOp, NoOp).
 3. The listing_fee is now removed.

The following is the workflow proposed for setting the account_list:

 1. User has already created a marketplace and knows its id.
 2. User calls the "set_marketplace_configuration" function with the parameters (NoOp, NoOp, Set(BoundedVec::try_from(vec![5CDG..., 5Haz...])), NoOp).
 3. The account_list is now set.

The following is the workflow proposed for setting offchain data:

 1. User has already created a marketplace and knows its id.
 2. User prepares the metadata file in JSON format.
 3. User uploads it on IPFS retrieving the CID
 4. User calls the "set_marketplace_configuration" function with the parameters (NoOp, NoOp, NoOp, Set(CID)).
 5. The marketplace has offchain data set.

The following is the workflow proposed for listing an NFT:

 1. User has already created an NFT and knows its id.
 2. User knows the marketplace id on which he wants to list.
 3. User calls the "list_nft" function with the nft id, the marketplace id and the price.
 4. The NFT is now listed on the specified marketplace.

The following is the workflow proposed for unlisting an NFT:

 1. User has already created and listed an NFT and knows its id.
 2. User calls the "unlist_nft" function with the nft id.
 3. The NFT is now unlisted.

The following is the workflow proposed for buying an NFT:

 1. User knows the NFT he wants to buy.
 2. User has enough funds to buy the NFT.
 3. User calls the "buy_nft" function with the nft id.
 4. The NFT is now owned by the buyer.


## Test cases

* User can create a marketplace.
* User can transfer a marketplace ownership.
* User can change the marketplace kind.
* User can change the marketplace configuration.
* User can list an NFT if authorized.
* User can unlist a listed NFT.
* User can buy an NFT if he has enough funds.
 
## References
TBD

## Copyright
TBD
