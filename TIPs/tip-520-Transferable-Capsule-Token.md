---
tip: TIP-520
title: Transferable Capsule Token
version: 1.0
category: NFT, Token
authors: Benjamin Arthuys <benjamin@capsule-corp.io>
created: 2022-10-06
---

# Transferable Capsule Token (TCT)

## Simple Summary

Ternoa has created the "Transferable Capsule Token" which is a new kind of token that allow users to store unlimited confidential data transferable on time.

## Abstract

Think of capsule as enhanced form of NFT which can take any number of files in encrypted media. Issuer should be able to convert his NFT into TCT and to be able to add limitless datas. 

## Motivation

Existing storage solutions are limited and does not provide automated transfers services.

- Centralized storage solutions mean we have to be confident on the company that manage storage in term of security, maintenance and ethic.
- Data Privacy is something foundamental for humanity. Only decentralized solutions can provide real data privacy allowing users to get ownership of their own datas.
- Data transmission is something we all think about but no secured and automated services exist.

For all those points, Ternoa created TCT to bring a powerful solution to solve data issues.

## Specification

### External Interfaces

onchain interfaces:

```rust
interface { 
  
}
```

### Existing Interfaces changed

```rust
interface {
  
}
```

## Constraints

## Additional Info

## Metadata

metadata example

```json
{
	
}
```

## End-to-end workflows (Ternoa-specific)

**TCT Minting Workflow:**

1. User selects data for storing privately in a TCT.
2. The Wallet or dApp use the Ternoa SDK to generate a key pair for each TCT.
3. The private key of the generated keypair is used to encrypt data.
4. The encrypted data are stored on IPFS and its content id (CID) is recorded.
5. The CID of the encrypted data is used to construct the offchain metadata json file
6. The offchain metadata file is stored on IPFS, and its content id (CID) is used to trigger an extrinsic on the blockchain to mint a TCT.
7. The id of the new TCT is obtained from the blockchain by the wallet/Dapp using Ternoa SDK. 
8. The secret key used to encrypt the data is split into shares using a threshold secret scheme (such as Shamir Secret Shares). 
9. Discovery of enclave locations (TBD)
9. Each threshold share is then stored on a different enclave along with the associated TCT id. The number of threshold shares of the encryption key should be equal to the number of TEE enclaves running as offchain components.
10. Each TEE enclaves then posts a transaction on the blockchain confirming receipt of the secret share for a given TCT.

**TCT Displaying Workflow (Owner View)**

1. User requests the wallet or dApp to decrypt data associated with the TCT owned by them.
2. The wallet/dApp sends a request to each enclave with a signature generated from the user's account key, asking for the secret share associated with a given TCT.
3. The TEE enclave verifies if the requestor of the secret share is the owner of the TCT. Invalid requests are rejected. If the ownership is successfully verified from the blockchain, the secret share is sent to the requesting wallet/dApp.
4. The wallet/dApp receives the minimum threshold of shares from the set of available enclaves, and  reconstructs the encryption key for the secret. 
5. The encryption key is used to retrieve the original secret from IPFs which is then displayed to the user.

**The enclave program should run within a TEE environment with the following characteristics:**

1. The enclave program is deployed on a set of TEE-enabled hardware. It is recommended to have a minimum set of 5 TEE machines running in a cluster.
2. The enclave program should support interfaces to store and retrive secrets. The details of the interfaces provided by the TEE enclave program is described later in this section.
3. The enclave program should support remote attestation, which is a servie offered by the TEE processor vendor.
4. Each of the TEE machines should store one of the secret shares associated with an NFT. If the TEE cluster comprises of 5 machines as recommended, there would be a set of five secret shares generated each of which is stored on one of the TEE machines through a request to the TEE enclave program.
5. There should be a stand-by TEE cluster of 5 machines that can be manually activated if the primary TEE cluster malfunctions. There should be ability for the secret shares stored in the primary TEE cluster to be securely transferred to the backup TEE cluster, so that the secondary TEE cluster can be activated in case of contingencies.

## Test cases

- User **SHOULD** mint a TCT directly
- Owner **SHOULD** convert a Basic NFT to TCT
- Owner **SHOULD** decrypt content data associated with the TCT
- Owner **SHOULD** transfer a TCT using tansmission protocols
- Owner **SHOULD NOT** transfer a TCT wihtout using transmission protocols
- Owner **SHOULD NOT** list TCT in the marketplace
- Owner **SHOULD NOT** trade TCT in any marketplace
 
## References

## Copyright

Copyright and related rights waived via CC0.