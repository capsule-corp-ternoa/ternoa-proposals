# Capsule
## Structures
```rust
pub  struct  NFTState  {  // added is_syncing_capsule
	pub is_capsule:  bool,
	pub is_listed:  bool,
	pub is_secret:  bool,
	pub is_delegated:  bool,
	pub is_soulbound:  bool,
	pub is_syncing:  bool,
	pub is_syncing_capsule: bool,
}
```

## Types
```rust
type  InitialCapsuleMintFee: Get<BalanceOf<Self>>;
type  ShardsNumber: Get<u32>;
```

## Enums
```rust
// No enums
```

## Limits
```rust
type  NFTOffchainDataLimit:  Get<u32>(150)
```

## Storage Units
```rust
pub  type  CapsuleMintFee:  StorageValue<Balance>
```
```rust
pub  type  ShardsNumber:  StorageValue<u32> // Same as NFT (eg. 5)
```
```rust
pub  type  CapsulesOffchainData:  StorageMap<NFTId,  BoundedVec<u8,  NFTOffchainDataLimit>>
```
```rust
pub  type  CapsuleShardsCount:  StorageMap<NFTId,  BoundedVec<EnclaveId,  ShardsNumber>>
```

## Hooks
```rust
// No hooks
```
## Extrinsic
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

	/// Interface Id: TIP530-06v
	/// Description: This interface is called by each of the TEE enclaves to confirm receipt of secret share for a given Capsule NFT. When all enclaves from a cluster confirm receipt of threshold shares, the Capsule NFT status goes to 'Minted', after which it can be transferred through a transmission protocol. This is a private interface available only for the enclaves to use
	/// Constraint(s): Refer to section 'Constraint'
	add_capsule_shard(nft_id: NFTId);
	
	/// Interface Id: TIP530-07
	/// Description: This interface is called by the owner of the capsule to signify that new keys were requested by the capsule owner. This will change the is_syncing flag to true and will allow enclave to call add_capsule_shard again
	/// Constraint(s): Refer to section 'Constraint'
	notify_enclave_key_update(nft_id: NFTId);
}
```
## Events
```rust
pub enum Event {
  NFTConvertedToCapsule{
    nft_id
    offchain_data
  },
  CapsuleOffchainDataSet {
	nft_id,
	offchain_data,
  },
  CapsuleShardAdded{
    nft_id,
    enclave
  },
  CapsuleSynced{
    nft_id,
  },
  CapsuleReverted{
    nft_id,
  },
  CapsuleMintFeeSet{
    fee,
  },
  CapsuleKeyUpdateNotified{
    nft_id,
  },
}
```
## Errors
```rust
pub enum Error {
  // Basic error
  // ...
  // Capsule Errors
  EnclaveAlreadyAddedCapsuleShard,
  CapsuleAlreadySynced,
  NFTIsNotCapsule,
}
```
## Side effects
```
NFT Pallet
- Checks on is_capsule, is_syncing_capsule

Marketplace Pallet
- Checks on is_capsule, is_syncing_capsule

Auction Pallet
- Checks on is_capsule, is_syncing_capsule

Rent Pallet
- Checks on is_capsule, is_syncing_capsule
```