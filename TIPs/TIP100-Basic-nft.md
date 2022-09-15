# Ternoa Improvement Proposal - __Basic NFTs__

| Author(s)      | Prabhu Eshwarla |
| ----------- | ----------- |
| Created   | 14 Sep 2022       |
| TIP Number   | TIP100       |
| Version   | v0.1       |
| Requires   |       |
| Status | Draft       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership of a digital or physical asset, on the blockchain. NFTs can represent digital assets such as images, videos, audio files, documents or simply some data. NFTs can also act as proxies for  physical assets such as real estate, consumer or industrial goods, or precious metals.

The media associated with NFTs, and other associated metadata, are stored offchain in a decentralised storage network such as IPFS, but the link to this metadata is stored onchain as part of the NFT. 

## Abstract

NFTs have metadata associated with them, some of them stored onchain, and others offchain. Examples of onchain metadata include name, description, preview image url, and the link to the offchain metadata. Examples of offchain metadata include the link to the media file associated with the NFT and additional properties that can be custom-defined by each dApp.  

NFTs can be minted into a collection, transferred from one account to another, listed on a marketplace, put up for sale on an auction, or even rented out to another account for a fee. NFTs can optionally have royanties associated with them, where the original creator of the NFT is paid a percentage of the sale value whenever the NFT changes ownership on a marketplace. There can also be special kind of NFTs that are non-transferable. Examples include personal credentials such as degree certificatees or an airline ticket.  

The range of operations and utilities associated with NFTs is a growing area of active research and web3 development. 

## Motivation

Digital assets on the internet have no inherent value as they can be freely replicated and transferred to multiple recipients. Blockchain technology enables digital assets to have scarcity and ownership, while retaining immutable records of transactions. NFTs have emerged as a class of digital assets on the blockchain to take advantage of these properties. NFTs are native digital assets on the blockchain that represent ownership of another digital asset(stored anywhere on the web) or a physical irl asset. They enable a huge new category of decentralised applications that was previously not possible or very difficult to accomplish.

## Specification

### Lifecycle states

__Basic NFTs__ can have one of the following lifecycle associated with them:
* Minted (when the NFT is first created on the blockchain)
* Burned (when the NFT is deleted from the storage of the blockchain)

Additionally if extensions are included, NFTs can have the following additional states:
* Delegated
* Non-transferable (soulbound)

### External interfaces

__Basic NFT__ should support the following onchain interfaces:
```
interface {

  /// Interface Id: TIP100-01
  /// Description: User can create a __Basic NFT__
  /// Constraint(s): Refer to section 'Rules'
  
  create_nft(CID offchain_uri, Permill royalty, CollectionId collection_id?, bool is_soulbound );
  
/// Interface Id: TIP100-02
  /// Description: This interface transfers the NFT from one account to another account.
  /// Constraint(s): Refer to section 'Rules'

  transfer_nft(NFTId nft_id, AccountId recipient_account)

  /// Interface Id: TIP100-03
  /// Description: This interface removes an NFT from storage. This operation is irreversible.
  /// Constraint(s): Refer to section 'Rules'

  burn_nft(NFTId nft_id)

  /// Interface Id: TIP100-04
  /// Description: This interface sets the mint fee for the NFT. This can only be changed through governance.
  /// Constraint(s): Refer to section 'Rules'

  set_nft_mint_fee(Currency fee)

}

```

### Rules and constraints

#### create_nft
- The metadata associated with the NFT should be stored in json format offchain, and the link to this file should be stored onchain. Offchain metadata should use ERC-1155 conventions.

#### transfer_nft
- Only the owner of the NFT can transfer it to another account

#### burn_nft
- Only the owner of the NFT can burn the NFT

#### set_nft_mint_fee
- This operation can only be performed through governance of the blockchain.

## Additional Info

__Basic NFT__ can have several extensions that can be optionally implemented by any chain or dApp adopting TIP-500. Examples of extensions include royalties, delegation, collection and soulbound tokens. Each of these will be described in a separate Ternoa Interface Proposal (TIP). 

However, the interface for ```create_nft``` in this TIP includes parameters for royalty, collection id and soulbound type. This is because, by default, it is expected that Ternoa NFTs will incorporate these extensions.

## Metadata

The metadata structure for a __Basic NFT__ has been adopted from ERC-1155, and can have the following (suggested) format:
```
{
2   "title":"",
3   "description":"",
4   "image":"",
5   "properties":{
6      "media":{
7         "hash":"media hash",
8         "type":"Type of media (file format)",
9         "size":"size of the encrypted media"
10      }
11   }
```

## End-to-end workflow (Ternoa-specific)

The following is the workflow for creating a __Basic NFT__:
1. User selects a media or custom data to be associated with the NFT, and stores it in a decentralised storage like IPFS. The Contend Id (CID) of the media file is retrieved.
2. The offchain metadata file is constructed using CID and other metadata properties. This offchain file is stored in IPFS (or similar storage). The CID of the metadata file is retrieved.
2. The NFT is created using the CID of the offchain metadata file, along with other onchain metadata.

The following is the workflow for viewing a __Basic NFT__:
1. User requests the wallet or dApp to view the media/data associated with the NFT owned by them.
2. The wallet/dApp retrieves the CID of the offchain metadata file, and sends a request to the IPFS node to retrive it.
3. The wallet/dApp retrieves the CID of the NFT media/data from the offchain metadata file, and requests the IPFS node for the media.
4. The wallet/dApp receives the media and displays it to user. 

## Test cases

* User can mint/create a __Basic NFT__ 
* User can burn a __Basic NFT__ 
* Mint fee for NFTs can be altered through an onchain governance proposal
 
## References
TBD

## Copyright
TBD