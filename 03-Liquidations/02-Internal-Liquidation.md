# Internal Liquidation

Internal liquidation is Twyne's primary liquidation mechanism, designed to protect positions from being liquidated by external lending protocols. This document explains how internal liquidation works, who can perform it, and what happens during the process.

## Purpose of Internal Liquidation

Twyne's internal liquidation serves several critical purposes:

1. **Prevent External Liquidations**: Internal liquidations trigger before external lending protocols would liquidate positions, which are typically more punitive and less capital-efficient.

2. **Maintain Protocol Solvency**: Liquidations ensure Credit LPs' funds remain safe by resolving unhealthy positions.

3. **Capital Preservation**: The internal liquidation mechanism aims to preserve capital within the Twyne ecosystem rather than losing it to external liquidators.

4. **Orderly Resolution**: Provides a more controlled process for resolving unhealthy positions compared to external liquidations.

## Liquidation Mechanism: Position Inheritance

Unlike many DeFi protocols that liquidate positions by selling collateral to cover debt, Twyne uses a unique "position inheritance" mechanism:

1. **Liquidator Inherits Position**: When a position becomes unhealthy, any liquidator can call a function to take over (inherit) the borrower's entire position.

2. **Ownership Transfer**: The Collateral Vault's ownership transfers from the original borrower to the liquidator.

3. **All Assets and Liabilities Transfer**: The liquidator inherits all collateral and debt of the original position.

4. **Credit Reservation Preserved**: The credit reserved from the Intermediate Vault remains the same, continuing to back the position.

This approach has several advantages:
- No need for price oracles to determine fair liquidation prices
- No risk of failed liquidations due to slippage
- More capital-efficient as value remains within the protocol
- Liquidator can improve position health gradually rather than in a single transaction

## Liquidation Conditions

A position is eligible for internal liquidation when the `_canLiquidate()` function returns true, which happens when either:

1. **Twyne LTV is exceeded**: `Debt > Twyne_LTV * User_Collateral`
2. **External protocol safety threshold is approached**: `Debt > Safety_Buffer * External_LTV * Total_Collateral`

For a detailed explanation of these conditions, see [Liquidation Conditions](./01-Liquidation-Conditions.md).

## Liquidation Process

### Step 1: Position Becomes Unhealthy

A position can become unhealthy due to:
- Interest accrual reducing user collateral
- Market price movements
- User withdrawing too much collateral
- User borrowing too much debt

### Step 2: Liquidation is Triggered

Any user (liquidator) can call the `liquidate()` function on an unhealthy Collateral Vault.

### Step 3: Position Ownership Transfers

The liquidator becomes the new borrower, taking over full control of the Collateral Vault, including:
- All deposited collateral
- All borrowed debt
- All reserved credit from the Intermediate Vault

### Step 4: Liquidator Restores Position Health

The liquidator must now make the position healthy again. This can be done by:
- Adding more collateral
- Repaying some of the debt
- Some combination of both actions

## Incentives for Liquidators

Why would users want to become liquidators in Twyne? There are several incentives:

1. **Access to Leveraged Positions**: Liquidators can acquire leveraged positions without needing to set them up from scratch.

2. **Discounted Value**: The positions are typically "in the money" when accounting for the total collateral and debt. A rational liquidator would only liquidate if they can make the position healthy at a cost less than its value.

3. **Capture Rebalancing Opportunities**: As interest accrues, excess credit builds up that can be released through rebalancing after making the position healthy.

4. **Liquidation Bonuses**: In some implementations, Twyne could include an explicit liquidation bonus by adding a small discount to the position's effective value.

## Potential Liquidator Strategies

Liquidators might employ several strategies to profit from liquidation:

1. **Add Minimal Collateral**: Calculate the minimum amount of collateral needed to make the position healthy and add only that amount.

2. **Repay Partial Debt**: Instead of adding collateral, repay some of the debt to bring the LTV back under the threshold.

3. **Arbitrage Across Protocols**: Use flash loans or other capital to perform the liquidation and immediately extract value by rebalancing positions across protocols.

4. **Hold as Long-term Position**: Simply hold the position if it aligns with the liquidator's investment strategy.

## Implementation Considerations

### Gas Efficiency

Since internal liquidation requires multiple transactions (liquidate, then make position healthy), gas costs are an important consideration. Implementations should optimize gas usage to ensure liquidations remain profitable.

### Prevention of External Liquidations

If no one liquidates an unhealthy position and it eventually gets liquidated by the external protocol, there's a risk of value loss to the Credit LPs. Twyne should include mechanisms to incentivize timely liquidations to avoid value-extracting losses to impact the protocol.

### Liquidation Monitoring

The protocol or third-party services should provide monitoring tools to identify liquidatable positions, making it easier for potential liquidators to find opportunities.

## Edge Cases and Risks

### Liquidator Abandons Position

If a liquidator takes over a position but doesn't make it healthy, the position could still be liquidated by the external protocol. This risk is mitigated by economic incentives – the liquidator has a clear incentive to make the position healthy to avoid losing their capital.

### Rapid Price Movements

In cases of extreme market volatility, positions might be liquidated by external protocols before internal liquidators can act. Twyne's safety buffer helps mitigate this risk.

### Failed Health Restoration

If a liquidator tries but fails to make a position healthy (e.g., due to continuing price drops), they may suffer losses. This risk is part of the liquidator's calculation when deciding whether to perform a liquidation.

## Related Topics

- [Liquidation Conditions](./01-Liquidation-Conditions.md): When positions can be liquidated
- [External Liquidation](./03-External-Liquidation.md): What happens when external protocols liquidate positions
- [Position Management](../02-Core-Mechanics/04-Position-Management.md): How users can manage positions to avoid liquidation
- [Rebalancing](../02-Core-Mechanics/03-Rebalancing.md): How excess credit is released after liquidation
