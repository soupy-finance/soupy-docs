---
sidebar_posiition: 2
slug: /litepaper
---

# Litepaper 

## Introduction 
Current traders in the cryptocurrency ecosystem must choose between two options: highly-performant centralized exchanges (CEXs) packed with features, and fully decentralized exchanges (DEXs) which have significant performance overheads and lack most of these features. The most critical missing features include full order books, cross-chain liquidity, cross-chain deposits and withdrawals, and fiat onboarding. While many of these capabilities may never be possible using smart contracts on general purpose blockchains, blockchain frameworks have advanced to the point where all of these can be implemented in a robust manner on a dedicated chain. Of course, it would be remiss to not emphasize the downsides of CEXs that warrant decentralization in the first place. In the past several months alone, CEXs have frozen withdrawals, disabled trading without warning, added tremendous latency to deposits during volatility, and outright malfunctioned at the worst times. At Soupy, we are using the Cosmos SDK to develop a state-of-the-art decentralized exchange within a custom blockchain, with the express goal of integrating all aforementioned features. A particular emphasis will be placed on acting as a hub for trading between all major blockchains. 

## The Chain: Noodle
The custom blockchain upon which all of Soupy runs is a Cosmos SDK proof-of-stake chain that uses Tendermint consensus to achieve sub-second finality. Called Noodle, this blockchain is planned to consist of 10+ validators staking the native SOUP token for proportional validation power. The code is split among several modules each handling aspects of the exchange, including asset bridging, live oracles, and complex order matching. More modules will be added as development progresses to facilitate our long-term goals, including features such as perpetual futures, margin trading, asset lending, a fully-backed decentralized stablecoin, and more. 

One obstacle to the usage of some DEXs is the requirement that the user have a wallet for the specific chain on which the exchange runs and that they confirm every single action through their wallet application. While Soupy will support both browser extension (Keplr, Cosmostation) and hardware wallets, the recommended wallet system will be our seamless on-site wallet that is stored encrypted in local storage. This offers security comparable to browser extension wallets, but it also creates a user experience similar to logging in to a CEX and allows the user to perform fast trading actions without needing to manually confirm all Noodle transactions.

## Asset Bridging
One of the core utilities of CEXs is the ability to deposit assets from almost any chain, trade for another asset, and seamlessly withdraw to an entirely different chain. This is a major feature we will support on Soupy, in a manner that allows for nearly endless new chain integrations. Doing so is not necessarily trivial, but we have developed a system that will make it posible in a decentralized and secure manner.

### Deposits
All supported blockchains will have a smart contract vault that users send their assets to in order to deposit on the Soupy exchange. Noodle validators will monitor for deposit events on all chains and, upon observing a deposit, post their observation on the Noodle chain. Once over half of the validation power has observed a deposit the associated account will credited those assets on the exchange. All of this will occur in a matter of seconds after the deposit transaction is finalized on the associated chain. In the rare event more than half of all validators miss a deposit event, the user can trivially call the refresh function on the smart contract to alert the validators to check for the associated deposit. 

### Withdrawals
Somewhat more complicated is the system for withdrawals. In a basic sense, all that is needed is for the smart contracts to receive an authenticated transaction from the Noodle validator set. However, because the set of validators and their associated weights can change at any time, keeping the smart contracts on all chains synchronized is nontrivial. To solve this, Soupy will utilize a set of “guardians”, which is a set of staked, withdrawal-specific validators that are not necessarily the same for each chain and may even differ from the Noodle validator set. These guardians observe withdrawal transactions on the Noodle chain and post their signatures of the withdrawal data on the Noodle chain. Users can then submit the signatures for at least of half of the guardians to the smart contract to complete their withdrawal.

Since withdrawals are one of the most dangerous aspect of any exchange security-wise, initializing and updating the guardian sets must be done carefully while maintaining decentralization. New chains will be added by publishing the associated smart contract (including an initial guardian set), passing a governance proposal, and waiting for the guardians to lock their stake on Noodle. The guardian set will be updated on a chain when a governance proposal modifies it and at least half of the chain’s guardians authenticate the set update. Guardians will also authenticate set updates for their chain when a guardian is slashed by Noodle’s validators for misbehaving.

## Price Oracles
One of the biggest obstacles to running a fully-featured exchange on existing blockchains is the lack of fast, inexpensive oracles. Most oracle systems update slowly (every ten seconds or slower) to reduce transaction fees, and those that are sufficiently fast incur significant costs. However, many oracle applications like perpetual futures and oracle-priced automatic market makers (oracle AMMs or oAMMs) need price updates about as quickly as users can send transactions. To facilitate price updates with these speeds, low costs, and flexibility regarding which data is tracked, Noodle validators will act like oracles. They will post price updates for every tracked asset on each block, with the transaction fee waived. On-chain applications that use this data will perform median aggregation on all of the prices provided by validators for a particular asset and utilize the variance to infer volatility.

