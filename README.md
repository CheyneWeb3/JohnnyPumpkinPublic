# Johnny Pumpkins NFT Marketplace  
*A full-length contract guide and feature compendium.*

---

## 1. Introduction  

Johnny Pumpkins NFT Marketplace is an on-chain trading venue for ERC-721 and ERC-1155 tokens. All listing logic, bidding, fee collection, compliance actions, and treasury routing live inside a single smart contract, guaranteeing that no central operator can tamper with trades, seize assets, or hide fees. This document explains **every feature, workflow, and safeguard** in plain language—use it as a reference whether you are a creator, collector, moderator, or integrator building on top.

---

## 2. Core Philosophy  

| Principle | How the Marketplace Implements It |
|-----------|------------------------------------|
| **Self-custody** | NFTs and ETH are always held by immutable escrow code; no off-chain wallets. |
| **Transparency** | Fees, treasury splits, bans, suspensions, and operator actions emit events—anyone can audit in real-time. |
| **Granular user control** | Sellers get optional cancellation, buyers receive instant refunds, moderators have force-cancel tools. |
| **Upgrade friendliness** | A built-in “deprecate & redirect” flag points users to future contracts without hiding historical data. |

---

## 3. Asset Support  

| Token Standard | Typical Use-case |
|----------------|------------------|
| **ERC-721** | 1/1 art, profile-picture collections, high-end game items. |
| **ERC-1155** | Multi-edition prints, fungible game consumables, bulk airdrops. |

Only collections that have been **whitelisted** by the contract owner can be traded—this blocks spam mints and malicious files from entering the feed.

---

## 4. Listing Types  

### 4.1 Fixed-Price Listings  
- **Launch speed:** 1 transaction.  
- **Price lock:** Cannot be changed once posted.  
- **Buyer flow:** Buyer calls `buy`, pays the exact ETH requested, receives the NFT instantly.  
- **Seller payout:** Net proceeds (after fees) hit the seller wallet atomically—no waiting period.  

### 4.2 Timed English Auctions  
- **Duration:** 1 – 7 whole days (seller chooses).  
- **Reserve price:** Seller’s `price` parameter becomes the opening bid.  
- **Immutable end:** No extension mechanics; this discourages last-second bot sniping and shortens final-price anxiety.  
- **Bidding logic:** Cumulative (see next section).  
- **Settlement:** Highest bidder claims the NFT once the timer expires.  

---

## 5. Bidding Mechanics in Depth  

### 5.1 Cumulative Bids  
Traditional auctions force each new bid to exceed the old one in a single transaction, refunding the previous highest bid in full. Johnny Pumpkins improves on this by letting bidders **top-up** their existing contribution:  

1. Place an initial bid—e.g., 1 ETH.  
2. Later, send an extra 0.3 ETH; the contract adds it to your previous stake, making your new effective bid 1.3 ETH.  

Benefits:  
- Saves gas (no complete un-stake + re-stake).  
- Reduces refund traffic and chain congestion.  
- Mirrors real-world auction psychology (“I’ll add a bit more”).

### 5.2 Minimum Increment  
There is **no fixed percentage step**—your combined bid simply has to beat the current highest by at least 1 wei. Creators can set de-facto increments by picking an appropriate reserve.

### 5.3 Out-Bids & Refund Ledger  
When you lose the top slot:  
- Your previous bid is written into `pendingRefunds[listingId][you]`.  
- You can withdraw immediately (no 24-hour cooling-off).  
- Choose between:  
  * **Single withdrawal** (`withdrawRefund(listingId)`)—good if you lost one auction.  
  * **Multi-sweep** (`withdrawAllRefunds(listingIds[])`)—ideal for power bidders who lost many at once.

### 5.4 Example Timeline  
| Time | Action | Contract State |
|------|--------|----------------|
| 12:00 | Seller lists NFT as 3-day auction, reserve 1 ETH. | highestBid 0 |
| 12:05 | Bidder A posts 1.2 ETH. | highestBid 1.2 (A) |
| 12:15 | Bidder B posts 1.4 ETH. | Refund bucket for A = 1.2 |
| 12:30 | Bidder A adds 0.3 ETH. | highestBid 1.5 (A) |
| Day 3 12:00 | Auction ends. | A is winner, other bidders refundable |
| 12:05 | A calls **claim**. | NFT → A, payout → Seller minus fee |

---

## 6. Claim Helpers—Never Lose a Win  

| Helper | What It Returns | Why It Matters |
|--------|-----------------|----------------|
| `hasUnclaimedAuction(address)` | `true/false` | Build wallet pop-ups (“You have 1 unclaimed NFT!”). |
| `unclaimedAuctionIds(address)` | Array of IDs | Feed these directly into a **Claim All** button. |

