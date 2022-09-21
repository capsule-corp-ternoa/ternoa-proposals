# Ternoa Improvement Proposal - NFT Auctions
|Author(s) | Aman Lalwani, Ghali El Ouarzazi |
| :------ | :------ |
| Created   | 19 Sep 2022 |
| TIP Number | TIP-300 |
| Version | v0.1 |
| Requires |  |
| Status | In Progress |
| Category | NFT |
| Discussions-To | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions/13 | 

## Simple Summary
Auctions can be an exciting way to sell an NFT. An auction is a type of sale where the bidders bid on the NFTs listed under Auction. 

## Abstract
In the objective of giving different ways to sell an NFT, auctions could be a great way to hype the project or an NFT.  Auctions are a type of sale where the NFT seller sets a minimum price they are willing to sell their NFT and by defining the time duration. Buyers can place bids with the amount they are willing to pay for the NFT as long as it is above the minimum price. At the end of the time period, the NFT is sold to the highest bidder. 

## Motivation
The main objective of auctions is to give sellers flexibility to sell their NFTs online. On the other hand the auctions can help maximize the value of NFTs as a subsequent bid will always be higher than the preceeding bid. Plus, this could be a mechanism for a project to discover price by auctioning the genesis collection rather than simply selling it.
And for users, who see adherence to a decentralized and disintermediated process as essential to the basic concept of blockchain, the only legitimate auction process for NFTs is on-chain. Conducting auctions on-chain lends the bidding process many of the benefits typically associated with blockchain-based transactions. For instance, auctions conducted on a blockchain are auditable: every bid is public and permanently recorded, which makes bids more secure and transparent. 

## Specification
### External interfaces
Auctions support the following onchain interfaces:
```rust
interface {
    /// Interface Id: TIP300-01
    /// Description: User can create an auction for an owned NFT by specifying the nft id, the marketplace id, the start block, the end block the start price, the optional buy it now price (price at which the NFT can be bought without bidding).
    /// Constraint(s): 
    ///     - User must own the NFT.
    ///     - Start block must not start in the past.
    ///     - End block must not be before start block.
    ///     - Duration must not be too long.
    ///     - Duration must not be too short.
    ///     - Auction start block must not be too far in the future.
    ///     - If buy it price is provided, it must be greater than start price.
    ///     - NFT must be available (not listed, capsule, delegated, soulbound, rented, ...).
    ///     - The marketplace must exist.
    ///     - User must be allowed to list on the marketplace.
    ///     - The start price must cover the flat commission fee if it exists.
    ///     - User must have enough tokens to cover the listing fee if it exists.
    create_auction(nft_Id: NFTId, marketplace_Id: MarketplaceID, start_block: BlockNumber, end_Block: BlockNumber, start_Price: BalanceOf, buy_ItPrice: Option<Balance>);

    /// Interface Id: TIP300-02
    /// Description: User can cancel an auction that has not started yet.
    /// Constraint(s): 
    ///     - User must be owner of the NFT.
    ///     - Auction must not have started.
    cancel_auction(nft_Id: NFTId);
    
    /// Interface Id: TIP300-03
    /// Description: User can end an auction if it is in the extended period (period starting when a bid was made at the end of the auction to avoid sniping).
    /// Constraint(s): 
    ///     - User must be owner of the NFT.
    ///     - Auction must be in the extended period.
    ///     - Auction must have at least one bid.
    end_auction(nft_Id: NFTId);
    
    /// Interface Id: TIP300-04
    /// Description: User can bid for an existing auction. If a bid already exist, it will be updated. If the bid was made at the end of the auction, the duration will be extended by the grace period duration.
    /// Constraint(s): 
    ///     - Bidder must not be auction owner.
    ///     - Auction must have started.
    ///     - Bid must be higher than previous highest bid.
    ///     - Bid must be higher or equal to starting price.
    ///     - Bidder must have enough funds to cover for the specified bid amount.
    add_bid(nft_Id: NFTId, amount: BalanceOf);
    
    /// Interface Id: TIP300-05
    /// Description: User can remove a bid if the auction is not in the end period (period a little before the end of the auction).
    /// Constraint(s): 
    ///     - Auction must not be in the end period.
    ///     - User must have made a bid.
    remove_bid(nft_Id: NFTId);
    
    /// Interface Id: TIP300-06
    /// Description: User can buy the NFT for the buy it price amount without having to bid.
    /// Constraint(s): 
    ///     - The auction must have a buy it price.
    ///     - Auction must have started.
    ///     - Buy it price must be higher than current highest bid. 
    ///     - User must have enough funds to cover the buy it price amount.
    ///     - User must not be auction creator.
    buy_it_now(nft_Id: NFTId);
    
    /// Interface Id: TIP300-07
    /// Description: A user that made a bid but did not win the auction can claim the bidded amount to retrieve funds to his account.
    /// Constraint(s): 
    ///     - User must have a pending claim (bid that did not win and is not retrieved).
    claim();
}
```
## Constraints
- To create an auction, the user must own the NFT.
- To create an auction, the start block must not start in the past.
- To create an auction, the end block must not be before start block.
- To create an auction, the duration must not be too long.
- To create an auction, the duration must not be too short.
- To create an auction, the start block must not be too far in the future.
- To create an auction and if buy it price is provided, it must be greater than start price.
- To create an auction, the NFT must be available (not listed, capsule, delegated, soulbound, rented, ...).
- To create an auction, the marketplace must exist.
- To create an auction, the user must be allowed to list on the marketplace.
- To create an auction, the start price must cover the flat commission fee if it exists.
- To create an auction, the user must have enough tokens to cover the listing fee if it exists.
- To cancel an auction, it must not have started.
- To end an auction, it must be in extended period.
- To end an auction, it must have at least one bid.
- To add a bid, bidder must not be auction owner.
- To add a bid, auction must have started.
- To add a bid, the amount must be higher than previous highest bid.
- To add a bid, the amount must be higher or equal than starting price.
- To add a bid, the bidder must have enough funds to cover for the specified bid amount.
- To remove a bid, auction must not be in the end period
- To remove a bid, user must have made a bid.
- To buy an auctioned NFT (buy it now), the auction must have specified a buy it now price.
- To buy an auctioned NFT (buy it now), the auction must have started.
- To buy an auctioned NFT (buy it now), the buy it price must be higher than the highest bid.
- To buy an auctioned NFT (buy it now), the user must have enough funds to cover the buy it price amount.
- To buy an auctioned NFT (buy it now), the user must not be auction creator.
- To claim an amount of tokens, the user must have a pending claim (bid that did not win and is not retrieved).

