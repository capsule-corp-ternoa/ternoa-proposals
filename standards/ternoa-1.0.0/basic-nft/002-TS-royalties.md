# Basic NFT Royalties Technical Specifications

## Title
TSP Number: TSP-002
Authors: @markopoloparadox, @peshwar9, @ipapandinas
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

### SDK

> #### [Events](#events)
> #### [Extrinsics helpers](#extrinsics-helpers)
> #### [Utils](#utils)

#### Events

```typescript
import { Event } from "@polkadot/types/interfaces/system";
export declare enum EventType {
  NFTRoyaltySet = "nft.NFTRoyaltySet",
}
export declare class BlockchainEvent {
  type: EventType;
  raw: Event;
  section: string;
  method: string;
  constructor(raw: Event, type: EventType);
  static fromEvent(event: Event): BlockchainEvent;
}

/**
 * This class represents the on-chain NFTRoyaltySetEvent event.
 */
export declare class NFTRoyaltySetEvent extends BlockchainEvent {
  nftId: number;
  royalty: number;
  /**
   * Construct the data object from the NFTRoyaltySetEvent event
   * @param event The NFTRoyaltySetEvent event
   */
  constructor(event: Event);
}
```

#### Extrinsics helpers

```typescript
import { IKeyringPair } from "@polkadot/types/types";

export declare type TransactionHashType = `0x${string}`;
export declare enum WaitUntil {
  BlockInclusion = 0,
  BlockFinalization = 1,
}

/**
 * @name setRoyaltyTx
 * @summary       Creates an unsigned unsubmitted Set-Royalty Transaction Hash.
 * @param id      The ID of the NFT.
 * @param amount  The new royalty value.
 * @returns       Unsigned unsubmitted Set-Royalty-NFT Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const setRoyaltyTx: (id: number, amount: number) => Promise<TransactionHashType>;
/**
 * @name setRoyalty
 * @summary           Sets the royalty of an NFT.
 * @param id          The ID of the NFT.
 * @param amount      The new royalty value.
 * @param keyring     Account that will sign the transaction.
 * @param waitUntil   Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns           NFTRoyaltySetEvent Blockchain event.
 */
export declare const setRoyalty: (id: number, amount: number, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<NFTRoyaltySetEvent>;
```

#### Utils

```typescript
/**
 * @name formatPermill
 * @summary         Checks that percent is in range 0 to 100 and format to permill.
 * @param percent   Number in range from 0 to 100 with max 4 decimals.
 * @returns         The formated percent in permill format.
 */
export declare const formatPermill: (percent: number) => number;
```
