# Basic NFT Technical Specifications

## Title
PSP Number: TSP-721
Authors: @markopoloparadox, @peshwar9
Status: Call for Feedback
Created: 2022-08-25
Reference Implementation [Link to a first reference implementation]

## Summary
A summary of the standard and the addressed issue.

## Motivation
The motivation should describe what motivated the development of the standard as well as why particular decisions were made.

## Specification
### Interfaces:
create_nft
```sh
{
  "name": "create_nft",
  "parametrs": [
    {
      "name": "offchain_data",
      "type": "BoundedVec<u8, OffchainDataLimit",
      "descriptions": "Represents location where the NFT metadata is stored"
    },
    ....
  ],
  "descriptions",
  "events": [
    { 
      "module": "NFT",
      "name": "transfer"
      "Output stuff" [
        {
          "name": "nftid",
          "type": "u32",
          "description": "It repeends NFT ID"
        }
      ]
    }
  ],
  "errors": ["Bla1", "Bla2"]

}
```
burn_nft
transfer_nft
set_nft_mint_fee

### Getters


### Extensions

#### Delegation
delegate_nft

#### Royalties
set_royalties

#### Collections
create_collection
burn_collection
close_connection
limit_collection

### Offchain Metadata

### Errors
  CannotTransferListedNFTs,
  CannotBurnListedNFTs,
  CannotDelegateListedNFTs,
  CannotSetRoyaltyForListedNFTs,
  CannotTransferDelegatedNFTs,
  CannotBurnDelegatedNFTs,
  CannotSetRoyaltyForDelegatedNFTs,
  CannotTransferCapsuleNFTs,
  CannotBurnCapsuleNFTs,
  CannotDelegateCapsuleNFTs,
  CannotTransferSoulboundNFTs,
  CannotSetRoyaltyForCapsuleNFTs,
  CannotTransferNFTsToYourself,
  CannotDelegateNFTsToYourself,
  CollectionLimitIsTooLow,
  CollectionLimitIsTooHigh,
  NFTNotFound,
  NFTNotFoundInCollection,
  NFTBelongToACollection,
  NotTheNFTOwner,
  NotTheNFTCreator,
  NotTheCollectionOwner,
  CollectionNotFound,
  CollectionIsClosed,
  CollectionHasReachedLimit,
  CollectionHasReachedMax,
  CollectionIsNotEmpty,
  CollectionLimitAlreadySet,
  CollectionNFTsNumberGreaterThanLimit,
  CannotAddMoreNftsToCollection,

## Tests
If applicable, please include a list of potential test cases to validate an implementation.

## Copyright
Each PSP must be labeled as placed in the public domain.