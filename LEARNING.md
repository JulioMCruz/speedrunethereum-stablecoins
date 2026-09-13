# Challenge 07: Stablecoins

- Challenge: https://speedrunethereum.com/challenge/stablecoins
- Date: 2026-09-13
- Repository: https://github.com/JulioMCruz/speedrunethereum-stablecoins
- Demo: https://speedrunethereum-stablecoins-dusky.vercel.app/dashboard
- Verified engine: https://sepolia.etherscan.io/address/0x69bba10c2fb04ced6331322d95e0d9e920f7a64d#code

## System model

MyUSD is minted against ETH collateral with a minimum 150% position ratio. Debt is represented as shares, so interest can accrue globally by increasing a debt exchange rate instead of updating every borrower. A separate staking contract pays a savings rate, while a rate controller coordinates borrowing and savings incentives.

## Invariants implemented

- Minting and collateral withdrawal must leave the account at or above the 150% collateral ratio.
- Debt shares convert to current debt through the accrued exchange rate.
- Interest accrues from elapsed time and the configured annual borrow rate before debt-changing operations.
- The borrow rate cannot be lower than the savings rate.
- Repayment is capped at the current debt, and liquidation clears debt shares atomically.
- Liquidators repay unsafe debt and receive debt-equivalent collateral plus 10%, capped by the collateral available.

## Public deployment

- RateController: `0x7ed42cb47a0c0563ca407769ffbe01a7be29750e`
- MyUSD: `0x9fd56b9c83c870a842f19c6ce3aebb45571f2f43`
- DEX: `0xb260a88af65d4e13074d5e936f9ec1c328728d08`
- Oracle: `0x79220a49c683ab425bf2eb11a34fb9a7ebbb54dd`
- MyUSDStaking: `0x250f3a77e5fd014dc798807a857245659b2e27d3`
- MyUSDEngine: `0x69bba10c2fb04ced6331322d95e0d9e920f7a64d`

The live setup deposits 0.02 ETH as collateral, mints about 12.43 MyUSD, and seeds the DEX with 0.005 ETH plus the corresponding MyUSD. This makes the public dashboard observable immediately.

## Evidence

- Official Hardhat suite: 45/45 passing.
- Solidity compilation, Next.js typecheck, and production build passed.
- All six contracts are verified on Sepolia Etherscan.
- On-chain calls confirmed 0.02 ETH collateral, active debt shares, and 0.005 ETH DEX liquidity.
- The Vercel dashboard returned HTTP 200 and loaded in Chrome against Sepolia.

## Reusable lessons

- Share accounting makes protocol-wide interest accrual constant-time, but every conversion needs one consistent exchange-rate scale and explicit rounding expectations.
- Accrue interest before minting, repaying, liquidating, or changing rates so each transition uses current debt.
- Solvency checks should use current debt value rather than stored principal.
- Borrow and savings rates form coupled incentives: borrowing changes supply, while savings changes demand.
- A spot-price DEX and unrestricted rate controller are teaching mechanisms. Production designs need robust oracles, bounded governance, bad-debt handling, partial liquidations, and reentrancy defenses.
