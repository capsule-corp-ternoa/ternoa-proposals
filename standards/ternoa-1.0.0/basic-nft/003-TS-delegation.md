# Basic NFT Delegation Technical Specifications

## Title
TSP Number: TSP-003
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
    "call": "delegate_nft",
    "description": "Delegates an NFT to someone or cancels a existing delegation.",
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
        "description": "The ID of the NFT that needs to be delegated or undelegated.",
        "constraints": [
          "The NFT needs to be owned by the caller.",
          "The NFT needs to be not listed for sale.",
          "The NFT needs to be not a capsule."
        ]
      },
      {
        "name": "recipient",
        "type": "Option<<T::Lookup as StaticLookup>::Source>",
        "description": "The account which is going to get the delegated NFT. If set to None it will undelegated the NFT."
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "NFTDelegated",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was delegated or undelegated."
          },
          {
            "name": "recipient",
            "type": "Option<AccountId>",
            "description": "If available, it represents the account that got the delegated NFT."
          }
        ]
      }
    ],
    "errors": [
      "nft::NotTheNFTOwner",
      "nft::CannotDelegateListedNFTs",
      "nft::CannotDelegateCapsuleNFTs"
    ]
  }
]
```

### SDK Storage Getters