# Non-Negative Excess Credit

The non-negative excess credit invariant is one of the two fundamental invariants that ensure Twyne's stability and safety. This document provides a comprehensive explanation of what this invariant means, why it's critical, how it's enforced, and the mathematical principles behind it.

## What is the Non-Negative Excess Credit Invariant?

At its core, the non-negative excess credit invariant states that:

**A collateral vault must never have negative excess credit.**

In other words, a borrower should never be allowed to reserve more credit from the Intermediate Vault than what they are entitled to based on their current collateral and parameters.

This can be expressed mathematically as:

$$\text{excessCredit} = C_{total} - \frac{C \cdot liqLTV_{twyne}}{\beta_{safety} \cdot liqLTV_{ext}} \geq 0$$

Where:
- $C_{total}$ = Total collateral in the vault (C + C_LP)
- $C$ = User collateral
- $liqLTV_{twyne}$ = Twyne liquidation LTV (user-selected)
- $liqLTV_{ext}$ = External protocol liquidation LTV
- $\beta_{safety}$ = Safety buffer (typically 0.95)

## Why is This Invariant Critical?

The non-negative excess credit invariant serves several crucial purposes:

1. **Credit LP Protection**: It ensures that borrowers reserve the credit required, which protects Credit LPs from losing more than they should in external liquidation scenarios.

2. **External Liquidation Prevention**: By ensuring sufficient credit reservation, it maintains sufficient collateralization to avoid unexpected external liquidations.

3. **Protocol Stability**: It helps maintain the mathematical relationships that power Twyne's design, ensuring the entire system operates as intended.

4. **Borrower Experience**: It helps guarantee that Borrowers receive the liquidation LTV of their choosing.

If this invariant were violated, borrowers could potentially get liquidated on the underlying lending market before they are signalled as liquidatable on Twyne, this indirectly would also put Credit LPs' funds at risk and destabilizing the protocol.

## Mathematical Derivation

Let's derive the non-negative excess credit invariant from Twyne's fundamental equations:

1. For a healthy position, we need to satisfy two constraints:
   - Twyne constraint: $B \leq liqLTV_{twyne} \cdot C$
   - External constraint: $B \leq \beta_{safety} \cdot liqLTV_{ext} \cdot (C + C_{LP})$

2. For optimal credit reservation, we want these constraints to be equal at the liquidation threshold:
   $liqLTV_{twyne} \cdot C = \beta_{safety} \cdot liqLTV_{ext} \cdot (C + C_{LP})$

3. Solving for $C_{LP}$:
   $C_{LP} = \frac{liqLTV_{twyne} \cdot C}{\beta_{safety} \cdot liqLTV_{ext}} - C$

4. Therefore, the total collateral the vault should have is:
   $C + C_{LP} = \frac{liqLTV_{twyne} \cdot C}{\beta_{safety} \cdot liqLTV_{ext}}$

5. The excess credit is then:
   $\text{excessCredit} = \text{actualTotal} - \text{requiredTotal}$
   $= (C + C_{LP}) - \frac{liqLTV_{twyne} \cdot C}{\beta_{safety} \cdot liqLTV_{ext}}$

6. This excess must be non-negative, which gives us our invariant.

## Python Implementation for Verification

Here's a Python function to verify the non-negative excess credit invariant:

```python
def has_non_negative_excess_credit(total_assets, user_collateral, twyne_liq_ltv,
                                 ext_lending_ltv, safety_buffer=0.95):
    """
    Check if a position has non-negative excess credit

    Parameters:
    -----------
    total_assets : float
        Total assets in the vault (C + C_LP)
    user_collateral : float
        User's collateral (C)
    twyne_liq_ltv : float
        Twyne liquidation LTV (e.g., 0.85 for 85%)
    ext_lending_ltv : float
        External lending protocol liquidation LTV (e.g., 0.75 for 75%)
    safety_buffer : float
        Safety buffer to avoid external liquidation (default 0.95)

    Returns:
    --------
    dict
        Verification results including excess credit amount
    """
    # Calculate maximum allowed total assets
    calculated_max_collateral = user_collateral * twyne_liq_ltv / (safety_buffer * ext_lending_ltv)

    # Calculate excess credit
    excess_credit = total_assets - calculated_max_collateral

    # Check if excess credit is non-negative
    is_valid = excess_credit => 0

    return {
        "is_valid": is_valid,
        "excess_credit": excess_credit,
        "max_allowed_total": calculated_max_collateral,
        "actual_total": total_assets
    }
```

## Example Scenarios

Let's examine several scenarios to understand how this invariant works in practice:

### Scenario 1: Optimal Credit Reservation

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Calculate required credit using the formula
required_credit = user_collateral * ((twyne_liq_ltv / (safety_buffer * ext_lending_ltv)) - 1)
# required_credit = 1.0 * [(0.85 / (0.95 * 0.75)) - 1.0] = 0.192 ETH

total_assets = user_collateral + required_credit  # 1.0 + 0.192 = 1.192 ETH

