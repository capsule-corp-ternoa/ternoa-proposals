---
tip: TIP-100
title: Basic NFT
version: 0.1.1
category: NFT
authors: Prabhu Eshwarla
created: 2022-09-14
---

# Ternoa Improvement Proposal - __Basic NFTs__

## Simple Summary

NFTs represent proof of ownership of a digital or physical asset, on the blockchain. NFTs can represent digital assets such as images, videos, audio files, documents or simply some data. NFTs can also act as proxies for physical assets such as real estate, consumer or industrial goods, or precious metals.

The media associated with NFTs, and other associated metadata, are stored offchain in a decentralised storage network such as IPFS, but the link to this metadata is stored onchain as part of the NFT. 

## Abstract

NFTs have metadata associated with them, some of them stored onchain, and others offchain. Examples of onchain metadata include name, description, preview image url, and the link to the offchain metadata. Examples of offchain metadata include the link to the media file associated with the NFT and additional properties that can be custom-defined by each dApp.  

NFTs can be minted into a collection, transferred from one account to another, listed on a marketplace, put up for sale or auctioned, or even rented out to another account for a fee. NFTs can optionally have royalties associated with them, where the original creator of the NFT is paid a percentage of the sale value whenever the NFT changes ownership on a marketplace. There can also be special kind of NFTs that are non-transferable. Examples include personal credentials such as university degree certificatees or a national identification document.  

The range of operations and utilities associated with NFTs is a growing area of active research and web3 development. 

## Motivation

Digital assets on the internet have no inherent value as they can be freely replicated and transferred to multiple recipients. Blockchain technology enables digital assets to have scarcity and ownership, while retaining immutable records of transactions. NFTs have emerged as a class of digital assets on the blockchain to take advantage of these properties. NFTs are native digital assets on the blockchain that represent ownership of a digital asset(could be stored anywhere on the web) or a physical IRL asset. They enable a huge new category of decentralised applications that was previously not possible or very difficult to accomplish.

## Specification

### Lifecycle states

__Basic NFTs__ can have one of the following lifecycle associated with them:
* Minted (when the NFT is first created on the blockchain)
* Burnt (when the NFT is deleted from the storage of the blockchain)

Additionally if extensions are included, NFTs can have the following additional states:
* Delegated
* Non-transferable (if it's a soulbound)

### External interfaces

__Basic NFT__ should support the following onchain interfaces:

```rust

interface {

  /// Interface Id: TIP100-01
  /// Description: User can create a __Basic NFT__
  /// Constraint(s): Refer to section 'Constraints'
  create_nft(offchainData: Bytes, royalty: Permill, collectionId: Option<u32>, isSoulbound: bool);
  
  /// Interface Id: TIP100-02
  /// Description: This interface transfers the NFT from one account to another account.
  /// Constraint(s): Refer to section 'Constraints'
  transfer_nft(nftId: u32(NFTId), recipientId: AccountId)

  /// Interface Id: TIP100-03
  /// Description: This interface removes an NFT from storage. This operation is irreversible.
  /// Constraint(s): Refer to section 'Constraints'
  burn_nft(nftId: u32 (NFTId))

  /// Interface Id: TIP100-04
  /// Description: This interface sets the mint fee for the NFT. This can only be changed through governance.
  /// Constraint(s): Refer to section 'Constraints'
  set_nft_mint_fee(fee: u128 (BalanceOf))

}

```

### Constraints

#### create_nft
- The metadata associated with the NFT should be stored in json format offchain, and the link to this file should be stored onchain. Offchain metadata should use ERC-1155 conventions.

#### transfer_nft
- Only the owner of the NFT can transfer it to another account

#### burn_nft
- Only the owner of the NFT can burn the NFT

#### set_nft_mint_fee
- This operation can only be performed through governance of the blockchain.

## Additional Info

__Basic NFT__ can have several extensions that can be optionally implemented by any chain or dApp adopting TIP-500. Examples of extensions include royalties, delegation, collection and soulbound tokens. Each of these will be described in a separate Ternoa Improvement Proposal (TIP). 

However, the interface for ```create_nft``` in this TIP includes parameters for royalty, collection id and soulbound type. This is because, by default, it is expected that Ternoa NFTs will incorporate these extensions.

## Metadata

The metadata structure for a __Basic NFT__ has been adopted from ERC-1155, and can have the following (suggested) format:

```json

{
  "title":"", //text
  "description":"", //text
  "image":"", //IPFS CID or complete getable URL
  "properties":{
    "media":{
      "hash":"media IPFS CID",
      "type":"Type of media (file format)",
      "size":"size of the media"
    }
  }
}

```

## End-to-end workflow (Ternoa-specific)

The following is the workflow for creating a __Basic NFT__:
1. User selects a media or custom data to be associated with the NFT, and stores it in a decentralised storage like IPFS. The Content Id (CID) of the media file is retrieved.
2. The offchain metadata file is constructed using CID and other metadata properties. This offchain file is stored in IPFS (or similar storage). The CID of the metadata file is retrieved.
3. The NFT is created using the CID of the offchain metadata file, along with other onchain metadata.

The following is the workflow for viewing a __Basic NFT__:
1. User requests the wallet or dApp to view the media/data associated with the NFT owned by them.
2. The wallet/dApp retrieves the CID of the offchain metadata file, and sends a request to the IPFS node to retrive it.
3. The wallet/dApp retrieves the CID of the NFT media/data from the offchain metadata file, and requests the IPFS node for the media.
4. The wallet/dApp receives the media and displays it to user. 

## Test cases

* User can mint/create a __Basic NFT__ 
* User can burn a __Basic NFT__ 
* User can transfer a __Basic NFT__
* Mint fee for NFTs can be altered through an onchain governance proposal
 
## References
TBD

## Copyright
TBD
