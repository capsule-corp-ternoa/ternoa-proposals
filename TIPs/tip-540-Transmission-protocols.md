
---
tip: TIP-540
title: Transmission protocols
version: 1.0
status: In progress
category: NFT, Token, Capsule, Oracles
authors: Ghali El Ouarzazi
created: 2023-01-13

---

# Transmission protocols

## Simple Summary

Ternoa has created transmission protocols to be able to change the state and ownership of a digital asset (NFT) automatically after a specific condition has been met. 

## Abstract

For a capsule or a secret NFT, only the owner, the delegatee or the rentee of the asset can view the secret. It requires manual actions.
With transmission protocols, we enable a way to securely transmit data / information by specifying a set of conditions (Time, will, consensus, death, ...).

## Motivation

We respond to existing centralized storage solutions by using capsule or secret NFTs.
To give a solution for secure data transmission, we have transmission protocols. 
Existing storage solutions are limited and does not provide automated transfers services.

## Specification

### External Interfaces

onchain interfaces:

```rust
interface {
	/// Cancellation period enum
	enum CancellationPeriod {
		None,
		UntilBlock<BlockNumber>,
		Anytime,
	}

	/// Transmission protocol enum
	enum TransmissionProtocol {
		AtBlock<BlockNumber>
		AtBlockWithReset<BlockNumber>
		OnConsent<BoundedVec<BoundedVec<AccoundId, AccountSizeLimit>, threshold: u8>>
	}

	/// Interface Id: TIP540-01
	/// Description: User can add a transmission protocol to any type of NFT. 
	/// The protocols are initially: 
	/// - AtBlock (block_number)
	/// - AtBlockWithReset (block_number)
	/// - OnConsent (BoundedVec<AccoundId>, threshold: u8)
	/// All protocols can be cancellable or not (details specified in TransmissionProtocol struct)
	/// Constraint(s): Refer to section 'Constraint'
	set_transmission_protocol(nft_id: NFTId, recipient: AccountId, protocol: TransmissionProtocol, cancellation_period: CancellationPeriod);

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
	set_protocol_fee(protocol_type: TransmissionProtocol, fee: u64);
}

```

## Constraints

### set_transmission_protocol
- Called by the owner of the NFT
- The NFT must not be in one of this state : 
	- Listed
	- Delegated
	- Rented
	- Syncing
	- Used as rental cancellation fee
	- In Transmission
- User must have enough funds to cover the transmission protocol fee

### remove_transmission_protocol
- Called by the owner of the NFT
- The NFT must be in transmission
- When setting the transmission protocol, owner must have had specified it is cancellable

### reset_timer
- Called by the owner of the NFT
- The NFT must be in transmission
- Protocol must be AtBlockWithReset

### add_consent
- Caller must be in the specified list
- The NFT must be in transmission
- Protocol must be OnConsent

### set_protocol_fee
- Caller must be root

## Additional Info
- This will be a separate pallet called transmission_protocols

## Metadata

No offchain metadata

## End-to-end workflows (Ternoa-specific)

## Test cases

## References

## Copyright

Copyright and related rights waived via CC0.