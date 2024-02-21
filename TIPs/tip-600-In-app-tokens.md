# Ternoa Improvement Proposal - __In-app tokens__

| Author(s)      | Prabhu Eshwarla |
| ----------- | ----------- |
| Created   | 20 Sep 2022       |
| TIP Number   | TIP-600       |
| Version   | v0.1       |
| Requires   |       |
| Status | In Progress       |
| Category   | Fungible tokens       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

A simple module for dealing with fungible assets that are meant for use within a dApp. 

## Abstract

In-app tokens feature enable asset management of fungible assets with a fixed supply, within a single decentralized application (dApp). The features include __asset issuance__ (aka __mint__), __asset transfers__, __asset freezing__, __asset destruction__ (aka __burn__). 

## Motivation

Several decentralized applications have a need to issue new assets and to move them between accounts. The goal of such assets is not to transfer them across chains or to list them on exchanges, but a way to engage and reward users of the dApps for performing certain actions within the dApp.  

## Specification

### External interfaces

__In-app token__ should support the following onchain interfaces:
``` rust
interface {
  /// Interface Id: TIP600-01
  /// Description: User can create a new asset class, taking the required deposit
	/// - `asset_id`: The identifier of the new asset. This must not be currently in use to identify
	/// an existing asset.
	/// - `admin`: The admin of this class of assets. The admin is the initial address of each
	/// member of the asset class's admin team.
	/// - `min_balance`: The minimum balance of this new asset that any single account must
	/// have. If an account's balance is reduced below this, then it collapses to zero.  
  /// Constraint(s): The origin must be Signed and the sender must have sufficient funds free.
  create(asset_id: AssetId, admin: AccountId, min_balance: Balance);
  
  /// Interface Id: TIP600-02
  /// Description: Transfer sender's assets to another account
	/// - `asset_id`: The identifier of the asset to have some amount transferred.
	/// - `target`: The account to be credited.
	/// - `amount`: The amount by which the sender's balance of assets should be reduced and
	/// `target`'s balance increased. The amount actually transferred may be slightly greater in
	/// the case that the transfer would otherwise take the sender balance above zero but below
	/// the minimum balance. Must be greater than zero.  
  /// Constraint(s): Origin must be Signed.
  transfer(asset_id: AssetId, target: AccountId, amount: Balance)

  /// Interface Id: TIP600-03
  /// Description: Destroys an entire asset class; called by the asset class's Owner
    /// - `asset_id`: The identifier of the asset to be destroyed. This must identify an existing
	/// asset.  
  /// Constraint(s): The origin must conform to `ForceOrigin` or must be Signed and the sender must be the
    /// owner of the asset `asset_id`.
  /// NOTE: It can be helpful to first freeze an asset before destroying it so that you
  /// can provide accurate witness information and prevent users from manipulating state
  /// in a way that can make it harder to destroy.
  destroy(asset_id: AssetId, witness: DestroyWitness)

  /// Interface Id: TIP600-04
  /// Description: Increases the asset balance of an account; called by the asset class's Issuer.
	/// - `asset_id`: The identifier of the asset to have some amount minted.
	/// - `beneficiary`: The account to be credited with the minted assets.
	/// - `amount`: The amount of the asset to be minted.  
  /// Constraint(s): The origin must be Signed and the sender must be the Issuer of the asset `id`.
  mint(asset_id: AssetId, beneficiary: AccountId, amount: Balance)

  /// Interface Id: TIP600-05
  /// Description: Decreases the asset balance of an account; called by the asset class's Admin
  /// - `asset_id`: The identifier of the asset to have some amount burned.
	/// - `target`: The account to be debited from.
	/// - `amount`: The maximum amount by which `who`'s balance should be reduced.
  /// Constraint(s): Origin must be Signed and the sender should be the Manager of the asset `id`.
  burn(asset_id: AssetId, target: AccountId, amount: Balance)  

  /// Interface Id: TIP600-06
  /// Description: Transfers between arbitrary accounts; called by the asset class's Admin.
	/// - `asset_id`: The identifier of the asset to have some amount transferred.
	/// - `source`: The account to be debited.
	/// - `dest`: The account to be credited.
	/// - `amount`: The amount by which the `source`'s balance of assets should be reduced and
	/// `dest`'s balance increased. The amount actually transferred may be slightly greater in
	/// the case that the transfer would otherwise take the `source` balance above zero but
	/// below the minimum balance. Must be greater than zero.  
  /// Constraint(s): Origin must be Signed and the sender should be the Admin of the asset `id`.
  force_transfer(asset_id: AssetId, source: AccountId, dest: AccountId, amount: Balance)  

  /// Interface Id: TIP600-07
  /// Description: Disallows further `transfer`s from an account; called by the asset class's Freezer account.
	/// - `asset_id`: The identifier of the asset to be frozen.
	/// - `target`: The account to be frozen.  
  /// Constraint(s): Origin must be Signed and the sender should be the Freezer of the asset `id`.
  freeze(asset_id: AssetId, target: AccountId)  

  /// Interface Id: TIP600-08
  /// Description: Allow unprivileged transfers from an account again.
	/// - `asset_id`: The identifier of the asset to be frozen.
	/// - `target`: The account to be unfrozen.  
  /// Constraint(s): Origin must be Signed and the sender should be the Admin of the asset `id`.
  thaw(asset_id: AssetId, target: AccountId)  

  /// Interface Id: TIP600-09
  /// Description: Changes an asset class's Owner; called by the asset class's Owner.
	/// - `asset_id`: The identifier of the asset.
	/// - `owner`: The new Owner of this asset.  
  /// Constraint(s): Origin must be Signed and the sender should be the Owner of the asset `id`.
  transfer_ownership(asset_id: AssetId, owner: AccountId)  

  /// Interface Id: TIP600-10
  /// Description: Set the metadata of an asset class; called by the asset class's Owner.
	/// - `asset_id`: The identifier of the asset to update.
	/// - `name`: The user friendly name of this asset.
	/// - `symbol`: The exchange symbol for this asset.
	/// - `decimals`: The number of decimals this asset uses to represent one unit.  
  /// Constraint(s):  Origin must be Signed and the sender should be the Owner of the asset `id`.
  set_metadata(asset_id: AssetId, name: String, symbol: String, decimals: u8)  

}

```

