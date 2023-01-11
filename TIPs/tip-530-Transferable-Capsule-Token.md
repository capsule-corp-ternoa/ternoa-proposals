---
tip: TIP-530
title: Transferable Capsule Token
version: 1.0
category: NFT, Token
authors: Benjamin Arthuys <benjamin@capsule-corp.io>
created: 2022-10-06
---

# Transferable Capsule Token (TCT)

## Simple Summary

Ternoa has created the "Transferable Capsule Token" which is a new kind of token that allow users to store unlimited confidential data transferable on time.

## Abstract

Think of capsule as enhanced form of NFT which can take any number of files in encrypted media. Issuer should be able to convert his NFT into TCT and to be able to add limitless datas. 

## Motivation

Existing storage solutions are limited and does not provide automated transfers services.

- Centralized storage solutions mean we have to be confident on the company that manage storage in term of security, maintenance and ethic.
- Data Privacy is something foundamental for humanity. Only decentralized solutions can provide real data privacy allowing users to get ownership of their own datas.
- Data transmission is something we all think about but no secured and automated services exist.

For all those points, Ternoa created TCT to bring a powerful solution to solve data issues.

## Specification

### External Interfaces

onchain interfaces:

```rust
interface {
  /// Interface Id: TIP520-01
  /// Description: User can convert an existing basic NFT into a Capsule NFT
  /// Constraint(s): Refer to section 'Rules'
  convert_to_capsule(nft_id: NFTId, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>);
  
  /// Interface Id: TIP520-02
  /// Description: User can directly create an on-chain Secret NFT
  /// Constraint(s): Refer to section 'Rules'
  create_capsule(offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);

  /// Interface Id: TIP520-03
  /// Description: User can set the capsule's offchain data. Note that capsules are mutable unlike secret NFTs
  /// Constraint(s): Refer to section 'Rules'
  set_capsule_offchaindata(nft_id: NFTId, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>)

  /// Interface Id: TIP520-04
  /// Description: Capsule mint fee can be changed through governance
  /// Constraint(s): Refer to section 'Rules'
  set_capsule_mint_fee(fee: u128 (BalanceOf))

  /// Interface Id: TIP520-05
  /// Description: This interface is called by each of the TEE enclaves to confirm receipt of secret share for a given Capsule NFT. When all enclaves from a cluster confirm receipt of threshold shares, the Capsule NFT status goes to 'Minted', after which it can be transferred through a transmission protocol. This is a private interface available only for the enclaves to use

  /// Constraint(s): Refer to section 'Rules'
  add_secret_share(NFTId nft_id)
}
```

## Constraints
Capsules will behave similar to secret NFTs for the following actions:
1. Transfers
2. Market place listing & trading
3. Delegation
4. Soulbound property
5. Collections
6. Renting
7. Ability to be used as rental fee.

Capsules differ from secret NFTs in the following areas:
1. Capsules have mutable content (offchain data), whereas secret NFT contents are immutable.
2. Capsules can have more than one media file but secret NFTs can only have one file. (Files can be of any type including text, pdfs, images, video, audio, etc).
3. Cryptographic keys used to encrypt contents can be changed for capsules by the current owner, but not for secret nfts. 
4. When a secret NFT is converted to a capsule, the key for secret nft cannot be changed, but the key for the capsule can be changed by capsule owner.
5. There will be a one-time fee for minting capsules.
6. A basic NFT can be burned even after it is converted into a secret NFT or capsule.
7. If a secret NFT or a capsule is delegated, the delegatee/rentee can decrypt the contents.
8. 
## Additional Info

## Metadata

The Capsule NFT is an extension of the Basic NFT. The Basic NFT has its own metadata that is stored in json format in IPFS. 
The format for the offchain metadata of Capsule NFT is suggested here:
```
{
   "title":"(Optional) This the title of the Capsule NFT",
   "description":"(Optional) Description of the Capsule NFT",
   "properties":{
      "encrypted_media":[{
        
         "hash":"media hash",
         "type":"Type of media (file format)",
         "size":"size of the encrypted media"
        
      }],
      "public_key_of_nft": "(Optional) public key associated with the Capsule NFT",
   }
}
```
This metadata of capsule NFT will be stored on IPFs, and it’s content Id (CID) will be stored onchain.

## End-to-end workflows (Ternoa-specific)

**Capsule Minting Workflow:**

1. User selects data for storing privately in a Capsule.
2. The Wallet or dApp encrypts the data with an encrypption key and uploads to IPFS
3. The Wallet/dApp mints Capsule NFT on-chain
4. The Wallet/dApp creates secret shares from encryption key, and stores the secret shares on Ternoa TEE enclaves
5. The TEE enclaves confirm receipt of shares for Capsule, by triggering extrinsics on blockchain.
6. The status of capsule NFT changes to 'Minted'. The capsule is now ready to be associated with a transmission protocol.

**Capsule displaying Workflow (Owner View)**

1. User requests the wallet or dApp to decrypt data associated with the TCT owned by them.
2. The wallet/dApp retrieves the secret shares from the enclaves and reconstructs the decryption key.
3. The Wallet/dApp retrieves the capsule media from IPFs, decrypts it and displays the data/media to user. Only owners of the capsule can view the data associated with the capsule.


## Test cases

- User **should be able to** mint a TCT directly
- Owner **should be able to** convert a Basic NFT to TCT
- Owner **should be able to** decrypt content data associated with the TCT
- Owner **should be able to** transfer a TCT either by direct transfer or using tansmission protocols
- Owner **should be able to** list TCT in the marketplace
- Owner **should be able to** trade TCT in any marketplace
- Owner **should be able to** rent or delegate a capsule
- Owner **should be able to** use a capsule as rental fee
 
## References

## Copyright

Copyright and related rights waived via CC0.
