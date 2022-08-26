# Basic NFT Royalties Technical Specifications

## Title
TSP Number: TSP-002
Authors: @markopoloparadox, @peshwar9
Status: Call for Feedback
Created: 2022-08-25
Reference Implementation [Link to a first reference implementation]

## Summary
A summary of the standard and the addressed issue. TODO

## Motivation
The motivation should describe what motivated the development of the standard as well as why particular decisions were made. TODO

## Specification
### Blockcahin Extrinsic Interfaces:
```json
[
  {
    "module": "nft",
    "call": "set_royalty",
    "description": "Sets the royalty of an NFT",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "The ID of the NFT that needs a change in royalty value.",
        "constraints": [
          "The NFT needs to be owned by the caller.",
          "The NFT needs to be created by the caller.",
          "The NFT needs to be not listed for sale.",
          "The NFT needs to be not a capsule.",
          "The NFT needs to be not delegated."
        ]
      },
      {
        "name": "royalty",
        "type": "Permill",
        "description": "The cut that the creator will receive on every secondary sale. 0% is 0 and 100% is 1000000.",
        "constraints": [
          "Minimum is 0 and Maximum is 1000000."
        ]
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "NFTRoyaltySet",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that the royalty was updated for."
          },
          {
            "name": "royalty",
            "type": "Permill",
            "description": "Represents the new royalty percentage cut."
          }
        ]
      }
    ],
    "errors": [
      "nft::NFTNotFound",
      "nft::NotTheNFTOwner",
      "nft::NotTheNFTCreator",
      "nft::CannotSetRoyaltyForListedNFTs",
      "nft::CannotSetRoyaltyForCapsuleNFTs",
      "nft::CannotSetRoyaltyForDelegatedNFTs"
    ]
  }
]
```

### SDK Storage Getters