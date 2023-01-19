# Transmission Protocols
## Structures
```rust
pub  struct  NFTState  {  // added is_transmission
	// ...
	pub is_transmission: bool,
}

pub struct TransmissionData {
	pub recipient: AccountId,
	pub protocol: TransmissionProtocol,
	pub cancellation: CancellationPeriod,
}
```

## Types
```rust
type  InitialAtBlockFee: Get<BalanceOf<Self>>;
type  InitialAtBlockWithResetFee: Get<BalanceOf<Self>>;
type  InitialOnConsentFee: Get<BalanceOf<Self>>;
type  InitialOnConsentAtBlockFee: Get<BalanceOf<Self>>;
```

## Enums
```rust
/// Transmission protocol kind enum
enum TransmissionProtocolKind {
	AtBlock, 
	AtBlockWithReset, 
	OnConsent, 
	OnConsentAtBlock,
}

/// Transmission protocol enum
enum TransmissionProtocol {
	AtBlock<BlockNumber>,
	AtBlockWithReset<BlockNumber>,
	OnConsent<(BoundedVec<AccoundId, AccountSizeLimit>, threshold: u8)>
	OnConsentAtBlock<(BoundedVec<AccoundId, AccountSizeLimit>, threshold: u8, BlockNumber)>
}

/// Cancellation period enum
enum CancellationPeriod {
	None,
	UntilBlock<BlockNumber>,
	Anytime,
}
```

## Limits
```rust
type MaxBlockDuration:  Get<u32>(525_948_766);
type MaxConsentListSize:  Get<u32>(10);
type SimultaneousTransmissionLimit: Get<u32>(10_000_000);
type ActionsInBlockLimit: Get<u32>(1_000);
```

## Storage Units
```rust
pub  type  AtBlockFee:  StorageValue<Balance>
pub  type  AtBlockWithResetFee:  StorageValue<Balance>
pub  type  OnConsentFee:  StorageValue<Balance>
pub  type  OnConsentAtBlockFee:  StorageValue<Balance>
```
```rust
pub  type  Transmissions:  StorageMap<NFTId, TransmissionData>
```
```rust
pub  type  AtBlockQueue:  StorageValue<BoundedVec<(NFTId, BlockNumber), SimultaneousTransmissionLimit>>
```
```rust
pub  type  OnConsentData:  StorageMap<NFTId, BoundedVec<(AccountId, MaxConsentListSize)>>
```
## Hooks
```
- AtBlockQueue hook to check if a capsule need to be transfered
```
## Extrinsic
```rust
interface {
	/// Interface Id: TIP540-01
	/// Description: User can add a transmission protocol to any type of NFT. 
	/// The protocols are initially: 
	/// - AtBlock (block_number)
	/// - AtBlockWithReset (block_number)
	/// - OnConsent (BoundedVec<AccoundId>, threshold: u8)
	/// All protocols can be cancellable or not (details specified in TransmissionProtocol struct)
	/// Constraint(s): Refer to section 'Constraint'
	set_transmission_protocol(nft_id: NFTId, recipient: AccountId, protocol: TransmissionProtocol, cancellation: CancellationPeriod);

	/// Interface Id: TIP540-02
	/// Description: User can remove the transmission protocol if he allowed it when adding it.
	/// Constraint(s): Refer to section 'Constraint'
	remove_transmission_protocol(nft_id: NFTId);

	/// Interface Id: TIP540-03
	/// Description: User can reset the timer for AtBlockWithReset protocol.
	/// Constraint(s): Refer to section 'Constraint'
	reset_timer(nft_id: NFTId, block_number: BlockNumber);
	
	/// Interface Id: TIP540-04
	/// Description: Users specified in the account list for OnConsent protocol can add their consent to send the NFT.
	/// If the threshold is reached, the NFT will be sent to recipient.
	/// Constraint(s): Refer to section 'Constraint'
	add_consent(nft_id: NFTId);

	/// Interface Id: TIP540-05
	/// Description: Specify the fee for a transmission protocol.
	/// Constraint(s): Refer to section 'Constraint'
	set_protocol_fee(protocol: TransmissionProtocolKind, fee: u64);
}
```
## Events
```rust
pub enum Event {
  ProtocolSet{
    nft_id,
    recipient,
    protocol,
    cancellation
  },
  ProtocolRemoved{
    nft_id,
  },
  TimerReset: {
	nft_id,
	new_block_number,
  }
  ConsentAdded{
    nft_id,
    from,
  },
  ProtocolFeeSet{
    protocol,
    fee,
  },
  Transmitted{
    nft_id,
  }
}
```
## Errors
```rust
pub enum Error {
  // Basic errors
  // ...
  // Tranmission protocols Errors
  NotInTransmission,
  ProtocolNotCancellable,
  ResetNotAllowedForThisProtocol,
  ConsentNotAllowedForThisProtocol,  
}
```
## Side effects
```
NFT Pallet
- Checks on is_transmission

Marketplace Pallet
- Checks on is_transmission

Auction Pallet
- Checks on is_transmission

Rent Pallet
- Checks on is_transmission
```