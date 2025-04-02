# Credit Reservation

Credit reservation is the core mechanism that enables Twyne to provide higher LTVs to borrowers while maintaining the security constraints of external lending protocols. This document explains the mathematical model, implementation details, and practical implications of credit reservation.

## Conceptual Overview

In Twyne, a borrower can access a higher loan-to-value (LTV) ratio than what external lending protocols allow by reserving additional credit from Credit LPs. This creates two different perspectives on the same position:

1. **Twyne's Perspective**: The borrower has collateral C and debt B, with a higher LTV.
2. **External Protocol's Perspective**: The position has total collateral (C + C_LP) and debt B, with a lower LTV.

Credit reservation is the process of calculating and securing the right amount of credit (C_LP) from Intermediate Vaults to make these two perspectives consistent and secure.

## Mathematical Model

### Key Variables

- **C**: User's collateral deposited in their Collateral Vault
- **C_LP**: Credit reserved from the Intermediate Vault
- **C_total = C + C_LP**: Total collateral as seen by the external protocol
- **B**: Borrowed amount from the external protocol
- **liqLTV_twyne**: Twyne's liquidation LTV (user-selected parameter)
- **liqLTV_ext**: External protocol's liquidation LTV (fixed parameter)
- **β_safety**: Safety buffer to prevent external liquidations (typically 0.95)

### The Credit Reservation Equation

The amount of credit that needs to be reserved is determined by the following equation:

$$C_{LP} = \left(\frac{liqLTV_{twyne}}{\beta_{safety} \cdot liqLTV_{ext}} - 1\right) \cdot C$$

This equation is derived from ensuring that the maximum borrow allowed by Twyne's perspective equals the maximum safe borrow allowed by the external protocol's perspective:

$$liqLTV_{twyne} \cdot C = \beta_{safety} \cdot liqLTV_{ext} \cdot (C + C_{LP})$$

## Credit Reservation in Practice

### The Reservation Process

When a borrower deposits collateral into their Collateral Vault:

1. The vault calculates the required credit using the formula above
2. It requests this amount of credit from the Intermediate Vault
3. If sufficient credit is available, it's reserved for the borrower's Collateral Vault. Otherwise, the vault reverts.
4. The combined collateral (borrower's collateral + reserved credit) is used to borrow from the external protocol

### Example Scenario

Let's walk through a concrete example:

- A borrower deposits 1 ETH into their Collateral Vault
- They set their desired Twyne liquidation LTV to 85%
- The external protocol's liquidation LTV is 75%
- The safety buffer is set to 95%

The required credit is:

$$C_{LP} = \left(\frac{0.85}{0.95 \cdot 0.75} - 1\right) \cdot 1 = \left(\frac{0.85}{0.7125} - 1\right) \cdot 1 = (1.1930 - 1) \cdot 1 = 0.1930 \text{ ETH}$$

So the Collateral Vault would reserve approximately 0.193 ETH worth of credit from the Intermediate Vault. This means:

- From Twyne's perspective, the borrower has 1 ETH collateral and can borrow up to 0.85 ETH (85% LTV)
- From the external protocol's perspective, the position has 1.193 ETH total collateral and can borrow up to 0.85 ETH (which is only about 71% LTV for the external protocol)

The difference between these two perspectives is what creates the higher borrowing capacity for the borrower.

### Constraints and Limitations

Several constraints affect credit reservation:

1. **Available Credit**: The Intermediate Vault must have sufficient unused credit to reserve
2. **Parameter Boundaries**:
   - `twyneLiqLTV` must be greater than 0 and less than or equal to `MAX_LTV`
   - `extLendingLTV` must be greater than 0
   - For the formula to work correctly, usually `twyneLiqLTV >= β_safety*extLendingLTV`
     - In practice, a value `twyneLiqLTV < extLendingLTV` (resulting in a negative reserving amount $$C_{LP}<0$$) simply means that the user is willing to forgo the borrowing power of $$|C_{LP}|$$ of their collateral. When this occurs, the Twyne frontend deposits that much of the user's collateral to the Intermediate Vault, leaving the remaining collateral in the Collateral Vault with `twyneLiqLTV=β_safety*extLendingLTV`. As such, the smart contracts always see `twyneLiqLTV >= β_safety*extLendingLTV` satisfied by the collateral vaults.

3. **Safety Buffer**: The safety buffer (`β_safety`) creates a margin of safety to prevent external liquidations

## Implications for Protocol Security

Credit reservation has significant implications for protocol security:

1. **Critical Invariant**: The Non-Negative Excess Credit invariant ensures that a Collateral Vault never reserves more credit than it's entitled to based on its current state.

2. **Position Health**: The amount of reserved credit directly affects the health of borrower positions relative to both Twyne's and the external protocol's liquidation thresholds.

3. **Liquidation Protection**: Proper credit reservation ensures that Twyne's internal liquidation mechanisms will trigger before external liquidations, protecting both borrowers and Credit LPs.

4. **Protection Against Market Volatility**: The safety buffer provides additional protection against market volatility, giving time for liquidations to occur internally before external protocols would liquidate.

## Related Topics

- [Interest Rates](./02-Interest-Rates.md): How interest affects the relationship between user collateral and reserved credit
- [Rebalancing](./03-Rebalancing.md): The process of releasing excess credit back to Intermediate Vaults
- [Non-Negative Excess Credit](../04-Protocol-Invariants/01-Non-Negative-Excess.md): The critical invariant related to credit reservation
- [Critical Functions](../05-Technical-Reference/01-Critical-Functions.md): Detailed explanations of key functions including `_hasNonNegativeExcessCredit()`
