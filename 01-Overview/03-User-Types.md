# User Types

Twyne's ecosystem revolves around two primary user types: Credit LPs and Borrowers. This document explains their roles, incentives, and interactions within the protocol.

## Credit LPs (CLPs)

### Definition

A Credit LP (CLP) is a lending market user who lends assets in exchange for yield but does not use these assets as collateral to borrow against them. They are essentially passive lenders in the external lending protocol.

### Characteristics

- Already have assets deposited in an external lending protocol (like Aave, Compound, or Euler)
- Hold receipt tokens (like aTokens, cTokens, or eTokens) from these protocols
- Do not use their deposit as collateral for borrowing
- Seek to maximize yield on their deposited assets

### Role in Twyne

In Twyne, Credit LPs:

1. Stake their lending market receipt tokens into Twyne's Intermediate Vaults
2. Allow their unused borrowing power to be delegated to borrowers
3. Earn additional yield beyond what the external lending protocol provides
4. Retain all original benefits and security guarantees from the external lending protocol

### Incentives

Credit LPs are motivated by:

- Higher yields on their deposits without taking on additional risk
- Maintaining the same security assumptions they already accepted when depositing into the external lending protocol
- No active management required (passive yield enhancement)

### Risk Considerations

It's important to note that Credit LPs:

- Never allow direct access to their underlying assets
- Only delegate borrowing power, not the assets themselves
- Maintain all security assumptions of the original lending market
- Are protected by Twyne's liquidation mechanisms that trigger before external protocol liquidations

## Borrowers

### Definition

A Borrower in Twyne is a user who wants to access higher LTV ratios than what's allowed by external lending protocols. They use their own assets as collateral but leverage the borrowing power delegated by Credit LPs.

### Characteristics

- Deposit collateral assets into their own Collateral Vault
- Set their desired liquidation LTV (higher than external protocol limits)
- Borrow assets from external lending protocols
- Actively manage their position

### Role in Twyne

In Twyne, Borrowers:

1. Deploy a Collateral Vault through the Twyne factory
2. Deposit collateral assets into their vault
3. Reserve credit (borrowing power) from Intermediate Vaults
4. Borrow assets from the external lending protocol
5. Maintain their position health by managing their LTV ratio

### Incentives

Borrowers are motivated by:

- Access to higher LTV ratios than external protocols allow
- Improved capital efficiency for their collateral
- More flexible borrowing positions
- Reduced liquidation risk (compared to trying to maintain high LTVs directly on external protocols)

### Risk Considerations

Borrowers must be aware that:

- They pay interest both to the external protocol and to Credit LPs
- Their positions can be liquidated if they breach Twyne's liquidation thresholds
- Internal liquidations in Twyne are designed to be more punitive than external liquidations
- They must actively monitor and maintain their position health

## Relationship Between User Types

The relationship between Credit LPs and Borrowers is mutually beneficial:

1. Credit LPs provide unused borrowing power to Borrowers
2. Borrowers pay interest to Credit LPs for using this borrowing power
3. External lending protocols benefit from increased utilization

This creates a symbiotic ecosystem where:

- Credit LPs earn more yield
- Borrowers access more flexible borrowing conditions
- Overall capital efficiency in DeFi increases

## Example Scenario

Let's walk through a concrete example:

1. Alice deposits 10 ETH in Euler but doesn't want to borrow against it
2. Alice becomes a Credit LP by staking her eETH in Twyne's Intermediate Vault
3. Bob wants to borrow with a higher LTV than Euler allows
4. Bob deploys a Collateral Vault through Twyne's factory
5. Bob deposits 5 ETH as collateral and sets his desired liquidation LTV
6. Bob's Collateral Vault reserves credit from the Intermediate Vault (using Alice's unused borrowing power)
7. With the combined collateral (Bob's ETH + reserved credit), Bob can borrow more than he could directly on Euler
8. Bob pays interest to Euler for his loan and to Alice for using her borrowing power
9. Alice earns extra yield without taking on additional risk

## Next Sections

- [Credit Reservation](../02-Core-Mechanics/01-Credit-Reservation.md): Learn how Twyne calculates the amount of credit to reserve.
- [Interest Rates](../02-Core-Mechanics/02-Interest-Rates.md): Understand how interest is calculated and distributed.