The helpers loop only over the live `activeListingIds` array—so calls are inexpensive and safe for dApps to poll.

---

## 7. Seller Powers & Protections  

| Feature | Details |
|---------|---------|
| **Batch Listing** | Post many NFTs—mixing ERC-721 and ERC-1155, fixed and auction—in one transaction. |
| **Seller-Cancel (if enabled)** | Pay a small flat fee (default 0.001 ETH) to pull back a listing. Auction must be at least halfway finished and have 0 bids. |
| **Unsold Auction Reclaim** | If an auction ends with zero bids, seller retrieves the NFT instantly (no fee). |
| **Per-User Stats** | `stats[you]` logs items sold, bought, volume—use for badges or dashboards. |

---

## 8. Buyer Comfort Features  

1. **Atomic purchase**—no need to “approve, then buy”; just one call.  
2. **Instant refunds**—ETH never vanishes into a refund queue.  
3. **Batch buy**—sweep multiple fixed-price listings in one go.  
4. **Claim helpers**—avoid “ghost wins” locked in escrow.  

---

## 9. Fee System & Economics  

| Variable | Default | Cap | Purpose |
|----------|---------|-----|---------|
| `feePercent` | 2.5 % | 10 % | Marketplace operating fee. |
| `feeSplitSuperFund` | 30 % of fee | 0-100 % | Share that flows to community/marketing treasury. |
| Remainder | 70 % | — | Profit treasury for protocol incentives. |
| Seller-cancel flat fee | 0.001 ETH | — | Anti-spam + revenue without hurting buyers. |

**Distribution order in every sale:**  
1. Calculate fee = price × feePercent ÷ 10 000.  
2. Send Super-Fund share to `superFundTreasury`.  
3. Send remainder to `profitsTreasury`.  
4. Pay net amount (price − fee) to seller.

These transfers are atomic; failure in any step reverts the entire sale, preventing partial payouts.

---

## 10. Compliance & Moderation  

### 10.1 Whitelist  
Only collections flagged `whitelistedCollections[nft] = true` can be listed. This single switch blocks scam clones and malware.

### 10.2 User Discipline  
| Action | Who Can Trigger | Effect |
|--------|-----------------|--------|
| **Suspend** | Owner | Temporary freeze—user cannot interact but listings stay live. |
| **Ban** | Owner | Full lockout + address added to public banned array. |
| **Unban / Unsuspend** | Owner | Restores privileges. |

### 10.3 Operator Role  
Operators (set by owner) can:  
- `forceCancel` any listing.  
- No authority over treasury or fees—limited blast radius.  

All compliance moves emit events, so block explorers, Discord bots, or analytics sites can reflect them within seconds.

---

## 11. Safety Engineering  

| Guard | Why It Matters |
|-------|----------------|
| **ReentrancyGuard** | Stops malicious callbacks during ETH/NFT transfers. |
| **Pausable** | One-click global freeze if a vulnerability is discovered. |
| **Escrow Isolation** | Each listing’s assets are tracked by ID; unrelated listings unaffected by a bug in one record. |
| **Recovery Calls** | Owner can return accidentally sent tokens without deploying a patch. |
| **Fee Cap (10 %)** | Prevents “governance capture” from raising fees to 100 %. |

---

## 12. Analytics & Integration End-Points  

| Call | Description |
|------|-------------|
| `activeListingCount()` | Quick integer for home-page counters. |
| `getActiveListingIds(offset, limit)` | Paginated feed—offset 0, limit 50 for fast infinite scroll. |
| `operatorCount()`, `getAllOperators()` | Build a moderator leaderboard. |
| `whitelistedCollectionCount()` | Good for UI filters. |
| `stats[address]` | Returns a struct—ideal for user profile pages. |
| Event stream | Subscribing to `Listed`, `BidPlaced`, `Bought`, etc. gives real-time updates without REST queries. |

---

## 13. Typical User Flows  

### 13.1 Artist Bulk Drop  
*Goal:* release 1 000 in-game skins, half at fixed price, half by auction.  

1. Approve Johnny Pumpkins to transfer the ERC-1155 contract.  
2. Prepare an array of 1 000 `BatchListingInput` structs.  
3. Call `batchList` once.  
4. Fixed-price items sell within minutes; auctions run for 3 days.  
5. Creator dashboard shows cumulative ETH and remaining active auctions.  

### 13.2 Collector Night-Raid  
*Goal:* Win multiple auctions, withdraw lost bids quickly.  

