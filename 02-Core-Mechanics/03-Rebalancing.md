# Rebalancing

Rebalancing is a core process in Twyne that ensures positions remain optimally sized as interest accrues over time. This document explains the purpose, mechanism, and implementation of rebalancing in Twyne.

## Purpose of Rebalancing

As interest accrues in Twyne, two key changes occur:

1. Borrower collateral decreases (due to the siphoning rate paid to Credit LPs)
2. Credit LP shares increase (due to receiving interest)

This creates a situation where a Collateral Vault has reserved more credit from the Intermediate Vault than it currently needs based on the borrower's reduced collateral. This excess credit is inefficient because:

1. It's reserved but not being used optimally
2. It could be made available to other borrowers
3. It costs the borrower interest without providing additional borrowing power

Rebalancing is the process of calculating and releasing this excess credit back to the Intermediate Vault, ensuring capital efficiency and proper position sizing.

## Excess Credit Calculation

### Mathematical Model

The excess credit is calculated as:

$$\text{excessCredit} = C_{total} - \frac{C \cdot liqLTV_{twyne}}{\beta_{safety} \cdot liqLTV_{ext}}$$

Where:
- $C_{total}$ = Total collateral in the vault (C + C_LP)
- $C$ = User's collateral
- $liqLTV_{twyne}$ = Twyne liquidation LTV
- $liqLTV_{ext}$ = External protocol liquidation LTV
- $\beta_{safety}$ = Safety buffer (typically 0.95)

This formula determines how much total collateral the vault currently has versus how much it should have based on the current user collateral and selected parameters.

## Rebalancing Process

### Step-by-Step Process

1. **Calculate Excess Credit**:
   - Determine the current user collateral (total assets - reserved credit)
   - Calculate how much total collateral should be required based on current user collateral
   - Compute the difference between actual total and required total

2. **Release Excess Credit**:
   - If excess credit is positive, release it back to the Intermediate Vault
   - Update internal accounting to reflect the new reserved credit amount
   - Emit events to track the rebalancing operation

3. **Verify Invariants**:
   - Ensure the position remains healthy after rebalancing
   - Confirm non-negative excess credit invariant is maintained

## Rebalancing Example

Let's walk through a concrete example of how rebalancing works in practice:

### Initial State

- Borrower has 10 ETH collateral
- Twyne liquidation LTV: 85%
- External lending protocol LTV: 75%
- Safety buffer: 95%
- Required credit initially: 10 ETH * [(0.85/(0.95*0.75)-1] = 1.92 ETH
- Total assets in collateral vault: 11.92 ETH

### After Interest Accrual

After a period of interest accrual:
- Borrower collateral reduced to 9.5 ETH due to interest payments
- Reserved credit increased to: 1.92 ETH + 0.5 ETH = 2.42 ETH
- Total assets in collateral vault: 11.92 ETH

Now we calculate the excess credit:
1. User collateral = 9.5 ETH
2. Required total = 9.5 * 0.85 / (0.95 * 0.75) = 11.333 ETH
3. Actual total = 11.92 ETH
4. Excess credit = 11.920 - 11.333 = 0.586 ETH

There's excess credit to release for a total of 0.586 ETH. The borrower is incentivized to release it to reduce their interest payments to the Credit Vault.

## When Rebalancing Occurs

Rebalancing in Twyne can happen in several ways:

1. **Automatic Rebalancing**:
   - When certain collateral-modifying operations are performed
   - Examples include deposits, withdrawals, or Twyne liquidation LTV parameter changes.

2. **Manual Rebalancing**:
   - Users can explicitly call the `rebalance()` function
   - This might be done periodically to optimize positions

3. **Forced Rebalancing**:
   - During liquidation processes
   - When external factors require position adjustments

## Precision Considerations

Implementing rebalancing requires careful attention to precision issues:

1. **Fixed-Point Arithmetic**: All calculations use fixed-point arithmetic (typically with 18 decimal places) to handle fractional values accurately

2. **Rounding Direction**: Calculations intentionally round in conservative directions:
   - When calculating excess credit, round down to avoid releasing too much
   - When calculating required credit, round up to ensure positions remain safe

3. **Minimum Thresholds**: Very small amounts of excess credit might not be worth the gas cost to release

## Security Implications

Rebalancing has several security implications:

1. **Manipulation Resistance**: The rebalancing mechanism must be resistant to exploitation through rapid changes in parameters or positions

2. **Gas Optimization**: Performing frequent small rebalances could lead to unnecessary gas costs

3. **Safety During Market Volatility**: Rebalancing must remain safe even during periods of high market volatility

4. **Position Health**: Rebalancing should never compromise position health or violate protocol invariants

## Related Topics

- [Credit Reservation](./01-Credit-Reservation.md): How credit is initially reserved from Intermediate Vaults
- [Interest Rates](./02-Interest-Rates.md): How interest affects positions and creates the need for rebalancing
- [Non-Negative Excess Credit](../04-Protocol-Invariants/01-Non-Negative-Excess.md): The critical invariant related to rebalancing
- [Position Management](./04-Position-Management.md): How users can incorporate rebalancing into their position management strategies
