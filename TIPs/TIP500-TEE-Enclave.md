# Ternoa Improvement Proposal - TEE Enclaves

| Author(s)      | Amin Razavi, Prabhu Eshwarla, Mohsin Riaz |
| ----------- | ----------- |
| Created   | 8 Sep 2022       |
| TIP Number   | TIP500       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary
TBD


## Abstract
TBD


## Motivation

One of the primary challenges to be solved in implementing secret NFTs is the secure storage and retrieval of the encryption keys associated with each Secret NFT. The architecture proposed involves storage and retrieval of keys in a trusted execution environment (TEE) which is an offchain component associated with the secret NFT solution. TEE programs running on processors such as SGX provide strong trust guarantees in terms of data privacy and verification of the programs running within them. This can be achieved through techniques such as remote attestation that gives assurance that the program running inside the enclave is running on genuine TEE hardware (such as SGX), and the programs have not been modified by the TEE node operators. Data storage on TEEs are also secured by sealing them with the secure keys associated with the TEE hardware and/or author of the TEE programs.


## Additional Info

TBD

## Specification - Offchain

The interfaces supported by the TEE enclave are describe below:
```
interface {
/// Registration of enclaves

RegisterEnclave()

UpdateEnclaveParameters()

UnregisterEnclave()

/// Store a secret share
StoreSecretShare(uint256 nft_id, bytes signed_encrypted_secret_share);

/// Retrieve a secret share
RetrieveSecretShare(uint256 nft_id, bytes signed_payload_from_nft_owner);
}

```

## End-to-end workflow (Ternoa-specific)
TBD (if this section is relevant for TEE)



## Constraints
TBD

## Test cases
TBD
 
## References
TBD

## Copyright
TBD