## The Exchange


### Advanced Spot Trading
A major aspect lacking from almost all DEXs is a traditional trading experience that offers price charts, order books, fast trade execution, and live updates. Of course, many of these are simply not possible in layer one solutions due to technical limitations. Soupy’s underlying blockchain technology allows for sub-second trade finality and price updates, along with fast order matching in a traditional order book format. To improve liquidity and reduce spreads, an oracle AMM will exist alongside the traditional order books. In fact, the oAMM’s liquidity will simply appear to traders as large orders on the books whose prices are updated every block. This allows anyone to become a passive market maker by simply depositing their assets into the oAMM pool and letting the exchange itself manage the prices. Additionally, Soupy will utilize the concept of universal pools that consist of a single asset and provide liquidity to any market in which that asset is present (e.g. USDI).

### Simple Swaps
More than just a UI abstraction for the order books and AMMs, simple swaps will allow users to swap and bridge assets without needing to have a Noodle account. Instead of submitting a normal deposit transaction to the smart contract, users will submit a “depositAndSwap” or “depositAndBridge” transaction in which the observations made by the Noodle validators will automatically trigger trade and withdrawal actions on the Noodle chain. This will help expand the utility of Soupy without adding significant development or complexity costs. Note that bridging will only be supported for assets that are natively supported on multiple chains, e.g. USDC.

### Perpetual Futures
Currently, perpetual futures see more volume than their associated spot markets across the crypto ecosystem. This is because they offer users the ability to speculate on prices with leverage and hedge their positions on other markets without holding the underlying assets. Since Soupy already features live price oracles for arbitrary assets, supporting perpetual futures for cryptocurrencies, stocks, and commodities with combined order books/oAMMs would be relatively simple addition with immense value for traders. The exact mechanisms behind leverage, account health, and cross-collateralization are not finalized yet, but the core plan is to support leveraged positions up to 100x on certain markets with USDI as collateral and the ability to offset a futures position (no risk of liquidation) by holding or short selling the corresponding asset via lending.

### Lending 
The ability to lend and borrow assets creates significant advantages for traders and those looking to earn yields in DeFi. Borrowing can allow a user to hedge funding rate arbitrages in perpetual futures, increase leverage in spot trading, and access deep liquidity for specific assets without suffering losses to spreads or slippage. Likewise, lending allows users who are holding a particular asset to earn passive income in addition to profits they gain from price increases. Similarly to the perpetual futures, the exact details of Soupy’s lending protocol will finalized at a later date. It will have a fixed utilization curve that determines interest rates and be closely tied to other platforms on Soupy like the USDI stablecoin, spot trading, and futures trading.

## The Token: SOUP
SOUP is the primary token that powers the Noodle blockchain and the Soupy exchange. It serves as the staking token for validators and guardians, the governance token for proposals that update and manage the exchange via the DAO, and as a store of value that provides trading fee discounts.

### The DAO
The Soupy DAO will be responsible for adding and removing supported blockchain bridges, adding new officially supported assets and markets, adjusting values for mechanics like trading fees, funding rate curves, and interest curves, and managing the Soupy treasury. Similarly to other DAOs, changes can be initiated by creating a governance proposal using a small deposit of SOUP, the majority of SOUP votes must support the proposal and exceed a threshold, and successful proposals will have their deposit refunded. 

### Tokenomics
SOUP will have a total supply of 100 million tokens, with 50 million circulating after the ICO+IDO. The tokens will be divided in the following manner:
- 20 million in ICO
- 30 million in IDO
- 30 million in DAO treasury
- 20 million initially held by the founding team

#### The initial coin offering (ICO)
The ICO will occur over a one-week period on Ethereum in which users will deposit USDC into a smart contract that holds 20 million SOUP tokens. Users will be able to withdraw their USDC any time before the ICO ends, and upon the conclusion of the sale users with nonzero deposits will be able to redeem their proportional share of the SOUP ERC-20 tokens. The initial price of SOUP will be deemed to be (USDC deposits) / (20 million SOUP).

The USDC earned in the sale will be used for four purposes:
- A professional audit of the Soupy codebase
- Initial market liquidity
- An insurance fund
- Marketing and contributor recruitment