## Additional Info

In-app tokens cannot be transferred across chains , or listed for trade on an exchange. For these features, smart-contracts should be used.

## Metadata

NA

## Test cases

* User can create a new asset class
* User transfer assets from own account to another account
* Owner can destroy an asset class
* Asset class's issuer can increase asset balance of an account
* Asset class's admin can reduce asset balance of an account
* Asset class's admin can transfer assets between two arbitrary accounts
* Asset class's freezer account can disallow further transfers from an account
* Admin of asset class can unfreeze account and allow transfers again
* Asset class's owner can transfer asset to another account
* Asset class's owner can set metadata of asset class

### Issue:
Sending gToken to a wallet without the minimum required amount of CAPS results in an error.

#### Context:
In the process of sending gToken, it's crucial to ensure that the recipient wallet holds at least 0.015 CAPS to receive the gToken. Without this minimum CAPS balance, the recipient wallet won't be able to accept the gToken.

#### Proposed Solution:
1. Check if the recipient wallet has a sufficient CAPS balance before sending gToken.
2. If the CAPS balance is insufficient, first send the necessary amount of CAPS to the recipient wallet, then proceed to send the gToken.

#### Associated Risks:
Sending gToken to wallets without a sufficient CAPS balance can lead to blockchain congestion and abuse issues. It's advisable to exercise caution when sending gToken and ensure that recipient wallets meet the minimum CAPS balance requirements.

#### Minimum Required CAPS Amount:
0.015 CAPS
 
## References
TBD

## Copyright
TBD
