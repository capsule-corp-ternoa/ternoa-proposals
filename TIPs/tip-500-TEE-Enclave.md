# Ternoa Improvement Proposal - TEE Enclaves

| Author(s)      | Amin Razavi, Prabhu Eshwarla, Mohsin Riaz |
| ----------- | ----------- |
| Created   | 8 Sep 2022       |
| TIP Number   | TIP-500       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary
Hardware trusted execution environment (TEE) is the most practical solution for confidentiality and privacy problem in blockchains, comparing to cryptographic algorithms like ZKP, FHE and SMPC. This document provides a standard for interfacing offchain TEE with essential components of blockchain with a focus on secret content for NFTs. 


## Abstract

The architecture proposed involves storage and retrieval of keys in a trusted execution environment (TEE) which is an offchain component associated with the secret NFT solution. TEE programs running on processors such as SGX provide strong trust guarantees in terms of data privacy and verification of the programs running within them. This can be achieved through techniques such as remote attestation that gives assurance that the program running inside the enclave is running on genuine TEE hardware (such as SGX), and the programs have not been modified by the TEE node operators. Data storage on TEEs are also secured by sealing them with the secure keys associated with the TEE hardware and/or author of the TEE programs. 

As an offchain extension of Key Management, Secure Computation and Confidential Storage for blockchains, there are at least five responsibilities of TEE : 

- Using Remote Attestation mechanism to prove the genuinity of hardware and the codes running on it to be approved by blockchain validators and registered on the blockchain

- Validation of the offchain requests from the application, comparing to onchain data (i.e NFT ownership)

- Processing the application request in a secure environment (i.e sealing the secrets)

- Providing the blockchain with verified offchain data gathered from application  (i.e Availablity of encryption key for a secure NFT, Oracle)

- Secure distributed backup and secure migration of secrets to other TEE machines


## Motivation
TEE is a decendant of HSM and TPM, with an affordable cost and flexibility of executing general purpose applications rather than just cryptographic functions. 

In spite of decades of scientific and technologic efforts for securely outsourcing the computation tsks and storage to untrusted parties, we don't have any real-world feasible cryptographic protocol yet, and the available candidates need more decade of research and development. Hardware based security however has been the ultimate solution for critical areas of financial and military applications, in modern Android mobile devices, the TEE is already unknowingly used every day, by millions of people as an HSM equivalent, through the use of a Trusted Application (TA) providing the Android KeyMaster functionality. Recently there hase been increasing attention to TEE as a source of trust for executing smart-contracts in blockchains.

While Basic NFTs merely represent ownership, Secure NFTs can furtherly contain sensitive information or allow access to services which can only be accessed by the respective owner. One of the primary challenges to be solved in implementing secret NFTs is the secure storage and retrieval of the encryption keys associated with each Secret NFT.

Leveraging TEEs to bring a confidentiality aspect to NFTs, enables a variety of different use cases across different segments to provide their services more efficiently and securely. With the help of Secure NFTs, blockchains can enable real-world services such as Digital Rights Management, Secret Markets and Access control.


### Threat Model

Major goal is to ensure the confidentiality of user data hosted by an untrusted server node. We assume a strong adversary with privileged access to OS and storage, who can not only monitor the content of all server’s memory, disk and communication, but also actively tamper with it. However, the adversary cannot access enclaves provided by TEE. In particular, data and computation inside an enclave are protected with respect to the confidentiality and integrity. 

We exclude TEE side-channel attacks from our scope, since these vulnerabilities are implementation specific and we can adopt more secure TEEs when needed. 
We do not consider the confidentiality of metadata and coarse statistical properties, such as the name of files and urls, and the length of values. In addition, we do not pursue strict indistinguishability of encrypted data and storage operations, since it usually leads to impractical performance penalty. Instead, we aim to provide operational data confidentiality where the information that the adversary learns is a function of data operations that have been performed. 
Since the adversary do not have the Data Encryption Key (DEK), it cannot perform arbitrary data operations of its choice.

Note that other security guarantees, such as data integrity and protection from denial of service or TEE/Adversaries collusion, should be considered by designers. In fact, the complexity of additionally protecting data integrity also depends on the design choices.

### Cluster Network
TBD

### Threshold Scheme
TBD

## Specification - Offchain

### APP/SDK Interface

```javascript
interface {
    /// Store a secret share
    StoreSecretShare(   uint32 nft_id, 
                        bytes encrypted_secret_share, 
                        bytes signature);

    /// Store a secret share
    ChangeSecretShare(  uint32 nft_id, 
                        bytes encrypted_secret_share, 
                        bytes signature);


    /// Retrieve a secret share
    RetrieveSecretShare(uint32 nft_id,
                        bytes optional_data,
                        bytes signature);
}

```

### Blockchain Interfaces (SGX-Pallet)[TIP-800]

``` javascript

// Add an enclave on blockchain
RegisterEnclave(bytes enclave_ias_report, bytes encalve_url);

// Remove an enclave from blockchain
UnregisterEnclave(bytes encalve_url)

```

### Extrinsics
``` javascript

// sgx-pallet : Scheduled keep-alive report to blockchain
// unsigned extrinsic
xt_heartbeat(bytes activity_proof);

// secret_nft-pllet, capsule-pallet : secret-share received,stored and sealed on enclave
// signed extrinsic
xt_secret(bytes xt_)

```

## End-to-end workflow (Ternoa-specific)
TBD (if this section is relevant for TEE)


## Constraints
TBD

## Test cases
- Wallet/SDK stores secret-share to an encalve 
- Wallet/SDK changes secret-share on encalve 
- Wallet/SDK retrieves stored secret-share from enclave

- Wallet/SDK stores secret-share to cluster with success reports
- Wallet/SDK changes secret-share on cluster with success reports
- Wallet/SDK retrieves stored secret-share from cluster
 
## References
TBD

## Copyright
TBD
