# Position Management

Position management in Twyne encompasses the full lifecycle of borrower positions, from creation through maintenance and eventual closure. This document explains the key operations, considerations, and best practices for managing positions in Twyne.

## Position Lifecycle Overview

A typical Twyne position goes through the following stages:

1. **Creation**: Deploying a Collateral Vault and setting initial parameters
2. **Funding**: Depositing collateral and reserving credit
3. **Borrowing**: Borrowing assets from external lending protocols
4. **Maintenance**: Monitoring health, adjusting parameters, and rebalancing
5. **Modification**: Adding/removing collateral or adjusting debt
6. **Closure**: Repaying all debt and withdrawing remaining collateral

Each of these stages involves specific operations and considerations that we'll explore in detail.

## Position Creation

### Deploying a Collateral Vault

The first step in creating a position is deploying a Collateral Vault through the Twyne factory.

### Setting Parameters

During creation, borrowers set key parameters for their position:

1. **Asset**: The collateral asset type (e.g., ETH, USDC)
2. **Protocol Pool**: The external lending protocol to integrate with
3. **Liquidation LTV (liqLTV)**: The desired Twyne liquidation LTV, which determines how high the borrowing capacity will be

## Position Funding

### Depositing Collateral

Once the Collateral Vault is deployed, borrowers can deposit collateral.

### Automatic Credit Reservation

When collateral is deposited, the vault automatically reserves the appropriate amount of credit from the Intermediate Vault based on the formula:

$$C_{LP} = \left(\frac{liqLTV_{twyne}}{\beta_{safety} \cdot liqLTV_{ext}} - 1\right) \cdot C$$

This credit reservation process is explained in detail in the [Credit Reservation](./01-Credit-Reservation.md) document.

## Borrowing

### External Borrowing

With collateral deposited and credit reserved, borrowers can borrow assets from the external lending protocol.

### Borrowing Limits

Borrowers can borrow up to the maximum determined by their Twyne liquidation LTV:

$$\text{maxBorrow} = liqLTV_{twyne} \cdot C$$

Where:
- $liqLTV_{twyne}$ = Twyne liquidation LTV
- $C$ = User collateral

In practice, borrowers should maintain some buffer below this maximum to avoid liquidation due to interest accrual or market fluctuations.

## Position Maintenance

### Monitoring Position Health

Borrowers should regularly monitor the health of their positions. The health is determined by:

1. **Twyne Health**: The ratio of maximum borrow (based on Twyne LTV) to current debt
2. **External Protocol Health**: The ratio of maximum safe borrow (based on external LTV) to current debt

A position is considered healthy if both health factors are greater than 1.0.

### Rebalancing

As interest accrues, borrowers should periodically rebalance their positions to release excess credit back to the Intermediate Vault. This process is explained in detail in the [Rebalancing](./03-Rebalancing.md) document.

## Position Modification

### Adding Collateral

Borrowers can add collateral to improve their position health.

### Withdrawing Collateral

Borrowers can withdraw collateral if their position remains healthy.

### Repaying Debt

Borrowers can repay part or all of their debt.

### Adjusting LTV

Borrowers can also adjust their desired Twyne liquidation LTV.

## Position Closure

### Repaying All Debt

To close a position, borrowers must first repay all outstanding debt.

### Withdrawing All Collateral

After repaying all debt, borrowers can withdraw their remaining collateral.

## Best Practices

### Risk Management

To manage positions effectively, borrowers should:

1. **Maintain a Buffer**: Keep LTV below the liquidation threshold to account for interest accrual and market fluctuations
2. **Regular Monitoring**: Frequently check position health, especially during market volatility
3. **Periodic Rebalancing**: Rebalance positions to release excess credit and optimize capital efficiency
4. **Gradual Adjustments**: Make gradual changes to position parameters rather than drastic modifications

### Liquidation Prevention

To avoid liquidation, borrowers can:

1. **Add Collateral**: Deposit additional collateral to improve health factor
2. **Repay Debt**: Reduce borrowed amount to lower LTV
3. **Lower LTV Setting**: Reduce the desired Twyne liquidation LTV
4. **Proactive Rebalancing**: Regularly rebalance to maintain optimal position sizing

### Capital Efficiency

To maximize capital efficiency, borrowers can:

1. **Optimize LTV**: Set LTV as high as risk tolerance allows
2. **Frequent Rebalancing**: Release excess credit promptly to avoid paying unnecessary interest
3. **Parameter Adjustments**: Adjust LTV based on market conditions and risk outlook

## Related Topics

- [Credit Reservation](./01-Credit-Reservation.md): How credit is initially reserved from Intermediate Vaults
- [Interest Rates](./02-Interest-Rates.md): How interest affects positions and creates the need for management
- [Rebalancing](./03-Rebalancing.md): How excess credit is released back to Intermediate Vaults
- [Liquidation Conditions](../03-Liquidations/01-Liquidation-Conditions.md): When positions become eligible for liquidation