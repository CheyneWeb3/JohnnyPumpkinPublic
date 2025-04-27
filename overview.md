**JonnysMarketplace: Features & User Guide**  

---

## 📖 Overview  
JonnysMarketplace combines fixed-price sales and timed auctions for both ERC-721 and ERC-1155 NFTs. It handles escrow, platform fees, user statistics, collection whitelisting, user bans/suspensions, and token recovery—all in one contract.

---

## ✨ Key Features  

- **Fixed-Price Sales**  
  - Sellers list NFTs at a set price  
  - Buyers pay exactly that price to purchase  

- **Timed Auctions**  
  - Sellers choose an auction duration (1–7 days)  
  - Bidders compete: highest bidder wins once time expires  

- **Unified ERC-721 & ERC-1155 Support**  
  - Single marketplace handles both standards seamlessly  

- **Platform Fees & Treasury Splits**  
  - Default fee: 2.5% of sale/auction proceeds  
  - Fee split: configurable percentage to a “SuperFund” treasury; remainder to a “Profits” treasury  

- **User & Global Statistics**  
  - Tracks per-user: listings created, items sold/bought, ETH value traded  
  - Tracks global counts: active listings, active auctions, total sales, total fees, total volume, unique users  

- **Collection Whitelist**  
  - Only approved NFT contracts may be listed  
  - Owner can add or remove collections  

- **Ban & Suspension Controls**  
  - Owner can suspend or ban misbehaving users, preventing any marketplace actions  
  - Operators can force-cancel listings  

- **Recovery Functions**  
  - Owner can recover any stray ERC-20, ERC-721 or ERC-1155 tokens sent to the contract  

- **Deprecation & Redirect**  
  - Owner can mark the marketplace deprecated and point to a new marketplace address  

---

## 🛠️ How to Use  

### For Sellers  
1. **List an NFT**  
   - Choose your token (ERC-721 or ERC-1155).  
   - Specify a price (fixed sale) or select “auction” plus duration (1–7 days).  
   - NFT is transferred into contract escrow.  

2. **Cancel a Listing**  
   - For fixed-price listings: cancel any time before a sale.  
   - For auctions: cancel once at least half the auction time has passed and if no bids exist.  
   - NFT returns to your wallet.  

### For Buyers & Bidders  
1. **Buy (Fixed Sale)**  
   - Pay exactly the listed price.  
   - NFT is delivered instantly.  

2. **Bid (Auction)**  
   - Submit a bid higher than the current top bid.  
   - Previous top bid is automatically refunded.  

3. **Claim (Auction Winner)**  
   - Once auction time ends, highest bidder finalizes purchase.  
   - NFT is transferred and funds (minus fees) go to seller.  

### For Operators & Admins  
1. **Force-Cancel Listings**  
   - Immediately cancel any sale or auction, refunding bidders and returning NFTs.  

2. **Manage Whitelist**  
   - Add or remove NFT contracts that may be listed.  

3. **Ban or Suspend Users**  
   - **Suspend**: temporarily block user actions.  
   - **Ban**: permanently block user actions.  

4. **Set Fees & Treasuries**  
   - Adjust fee percentage and how it splits between treasuries.  
   - Update SuperFund and Profits treasury addresses.  

5. **Recover Stray Tokens**  
   - Retrieve any ERC-20, ERC-721 or ERC-1155 tokens accidentally sent to the contract.  

6. **Deprecate & Redirect**  
   - Mark marketplace as deprecated and emit new marketplace address for front-end migration.  

---

## 📊 Dashboard & Reporting  

Use these publicly exposed values to power your UI or analytics:

- **Active Listings** & **Active Auctions**  
- **Total Sales**, **Total Volume Traded**, **Total Fees Collected**  
- **Unique User Count**  
- **Per-User Stats**: listings created, items sold/value, items bought/value  

---

## 🔔 Events & Notifications  

Front-ends and indexers can listen for marketplace events:

- **Listing Created**, **Sale Completed**, **Bid Placed**, **Auction Claimed**  
- **Listing Cancelled** (by seller or operator)  
- **User Banned/Suspended**, **Collection Whitelisted/Removed**  
- **Token Recovery** actions and **Marketplace Deprecated**  

---

With this guide, everyone—from casual buyers to contract administrators—can understand the capabilities of JonnysMarketplace and integrate it smoothly into their dApp. Enjoy trading!
