
---
tip: TIP-530
title: Capsule
version: 1.0
status: In progress
category: NFT, Token
authors: Ghali El Ouarzazi, Prabhu Eshwarla, Benjamin Arthuys <benjamin@capsule-corp.io>
created: 2022-10-06

---

# Capsule

## Simple Summary

Ternoa has created the "Capsule" token which is a new kind of token that allow users to store unlimited confidential data transferable on time.

## Abstract

Think of capsule as enhanced form of NFT which can take any number of files in encrypted media. Issuer should be able to convert his NFT into a Capsule and to be able to add limitless datas.

## Motivation

Existing storage solutions are limited and does not provide automated transfers services.

- Centralized storage solutions mean we have to be confident on the company that manage storage in term of security, maintenance and ethic.
- Data Privacy is something foundamental for humanity. Only decentralized solutions can provide real data privacy allowing users to get ownership of their own datas.
- Data transmission is something we all think about but no secured and automated services exist.
  For all those points, Ternoa created Capsules to bring a powerful solution to solve data issues.

## Specification

### External Interfaces

onchain interfaces:

```rust
interface {
	/// Interface Id: TIP530-01
	/// Description: User can convert an existing basic NFT into a Capsule NFT
	/// Constraint(s): Refer to section 'Constraint'
	convert_to_capsule(nft_id: NFTId, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>);

	/// Interface Id: TIP530-02
	/// Description: User can directly create an on-chain Secret NFT
	/// Constraint(s): Refer to section 'Constraint'
	create_capsule(offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);

	/// Interface Id: TIP530-03
	/// Description: User can remove the capsule part of his NFT
	/// Constraint(s): Refer to section 'Constraint'
	revert_capsule(nft_id: NFT_ID);

	/// Interface Id: TIP530-04
	/// Description: User can set the capsule's offchain data. Note that capsules are mutable unlike secret NFTs
	/// Constraint(s): Refer to section 'Constraint'
	set_capsule_offchaindata(nft_id: NFTId, capsule_offchain_data: BoundedVec<u8, NFTOffchainDataLimit>);

	/// Interface Id: TIP530-05
	/// Description: Capsule mint fee can be changed through governance
	/// Constraint(s): Refer to section 'Constraint'
	set_capsule_mint_fee(fee: u128 (BalanceOf));

	/// Interface Id: TIP530-06
	/// Description: This interface is called by each of the TEE enclaves to confirm receipt of secret share for a given Capsule NFT. When all enclaves from a cluster confirm receipt of threshold shares, the Capsule NFT status goes to 'Minted', after which it can be transferred through a transmission protocol. This is a private interface available only for the enclaves to use
	/// Constraint(s): Refer to section 'Constraint'
	add_capsule_shard(nft_id: NFTId);
	
	/// Interface Id: TIP530-07
	/// Description: This interface is called by the owner of the capsule to signify that new keys were requested by the capsule owner. This will change the is_syncing flag to true and will allow enclave to call add_capsule_shard again
	/// Constraint(s): Refer to section 'Constraint'
	notify_enclave_key_update(nft_id: NFTId);
}

```

## Constraints

### convert_to_capsule
- Called by the owner of the NFT
- The NFT must not be in one of this state : 
	- Listed
	- Delegated
	- Rented
	- Syncing
	- Used as rental cancellation fee
	- Capsule
	- In transmission
- The NFT when converted to capsule will be in "Syncing" state. Only when all the secret shares associated with the capsule have been stored in the enclaves, should the Capsule change to not syncing.

### create_capsule
- The NFT when converted to capsule will be in "Syncing" state. Only when all the secret shares associated with the capsule have been stored in the enclaves, should the Capsule change to not syncing.

### revert_capsule
- Called by the owner of the NFT
- The capsule must not be in one of this state
	- Listed
	- Delegated
	- Rented
	- Syncing
 	- Used as rental cancellation fee
	- In transmission
- The NFT must be a capsule

### set_capsule_offchaindata
- Called by the owner of the NFT
- The capsule must not be in one of this state
	- Listed
	- Delegated
	- Rented
	- Syncing
 	- Used as rental cancellation fee
	- In transmission
- When called, the offchain data must use the same keys to convert the medias
- The NFT must be a capsule

### set_capsule_mint_fee
- Root only

### add_capsule_shard
- Only enclaves can use this interface. Not to be used by dApps or users.
- When all the secret shares associated with a secret NFT have been confirmed to be received, then the NFT state should be changed from 'Pending Mint' to 'Minted'
- NFT must be in syncing state
- The NFT must be a capsule

### notify_enclave_key_update
- Called by the owner of the NFT
- The capsule must not be in one of this state
	- Listed
	- Delegated
	- Rented
	- Syncing
 	- Used as rental cancellation fee
	- In transmission
- The NFT must be a capsule

## Additional Info
- Capsules have mutable content (offchain data), whereas secret NFT contents are immutable.
- Capsules can have more than one media file but secret NFTs can only have one file. (Files can be of any type including text, pdfs, images, video, audio, etc).
- Cryptographic keys used to encrypt contents can be changed for capsules by the current owner, but not for secret nfts.
- When a secret NFT is converted to a capsule, the key for secret nft cannot be changed, but the key for the capsule can be changed by capsule owner.
- There will be a one-time fee for minting capsules.
- A basic NFT can be burned even after it is converted into a secret NFT or capsule.
- If a secret NFT or a capsule is delegated / rented, the delegatee/rentee can decrypt the contents.
- Decision should be made from tech to chose if 
	- Capsule is a separate pallet
	- Capsule is integrated in NFT pallet
	- Create capsule is in NFT pallet and the rest is in a separate pallet

## Metadata

The Capsule NFT is an extension of the Basic NFT. The Basic NFT has its own metadata that is stored in json format in IPFS.

The format for the offchain metadata of Capsule NFT is suggested here:

```json
{
	"title":"(Optional) This the title of the Capsule NFT",
	"description":"(Optional) Description of the Capsule NFT",
	"properties":{
		"encrypted_media":[{
			"hash":"media hash",
			"type":"Type of media (file format)",
			"size":"size of the encrypted media"
		}],
		"public_key_of_nft": "public key associated with the Capsule NFT",
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

1. User requests the wallet or dApp to decrypt data associated with the Capsule owned by them.
2. The wallet/dApp retrieves the secret shares from the enclaves and reconstructs the decryption key.
3. The Wallet/dApp retrieves the capsule media from IPFs, decrypts it and displays the data/media to user. Only owners of the capsule can view the data associated with the capsule.

## Test cases
- User **should be able to** mint a Capsule directly
- Owner **should be able to** convert a Basic NFT to Capsule
- Owner **should be able to** decrypt content data associated with the Capsule
- Owner **should be able to** transfer a Capsule either by direct transfer or using tansmission protocols
- Owner **should be able to** list Capsule in the marketplace
- Owner **should be able to** trade Capsule in any marketplace
- Owner **should be able to** rent or delegate a capsule
- Owner **should be able to** use a capsule as rental fee

## References

## Copyright

Copyright and related rights waived via CC0.