1. Place bids on 10 listings.  
2. Sleep. Wake up to see wins and losses.  
3. Call `unclaimedAuctionIds(myAddress)`, iterate through IDs:  
   * Call `claim` on the 3 won ones.  
4. Pass the same 10 IDs into `withdrawAllRefunds`, reclaim ETH from the 7 losses.  
5. Spend refunds on new items.

### 13.3 Moderator Takedown  
*Goal:* Remove a scam collection.  

1. Operator runs a script fetching listing IDs belonging to the malicious NFT address.  
2. Calls `forceCancel` for each—each step refunds bidders and returns NFTs to scammer wallet.  
3. Owner adds the address to banned list; front-ends auto-hide future attempts.  

---

## 14. Roadmap Highlights (No Promises, Only Ideas)  

| Idea | What It Would Add |
|------|-------------------|
| **Layer-2 Mirrors** | Duplicate listings on Arbitrum/Base for low-fee bidding, settle back on mainnet. |
| **Royalty Splitter** | Optional ERC-2981 enforcement for collections that want creator royalties. |
| **Private (zk) Auction Mode** | Commit-reveal bidding so amounts remain hidden until the reveal phase. |
| **In-Contract Sweeps** | One transaction to claim NFTs *and* withdraw refunds. |
| **Advanced Search Index** | On-chain tagging for traits, enabling permissionless rarity tools. |

---

## 15. Frequently Asked Questions  

**How big can a batch listing be?**  
Gas limits decide. As a rule of thumb, a few hundred ERC-721s or a bit fewer 1155s fit comfortably under typical block limits.

**Can someone snipe the auction at the last second?**  
Yes, if they beat the highest bid before the endTime. Because timers do not auto-extend, early bidders often bid higher in advance. Front-ends can surface “Time Remaining” prominently.

**What if my wallet fails while claiming?**  
The NFT stays safe in escrow; the claim call can be repeated anytime by the rightful winner.

**Do I need to “approve” before each sale?**  
For ERC-721, token-specific approvals happen automatically when you list (using `safeTransferFrom`). For ERC-1155, front-ends usually help you grant a single operator approval to the marketplace first.

**Can the owner freeze my funds?**  
No. The owner can pause new actions but cannot seize existing escrowed NFTs or bids—those settle as soon as the pause lifts.

**What stops the owner from raising fees to the cap every week?**  
Fees changes are on-chain and extremely visible; migrating owner keys to a DAO or multi-sig is recommended to align incentives long-term.

---

## 16. Glossary  

| Term | Meaning |
|------|---------|
| **Listing** | A record representing an NFT for sale or auction. |
| **ActiveListingIds** | Dynamic array holding every live listing ID. |
| **Escrow** | The contract’s own balance of NFTs/ETH held on behalf of users. |
| **Operator** | A moderator granted limited powers (mostly cancellation). |
| **Super-Fund Treasury** | Wallet for community growth initiatives. |
| **Profits Treasury** | Wallet that captures protocol revenue. |
| **PendingRefunds** | Mapping storing ETH owed to out-bid participants. |

---

## 17. Event Reference (Quick Copy-Paste for Indexers)  

| Event | Fires When |
|-------|------------|
| **Listed** | New listing or auction posted. |
| **BidPlaced** | Valid bid accepted. |
| **Bought** | Fixed-price item purchased. |
| **Claimed** | Auction winner receives NFT. |
| **Cancelled** | Seller cancel, unsold reclaim, or self-cancellation. |
| **ListingForceCancelled** | Operator/owner cancels a listing. |
| **ListingCancelledWithFee** | Paid seller-cancel event. |
| **RefundWithdrawn** | User pulls out-bid funds. |
| **UserBanned / UserUnbanned** | Compliance action. |
| **CollectionWhitelisted / CollectionRemoved** | Collection status change. |
| **MarketplacePaused / MarketplaceUnpaused** | Global on/off switch toggled. |

---

## 18. Final Thoughts  

Johnny Pumpkins NFT Marketplace marries **deep functionality** with **on-chain integrity**:

* Sellers enjoy batch tools, optional cancellation, and clear fee maths.  
* Buyers get cumulative bidding, instant refunds, and easy claim helpers.  
* Moderators wield force-cancel and public ban tools to keep everyone safe.  
* Builders plug into a consistent event stream and clean view functions.  

Everything lives on Ethereum, guarded by battle-tested OpenZeppelin libraries and bolstered by simple, human-readable mechanics. Whether you’re listing a single pumpkin-themed masterpiece or building a dashboard for thousands of daily trades, Johnny Pumpkins is ready to serve—no seeds, no strings, just pure, tasty decentralisation.  

**Happy trading—and may the auctions be ever in your flavour!** 🎃