## Additional Info
- If a bid list becomes too long, to add a new bid, the first one will get "dropped" and the user will have to claim the amount to retrieve his funds even if the auction has not finished yet.

## Metadata

## End-to-end workflows (Ternoa-specific)

The following is the workflow proposed for creating an auction:

1. User has already minted an NFT and is aware of the NFT Id
2. User creates an auction by calling the "create_auction" function and specifying the nft id, the marketplace id, the start block, the end block, start price, and optionally the buy it now price.
3. The auction is now created and will start at the specified start block.

The following is the workflow proposed for cancelling an auction:

 1. User has an auction that did not start and knows the corresponding NFT Id.
 2. The user calls the "cancel_auction" function specifying the NFT Id.
 3. The auction is now cancelled.

The following is the workflow proposed for ending an auction:

 1. User has an auction that has started and knows the corresponding NFT Id.
 2. An other user has made a bid at the end of the auction and made it extended.
 3. The auction owner calls the "end_auction" function specifying the NFT Id.
 4. The auction is now finished and the NFT / funds have been transfered.

The following is the workflow proposed for adding a bid:

 1. Bidder knows the id of a started auction.
 2. Bidder calls the "add_bid" function specifying the NFT Id and an amount greater than the highest bid / starting price.
 3. The bid has been accounted for.

The following is the workflow proposed for updating a bid:

 1. Bidder knows the id of a started auction and has already bidded X Tokens.
 2. Bidder calls the "add_bid" function specifying the NFT Id and X + Y tokens.
 3. The bidders has Y tokens taken and added to his existing bid if his bid still exist, else X + Y will be taken and he will have to claim his previous existing bid.
 4. The bid is now updated.

The following is the workflow proposed for removing a bid:

1. Bidder knows the id of a started auction that is not in the ending period and has already bidded an amount of token.
2. Bidder calls the "remove_bid" function specifying the NFT Id.
3.  The bid is now removed and user has been refunded.

The following is the workflow proposed for Buy it Now:

1. Buyer knows the id of a started auction that speicified the buy it now price. The buy it price is greater than current highest bid / start price and user has enough funds.
2. Buyer calls the "buy_it_now" funtion specifying the NFT Id.
3. The auction is now finished the the NFT / price has been transfered.

The following is the workflow proposed for claiming an amount that was bidded but did not win the auction:

1. Bidder has made a bid that did not win the auction.
2. Bidder calls the "claim" function.
3. Funds bidded are given back to the bidder.

## Test cases
- User can create an auction for an NFT.
- User can cancel the auction.
- User can end the auction in the extended period.
- User can add a bid.
- User can remove a bid before the end period.
- User can buy an NFT at Buy it Now Price if it was specified.
- An auction should complete automatically at the end of the period.
- A user must be able to claim his bidded amount if he did not win the auction.

## References
TBD

## Copyright
TBD