# Verify the invariant
result = has_non_negative_excess_credit(
    total_assets, user_collateral, twyne_liq_ltv, ext_lending_ltv, safety_buffer
)
print(f"Invariant satisfied: {result['is_valid']}")
print(f"Excess credit: {result['excess_credit']} ETH")
# Invariant satisfied: True
# Excess credit: 0.0 ETH (or very close to 0 due to floating point precision)
```

In this scenario, the borrower has reserved exactly the amount of credit they're entitled to. The excess credit is 0, which satisfies the invariant.

### Scenario 2: Under-Reserved Credit

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Reserve less credit than entitled
reserved_credit = 0.1     # 0.1 ETH (less than the 0.192 ETH required)
total_assets = user_collateral + reserved_credit  # 1.0 + 0.1 = 1.1 ETH

# Verify the invariant
result = has_non_negative_excess_credit(
    total_assets, user_collateral, twyne_liq_ltv, ext_lending_ltv, safety_buffer
)
print(f"Invariant satisfied: {result['is_valid']}")
print(f"Excess credit: {result['excess_credit']} ETH")
# Invariant satisfied: False
# Excess credit:  -0.092 ETH
```

In this scenario, the borrower has reserved less credit than they should have. The excess credit is negative, violating the invariant.

### Scenario 3: Attempted Over-Reservation

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Attempt to reserve more credit than entitled
reserved_credit = 0.3     # 0.3 ETH (more than the 0.26 ETH entitled)
total_assets = user_collateral + reserved_credit  # 1.0 + 0.3 = 1.3 ETH

# Verify the invariant
result = has_non_negative_excess_credit(
    total_assets, user_collateral, twyne_liq_ltv, ext_lending_ltv, safety_buffer
)
print(f"Invariant satisfied: {result['is_valid']}")
print(f"Excess credit: {result['excess_credit']} ETH")
# Invariant satisfied: True
# Excess credit: 0.18 ETH
```

In this scenario, the borrower attempts to reserve more credit than they should have. The excess credit is positive, which violates the invariant (without being mechanically problematic). Twyne would revert this transaction.

### Scenario 4: Interest Accrual Impact

```python
# Initial parameters
user_collateral = 1.0     # 1.0 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Optimal credit reservation
required_credit = user_collateral *  ((twyne_liq_ltv / (safety_buffer * ext_lending_ltv)) - 1.0)
# required_credit = 0.192 ETH
total_assets = user_collateral + required_credit  # 1.192 ETH

# After interest accrual (reducing user collateral, increasing Credit LP share)
interest_amount = 0.05    # 0.05 ETH interest
user_collateral_after = user_collateral - interest_amount  # 0.95 ETH
reserved_credit_after = required_credit + interest_amount  # 0.242 ETH
total_assets_after = user_collateral_after + reserved_credit_after  # 1.192 ETH (unchanged)

# Calculate new required credit based on reduced user collateral
new_required_credit = user_collateral_after *  ((twyne_liq_ltv / (safety_buffer * ext_lending_ltv)) - 1.0)
# new_required_credit = 0.95 *  ((0.85 / (0.95 * 0.75)) - 1.0) = 0.1833 ETH

# Verify the invariant
result = has_non_negative_excess_credit(
    total_assets_after, user_collateral_after, twyne_liq_ltv, ext_lending_ltv, safety_buffer
)
print(f"Invariant satisfied: {result['is_valid']}")
print(f"Excess credit: {result['excess_credit']} ETH")
# Invariant satisfied: True
# Excess credit: 0.0587 ETH
```

After interest accrues, the position has accumulated excess credit. This is why Twyne needs a rebalancing mechanism to release excess credit back to the Intermediate Vault. Without it, the borrower would be paying interest on 0.0587 ETH extra which are not contributing to boosting their borrowing power. Alternatively the borrower could choose to signal for a higher Twyne liquidation LTV to take advantage of this excess credit reserved.

## Connection to Rebalancing

The non-negative excess credit invariant directly relates to the rebalancing mechanism:

1. As interest accrues, user collateral decreases and Credit LP share increases
2. This can lead to excess credit if not addressed
3. Rebalancing releases excess credit to maintain the invariant
4. The amount to release is precisely the excess credit value

## Edge Cases and Considerations

### Precision Issues

Fixed-point arithmetic in Solidity can lead to rounding errors. The implementation is structured to always round in a conservative direction, ensuring the invariant is never violated due to precision issues.

### Parameter Boundaries

The non-negative excess credit invariant assumes certain parameter boundaries:
- `twyneLiqLTV` must be greater than 0 and less than or equal to `MAX_LTV`
- `extLendingLTV` must be greater than 0
- `SAFE_LTV_BUFFER` must be less than 1.0 and greater than 0

These boundaries are enforced in the contract initialization and parameter setting functions.

## Impact on User Operations

The non-negative excess credit invariant affects several user operations:

1. **Deposits**: When depositing collateral, users may reserve additional credit, but only up to the allowed amount
2. **Withdrawals**: When withdrawing collateral, excess credit may need to be released
3. **Setting LTV**: When changing the Twyne liquidation LTV, credit reservation may need adjustment
4. **Interest Accrual**: As interest accrues, rebalancing may be needed to maintain the invariant

## Related Topics

- [Position Health Invariant](./02-Position-Health.md): The other critical invariant that works alongside non-negative excess credit
- [Credit Reservation](../02-Core-Mechanics/01-Credit-Reservation.md): How credit is reserved based on the formula derived above
- [Rebalancing](../02-Core-Mechanics/03-Rebalancing.md): How excess credit is released to maintain this invariant
- [Position Management](../02-Core-Mechanics/04-Position-Management.md): How users can optimize their positions within invariant constraints