#### The initial DEX offering (IDO)
Since Soupy is itself a DEX, the IDO will occur on the day of the mainnet launch in the form of a simple constant product AMM. Liquidity will slowly be added to the pool at a ratio that maintains the ICO price or higher, using some of the USDC earned from the ICO and 30 million total SOUP. USDC earned in this sale will be used to further boost initial exchange liquidity and add to the insurance fund.

## The Stablecoin: USDI
Recent events involving algorithmic stablecoins have made it very clear that the stability only holds when there is investor confidence in underlying ecosystem. Here at Soupy, we feel such mechanisms are unsustainable for any long-term project, so our cross-chain stablecoin will instead be fully backed and decentralized using a similar approach to MakerDAO’s DAI. Since the stablecoin will be closely tied to the Soupy DEX, the collateral will offer significant utility in the form of swap and loan liquidity. Interest and income generated from this utility will allow the stablecoin to resist inflation of the asset (USD) it is pegged to and generate returns for holders. Additionally, unlike many other stablecoins, any holder will be able to instantly swap $1 worth of USDI for $1 worth of the assets backing it at any time. Users performing this action reduces the debts of minters and actually reduces minters' liquidation risk, unlike algorithmic stablecoins which become increasingly unstable as more users redeem. 

### Collateralization
There will be two mechanisms to mint USDI: 
- Depositing crypto assets as collateral at a 150% ratio
- $1 to $1 exchange in USDC (i.e. adjusted for USDI price deflation)

When minting USDI using arbitrary assets as collateral, users must deposit 150% of the value they wish to mint based on the current oracle prices. The minter will be responsible for maintaining a 120% collateralization ratio, and failure to do so will result in liquidation. Liquidators will be able to either close the position by depositing the USDI debt (receiving a 20% profit) or take over the position by providing enough new collateral to exceed 120%. A combination of the insurance fund and deflation vault will be used to guarantee at least 5% profit on a liquidation to ensure liquidators are always incentivized, even with extreme volatility. 

The 1:1 minting mechanism using USDC acts mainly as an rapid onboarding process for users who deposit USDC into the exchange, as most markets will use USDI as the quote asset to avoid fragmenting liquidity. Minting in this manner will be effortless and incur no fees (besides the miniscule blockchain transaction fee). 

### Inflation resistance and stable yields 
All USDI collateral will be automatically used as both lending liquidity and market making liquidity in X/USDI markets. In fact, the market making liquidity is the primary redemption mechanism for general USDI users. When a user buys an asset on one of these markets the USDI used will be burned to reduce the debts of minters and part of the associated trading fees will be placed into the “deflation vault”. When assets are borrowed via the lending protocol, 80% of interest generated will be given to minters and 20% will be added to the deflation vault. 

The price of USDI on Soupy at any point in time will be equal to `(supply + deflation vault value) / supply`, meaning users who hold USDI will have their wealth grow relative to USD as the deflation vault receives income. Users will also be able to stake their USDI for significantly higher returns, a process which adds that USDI to the global AMM pool. This will boost liquidity on all official markets and generate returns in the form of trading fees, spreads, and futures funding. This creates a mechanism by which the stablecoin can offer high, stable yields in the long run without relying on constant investor funding or Ponzi mechanics. 

## Fiat onboarding
Soupy is currently exploring potential partners who would offer USD ⇔ USDI or USD ⇔ USDC swaps at little to no cost. These would be third party companies operating as independent entities with no direct integration into the DEX to avoid centralization issues.

## Custom markets and cross-chain tokens as a service
When launching a new token for a project, developers must generally choose a native chain for that token. One goal of Soupy is to radically redesign that process by allowing any token to be cross-chain by default. The exact details of the service are TBD, but the core idea is anyone will be able to create a token and associated market on the chain for a small DAO-controlled cost. It will then be seamlessly bridgeable to all supported chains via the existing deposit/withdrawal contracts acting as mint/burn sites. For existing tokens, users will be able to create a custom market that supports tokens deposited from a designated native chain. These custom markets will have the opportunity to become officially supported markets with DAO approval.

## The Timeline
**Q2 2022**
- Litepaper publication
- Noodle devnet with spot trading

**Q3 2022**
- Noodle testnet with swaps, spot trading, and full cross-chain deposit/withdrawal support
- SOUP token ICO

**Q4 2022**
- Noodle mainnet genesis with swaps, spot trading, and full cross-chain support
- SOUP token IDO on the Soupy exchange
- USDI stablecoin launch
- Lending platform launch

**Q1 2023**
- Perpetual futures launch
- Fiat on/offboarding launch
- Custom market and cross-chain token service launch
