
# Ternoa Improvement Proposal - Delegation

| Author(s)      | Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 15 Sep 2022       |
| TIP Number   | TIP103       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. Delegation is a way to designate someone to "lend" your NFT. This on-chain reference to an address has not direct meaning or purpose but combined with dApps feature, this can be a powerfull feature.

## Abstract

In the objective of giving control over NFTs, owners can designate someone as the delegatee of an NFT. The ownership of the NFT does not change during the delegation. The owner needs to manually undelegate his NFT to revoke the rights given to the delegatee.

## Motivation

The main objective of delegation is to give games, marketplaces, or any dApps flexibility about their NFTs. Being able to give rights and verify delegation status without changing the ownership can be really powerfull. If we take an exemple about cat breeding, I could delegate a rare cat to my friend so he can breed it with his own. A game could want to give an experience boost from level 1 to 10 and revoke it automatically when the player reaches level 10. The delegation feature can be compared to a free renting revocable at any moment.

## Specification

### Delegation External Interfaces
Delegation support the following onchain interface:
```rust
interface {
  /// Interface Id: TIP103-01
  /// Description: User set or unset the delegatee of an NFT. If a recipient is provided, the recipient will be delegatee, else delegation will be cancel / revoked.
  /// Constraint(s): 
  ///       - The user must be the owner of the NFT
  ///		    - The NFT must be available (not listed, delegated, rented, ...)
  delegate_nft(nft_id: NFTId, recipient: Option<AccountId>);
}
```

## Constraints
 - User must be owner of an NFT to set its delegatee.
 - NFT mus be free to set its delegatee.

## Additional Info

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for delegating an NFT:

 1. User has already created an NFT and knows its id.
 2. User calls the "delegate_nft" function with the NFT Id and the recipient.
 3. The recipient is now the delegatee of the NFT.

The following is the workflow proposed for revoking a delegation:

 1. User has already created an NFT and knows its id.
 2. User calls the "delegate_nft" function with the NFT Id and "None" as the recipient.
 3. The NFT is not anymore delegated.

## Test cases

* User can delegate an available NFT he owns.
* User can undelegate a delegated an NFT he owns.
 
## References
TBD

## Copyright
TBD
