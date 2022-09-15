# Ternoa Improvement Proposal - Collections

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 12 Sep 2022       |
| TIP Number   | TIP101       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. Creators need a way to group NFTs in a higher order entity. This entity is a collection. If a collection is created, NFTs can be put in it at its creation or after.

## Abstract

In EVM / Smart Contract based, collection are represented by smart contract. It implies some advantages and drawback. "Buyers" need to mint their NFT themselves, each contract will have its own set of rules, the smart contract can follow or not some standards.
At Ternoa, you can create and be the owner of a collection. When minting NFTs, you can specify this collection and put it on sale. The NFTs will always be part of this collection and it's origin / grouping can be ensured over time. Collections will always follow the same set of rules.

## Motivation

The main objective of collections are to give a way to group NFTs. Grouping can have multiple meaning, eg. having the same NFT in a high quantity grouped in the same collection or having multiple NFTs sharing the same types of attributes, while still being different, gathered in one entity. This can ease navigation in dApps and more generally improve user experience while using Ternoa's chain.

## Specification

### Collection External Interfaces
Collections support the following onchain interfaces:
```rust
interface { 
  /// Interface Id: TIP101-01
  /// Description: User can create a new collection.
  /// Constraint(s): 
  ///		- None
  create_collection(offchain_data, BoundedVec<u8, CollectionOffchainDataLimit>, limit: Option<u32>);


  /// Interface Id: TIP101-02
  /// Description: User can add NFTs to his collection.
  /// Constraint(s):  
  ///		- The collection must not be closed.
  ///		- The collection must not have reached limit if it has one.
  ///		- NFT must not already be in a collection.
  ///		- NFT must be owned.
  ///		- Collection must be owned.
  add_nft_to_collection(nft_id: NFTId, collection_id: CollectionId);

  /// Interface Id: TIP101-03
  /// Description: User can add NFTs to his collection.
  /// Constraint(s):  
  ///		- Collection must be owned.
  ///		- The collection must be empty. (If NFTs are in the collection, they need to be burned)
  burn_collection(collection_id: CollectionId);

  /// Interface Id: TIP101-04
  /// Description: User can close a collection signalling that no more NFTs will be added in the collection.
  /// Constraint(s):  
  ///		- Collection must be owned.
  ///		- Collection must not be already closed.
  close_collection(collection_id: CollectionId);
  
  /// Interface Id: TIP101-05
  /// Description: User can add a maximum number of NFTs as a limit to his collection. Once the collection reaches that number of collection, it will be considered limited (complete)
  /// Constraint(s):  
  ///		- Collection must be owned.
  ///		- Collection must not be closed.
  ///		- Collection must not already have a limit (it can be set on collection creation).
  ///		- Collection must have an inferior amount of NFTs that the speified limit.
  limit_collection(collection_id: CollectionId, limit: u32);
}
```

### Existing Interfaces changed for collections
```rust
interface {
  /// Interface Id: TIP101-06
  /// Description: User can create an NFT with any existing collection he owns.
  /// Constraint(s): 
  ///		- The collection must not be closed.
  ///		- The collection must not have reached limit if it has one.
  ///		- Collection must be owned.
  create_nft(owner: AccountId, offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);
}
```

## Constraints

 - Collection must not be close or limited to add an NFT in it
 - Collection must be owned to add NFT in it
 - Collection must be owned to close it
 - Collection must be owned to add a limit
 - Collection must be empty to be burned

## Additional Info

## Metadata

Collection, like NFTs, have their own metadata that are store in json in either IPFS, a private server, any other type of storage. Metadatas cannot be enforced but we suggest this format:
```json
{
	"name":"This the title of the collection",
	"description":"Description of the collection",
	"profile_image":"Profile picture of the collection (Link / IPFS / Any)",
	"banner_image":"Banner picture of the collection (Link / IPFS / Any)",
}
```
This metadata is stored offchain and only the CID (in case of IPFS), the link or the string corresponding to offchain data, will be stored on chain.

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating a collection:

 1. User create his metadata in a json format
 2. User upload it to IPFS retrieving the CID
 3. User create the collection spcifying the CID and the optional desired limit
 4. User retrieves the collection id

The following is the workflow proposed for minting an NFT with a collection:

 1. User has already created a collection and knows the corresponding id
 2. User create his NFT metadata in a json format
 3. User upload the metadata to IPFS retrieving the CID
 4. User create his NFT by specifying the CID, royalties, soulbound flag but most importantly the collection id.

The following is the workflow proposed for adding an NFT to a collection:

 1. User has already created a collection and knows the corresponding id
 2. User has already created an NFT and knows the corresponding id
 3. User triggers the "add_nft_to_collection" function specifying the nft id and the collection id

The following is the workflow proposed for adding a limit to a collection:

 1. User has already create a collection WITHOUT specifying a limit and know the corresponding id
 2. User triggers the "limit_collection" function specifying the collection id and the maximum number of NFTs in the collection.
 3. Collection now has a maximum number (limit)

The following is the workflow proposed for closing a collection:

 1. User has already created a collection and knows the corresponding id
 2. User triggers the "close_collection" function specifying the collection id
 3. Collection is now close and can't accept any more NFTs

The following is the workflow proposed for burning a collection:
 1. User has already created a collection (EMPTY) and knows the corresponding id
 3. User triggers the "burn_collection" function specifying the collection id
 4. Collection does not exist anymore
 
The following is the workflow proposed for burning a collection with NFT in it:
 1. User has already created a collection and has NFTs in it and knows the corresponding id
 2. User triggers the "burn_nft" function for each NFTs in the collection. If he doesn't own all the NFTs, hs won't be able to burn them and burn the collection.
 3. User triggers the "burn_collection" function specifying the collection id
 4. Collection does not exist anymore

## Test cases

* User can create a collection
* User can create an NFT directly in a collection
* User can add an NFT to a collection
* User can limit his collection if not set at creation
* User can close his collection
* User can burn a collection if it's empty
 
## References
TBD

## Copyright
TBD
