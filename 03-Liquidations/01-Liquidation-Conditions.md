# Liquidation Conditions

Liquidations are a critical part of Twyne's risk management system. This document explains when and why positions become eligible for liquidation, the mathematical conditions that trigger liquidations, and how these conditions are implemented in code.

## Liquidation Thresholds Overview

Twyne operates with two distinct liquidation thresholds:

1. **Twyne Liquidation Threshold** - A user-selected parameter that determines when internal liquidation can occur
2. **External Protocol Liquidation Threshold** - Inherited from the external lending protocol (e.g., Euler)

The key insight of Twyne is that it triggers internal liquidations before external protocols would, allowing for a more controlled and potentially less punitive liquidation process.

## Mathematical Liquidation Conditions

A position in Twyne can be liquidated if either of these conditions is met:

### Condition 1: Twyne LTV Exceeded

```
B > liqLTV_twyne * C
```

Where:
- **B** = External debt (borrowed amount)
- **liqLTV_twyne** = Twyne liquidation LTV (user-selected parameter)
- **C** = User collateral (excluding reserved credit)

This condition checks if the borrower's debt exceeds what's allowed by their chosen Twyne LTV, considering only their own collateral (not the reserved credit).

### Condition 2: External Protocol Risk

```
B > safety_buffer * liqLTV_ext * (C + C_LP)
```

Where:
- **B** = External debt (borrowed amount)
- **safety_buffer** = Safety buffer (typically 0.95 or 95%)
- **liqLTV_ext** = External protocol liquidation LTV
- **C** = User collateral
- **C_LP** = Reserved credit from Intermediate Vault

This condition checks if the borrower's debt is approaching the level where external liquidation might occur. The safety buffer ensures Twyne acts before the external protocol would liquidate.

## Practical Example

Let's walk through an example to illustrate when liquidation would be triggered:

```python
# Example parameters
total_assets = 1.5        # 1.5 ETH (user collateral + reserved credit)
user_collateral = 1.0     # 1.0 ETH (user's own collateral)
external_debt = 0.8       # 0.8 ETH worth of borrowed assets
twyne_liq_ltv = 0.85      # 85% Twyne liquidation LTV
ext_lending_ltv = 0.75    # 75% external lending protocol LTV
safety_buffer = 0.95      # 95% safety buffer

# Calculate max borrow limits
max_borrow_twyne = user_collateral * twyne_liq_ltv
# max_borrow_twyne = 1.0 * 0.85 = 0.85 ETH

max_borrow_external = total_assets * ext_lending_ltv
# max_borrow_external = 1.5 * 0.75 = 1.2 ETH

# Check liquidation conditions
twyne_condition = external_debt > max_borrow_twyne
# twyne_condition = 0.8 > 0.85 = False

external_condition = external_debt > max_borrow_external
# external_condition = 0.8 > 1.2 = False

# Result: Position is healthy, cannot be liquidated
# If external_debt were 0.9 ETH, twyne_condition would be True
# and the position could be liquidated
```

## Monitoring Liquidation Risk

Users can monitor their liquidation risk by calculating two health factors:

1. **Twyne Health Factor** = max_borrow_twyne / current_debt
2. **External Health Factor** = max_borrow_external / current_debt

A health factor below 1.0 indicates the position is eligible for liquidation. Users should generally aim to maintain both health factors comfortably above 1.0 to avoid liquidation.

## Impact of Parameter Changes

Several parameters affect liquidation risk:

1. **Twyne Liquidation LTV (liqLTV_twyne)**:
   - Increasing this parameter allows borrowing more but increases liquidation risk
   - Decreasing this parameter reduces liquidation risk but limits borrowing capacity

2. **External Lending LTV (liqLTV_ext)**:
   - This is determined by the external protocol and can't be changed by users
   - Changes to this parameter by the external protocol affect all Twyne positions

3. **Safety Buffer**:
   - A lower safety buffer increases liquidation risk from external protocols
   - A higher safety buffer provides more protection but limits borrowing capacity

## Relationship with Other Mechanisms

Liquidation conditions interact closely with other Twyne mechanisms:

- **Credit Reservation**: The amount of credit reserved affects the external liquidation condition
- **Rebalancing**: Regular rebalancing helps maintain position health by releasing excess credit
- **Interest Accrual**: As interest accrues, positions gradually move closer to liquidation thresholds

## Related Topics

- [Internal Liquidation](./02-Internal-Liquidation.md): What happens when liquidation conditions are met
- [External Liquidation](./03-External-Liquidation.md): How external liquidations are handled
- [Position Management](../02-Core-Mechanics/04-Position-Management.md): How users can manage positions to avoid liquidation
- [Non-Negative Excess Credit](../04-Protocol-Invariants/01-Non-Negative-Excess.md): A related protocol invariant
