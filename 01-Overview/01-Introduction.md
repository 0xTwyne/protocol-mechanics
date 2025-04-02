# Introduction to Twyne

## What is Twyne?

Twyne is a noncustodial, risk-modular credit delegation protocol implemented for the Ethereum Virtual Machine. It addresses a fundamental inefficiency in current DeFi lending markets: the unused borrowing power of lenders who deposit assets but don't borrow against them.

## The Market Inefficiency

In traditional DeFi lending protocols (like Aave, Compound, or Euler), when users deposit assets, they receive a corresponding amount of receipt tokens. These tokens can serve two purposes:

1. Provide yield from the interest paid by borrowers
2. Be used as collateral to borrow other assets

However, many DeFi users only use their deposits for the first purpose (yield) and never borrow against them. This creates an inefficiency - there is significant unused borrowing power locked in these lending protocols.

## Twyne's Solution

Twyne creates a trustless primitive that allows these passive lenders (called Credit LPs) to earn additional yield by making their unused borrowing power available to borrowers who want to access higher loan-to-value (LTV) ratios than what's allowed by the underlying lending protocols.

Importantly, this is done without shouldering any new risks onto lending market participants who don't wish to interact with Twyne. The system is opt-in for all parties.

## Key Benefits

Twyne creates value for multiple participants in the DeFi ecosystem:

1. **For Credit LPs**: Earn additional yield on idle assets with no additional risk beyond what they've already accepted by depositing in the underlying lending protocol.

2. **For Borrowers**: Access higher LTVs than what's allowed by underlying lending protocols, reducing liquidation risk and improving capital efficiency.

3. **For Lending Markets**: Increase utilization of deposited assets, leading to more efficient use of capital.

## Example Scenario

Consider a user who deposits 10 ETH in a lending protocol like Aave but doesn't intend to borrow against it. With Twyne:

1. This user can become a Credit LP by staking their Aave receipt tokens (aETH) in Twyne
2. A borrower on Twyne can leverage this Credit LP's unused borrowing power
3. The borrower can access a higher LTV than Aave would normally allow
4. The Credit LP earns extra yield without taking on additional risk
5. Aave benefits from increased utilization

This creates a win-win-win situation for all participants while maintaining the security constraints of the underlying lending markets.

## Protocol Scope

Twyne V1 focuses on providing higher LTVs for borrowers while maintaining appropriate risk management mechanisms. It integrates with established lending protocols and doesn't require any changes to them.

The protocol is designed to be modular, allowing for future expansions and integrations with other DeFi primitives.

## Next Sections

- [Protocol Architecture](./02-Protocol-Architecture.md): Learn about the different components of Twyne and how they interact.
- [User Types](./03-User-Types.md): Understand the different user roles within the Twyne ecosystem.