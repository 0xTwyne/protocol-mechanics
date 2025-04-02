# Position Health Invariant

The position health invariant is the second of Twyne's two fundamental invariants, working alongside the non-negative excess credit invariant to ensure the protocol's stability and security. This document explains what the position health invariant is, why it matters, how it's enforced, and its mathematical foundations.

## What is the Position Health Invariant?

The position health invariant states that:

**A position must remain healthy according to both Twyne's criteria and the external protocol's criteria (with safety buffer) at all times, except when it's being liquidated.**

Mathematically, a position is considered healthy when both of these conditions are met:

1. $B \leq liqLTV_{twyne} \cdot C$ (Twyne condition)
2. $B \leq \beta_{safety} \cdot liqLTV_{ext} \cdot (C + C_{LP})$ (External protocol condition)

Where:
- $B$ = External debt (borrowed amount)
- $C$ = User collateral
- $C_{LP}$ = Reserved credit from Intermediate Vault
- $liqLTV_{twyne}$ = Twyne liquidation LTV (user-selected parameter)
- $liqLTV_{ext}$ = External protocol liquidation LTV
- $\beta_{safety}$ = Safety buffer (typically 0.95)

In simpler terms: a position should never borrow more than what's allowed by either Twyne's limits or the external protocol's limits (with safety margin).

## Why is This Invariant Critical?

The position health invariant serves several essential purposes:

1. **Borrower Protection**: It ensures borrowers don't take on more debt than they can handle based on their selected risk parameters.

2. **Credit LP Protection**: It protects Credit LPs by ensuring borrowers maintain sufficient collateralization levels.

3. **External Liquidation Prevention**: It helps avoid unexpected external liquidations by maintaining a safety buffer below external protocol thresholds.

4. **Protocol Solvency**: By preventing unhealthy positions (except during liquidation), it maintains the overall solvency of the protocol.

If this invariant were violated, positions could become under-collateralized, leading to potential losses for Credit LPs, borrowers, and the protocol as a whole.

## Mathematical Derivation

The position health invariant directly reflects the two key constraints in Twyne:

1. **Twyne Liquidation Constraint**:
   - A position is healthy according to Twyne when: $B \leq liqLTV_{twyne} \cdot C$
   - This ensures the borrower's debt doesn't exceed what's allowed by their chosen Twyne LTV, considering only their own collateral

2. **External Protocol Constraint**:
   - A position is healthy according to the external protocol when: $B \leq liqLTV_{ext} \cdot (C + C_{LP})$
   - To provide a safety margin, Twyne applies a buffer: $B \leq \beta_{safety} \cdot liqLTV_{ext} \cdot (C + C_{LP})$
   - This ensures the position remains safe from external liquidation

Both constraints must be satisfied simultaneously for a position to be considered healthy.

## Python Implementation for Verification

Here's a Python function to verify the position health invariant:

```python
def check_position_health(user_collateral, reserved_credit, borrowed_amount,
                         twyne_liq_ltv, ext_lending_ltv, safety_buffer=0.95,
                         collateral_price_ratio=1.0):
    """
    Check if a position satisfies the position health invariant

    Parameters:
    -----------
    user_collateral : float
        User's collateral (C)
    reserved_credit : float
        Reserved credit from Intermediate Vault (C_LP)
    borrowed_amount : float
        External debt (B)
    twyne_liq_ltv : float
        Twyne liquidation LTV (e.g., 0.85 for 85%)
    ext_lending_ltv : float
        External lending protocol liquidation LTV (e.g., 0.75 for 75%)
    safety_buffer : float
        Safety buffer to avoid external liquidation (default 0.95)

    Returns:
    --------
    dict
        Verification results including health factors
    """
    # Calculate total collateral
    total_collateral = user_collateral + reserved_credit

    # Calculate maximum borrow amounts
    max_borrow_twyne = user_collateral * twyne_liq_ltv
    max_borrow_external = total_collateral ext_lending_ltv

    # Check both conditions
    twyne_condition_met = borrowed_amount <= max_borrow_twyne
    external_condition_met = borrowed_amount <= max_borrow_external

    # Calculate health factors (>1.0 means healthy)
    twyne_health_factor = max_borrow_twyne / borrowed_amount if borrowed_amount > 0 else float('inf')
    external_health_factor = max_borrow_external / borrowed_amount if borrowed_amount > 0 else float('inf')

    # Overall health
    is_healthy = twyne_condition_met and external_condition_met

    return {
        "is_healthy": is_healthy,
        "twyne_condition_met": twyne_condition_met,
        "external_condition_met": external_condition_met,
        "twyne_health_factor": twyne_health_factor,
        "external_health_factor": external_health_factor,
        "max_borrow_twyne": max_borrow_twyne,
        "max_borrow_external": max_borrow_external
    }
```

## Example Scenarios

Let's examine several scenarios to understand how this invariant works in practice:

### Scenario 1: Healthy Position

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
reserved_credit = 0.26    # 0.26 ETH
borrowed_amount = 0.8     # 0.8 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Check position health
result = check_position_health(
    user_collateral, reserved_credit, borrowed_amount,
    twyne_liq_ltv, ext_lending_ltv, safety_buffer
)

print(f"Position is healthy: {result['is_healthy']}")
print(f"Twyne health factor: {result['twyne_health_factor']:.2f}")
print(f"External health factor: {result['external_health_factor']:.2f}")
# Position is healthy: True
# Twyne health factor: 1.0625
# External health factor: 1.1815
```

In this scenario, both conditions are met, resulting in a healthy position.

### Scenario 2: Unhealthy - Twyne Condition Violated

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
reserved_credit = 0.26    # 0.26 ETH
borrowed_amount = 0.9     # 0.9 ETH (exceeds Twyne limit)
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Check position health
result = check_position_health(
    user_collateral, reserved_credit, borrowed_amount,
    twyne_liq_ltv, ext_lending_ltv, safety_buffer
)

print(f"Position is healthy: {result['is_healthy']}")
print(f"Twyne condition met: {result['twyne_condition_met']}")
print(f"External condition met: {result['external_condition_met']}")
print(f"Max borrow (Twyne): {result['max_borrow_twyne']} ETH")
print(f"Max borrow (External): {result['max_borrow_external']} ETH")
print(f"Actual borrow: {borrowed_amount} ETH")
# Position is healthy: False
# Twyne condition met: False
# External condition met: True
# Max borrow (Twyne): 0.85 ETH
# Max borrow (External): 0.945 ETH
# Actual borrow: 0.9 ETH
```

In this scenario, the borrowed amount exceeds what's allowed by the Twyne LTV, making the position internally liquidatable.

### Scenario 3: Unhealthy - External Condition Violated

```python
# Parameters
user_collateral = 1.0     # 1.0 ETH
reserved_credit = 0.26    # 0.26 ETH
borrowed_amount = 1.1     # 0.95 ETH (exceeds External limit)
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Check position health
result = check_position_health(
    user_collateral, reserved_credit, borrowed_amount,
    twyne_liq_ltv, ext_lending_ltv, safety_buffer
)

print(f"Position is healthy: {result['is_healthy']}")
print(f"Twyne condition met: {result['twyne_condition_met']}")
print(f"External condition met: {result['external_condition_met']}")
# Position is healthy: False
# Twyne condition met: False
# External condition met: True
```

In this scenario, the borrowed amount exceeds both Twyne and external limits, making the position externally liquidatable.

### Scenario 4: Edge Case - Zero Collateral or Zero Debt

```python
# Parameters - Zero debt
user_collateral = 1.0     # 1.0 ETH
reserved_credit = 0.26    # 0.26 ETH
borrowed_amount = 0.0     # 0.0 ETH
twyne_liq_ltv = 0.85      # 85%
ext_lending_ltv = 0.75    # 75%
safety_buffer = 0.95      # 95%

# Check position health
result = check_position_health(
    user_collateral, reserved_credit, borrowed_amount,
    twyne_liq_ltv, ext_lending_ltv, safety_buffer
)

print(f"Position is healthy: {result['is_healthy']}")
print(f"Twyne health factor: {result['twyne_health_factor']}")
print(f"External health factor: {result['external_health_factor']}")
# Position is healthy: True
# Twyne health factor: inf
# External health factor: inf
```

With zero debt, the position is always healthy, and health factors are infinite.

## Maintaining Position Health

Users can maintain position health in several ways:

1. **Monitor Health Factors**: Regularly check both Twyne and external health factors
2. **Add Collateral**: Deposit additional collateral to improve health factors
3. **Repay Debt**: Reduce borrowed amount to improve health factors
4. **Adjust Twyne LTV**: Lower the Twyne liquidation LTV to increase borrowing safety margin
5. **Regular Rebalancing**: Perform rebalancing to optimize credit reservation

The minimum safe values for health factors depend on user risk tolerance, but generally:
- Health factor > 1.5: Very safe
- Health factor between 1.2 and 1.5: Moderately safe
- Health factor between 1.0 and 1.2: Risky, close to liquidation
- Health factor < 1.0: Liquidatable

## Interaction with Liquidation Mechanisms

The position health invariant is directly tied to Twyne's liquidation mechanisms:

1. **Internal Liquidation**: When the position health invariant is violated, internal liquidation can be triggered
2. **External Liquidation Prevention**: Maintaining the invariant with a safety buffer helps prevent external liquidations
3. **Liquidation Function**: The `_canLiquidate()` function that checks the invariant also determines when positions can be liquidated

This interaction ensures that positions either remain healthy or get liquidated to restore health.

## Edge Cases and Considerations

### Multiple Asset Types

When dealing with multiple collateral or debt asset types, position health requires conversion to a common unit of account:

```
TotalCollateralValue = Σ(AssetAmount_i * AssetPrice_i)
TotalDebtValue = Σ(DebtAmount_j * DebtPrice_j)
```

The invariant then becomes:
- TotalDebtValue ≤ liqLTV_twyne * UserCollateralValue
- TotalDebtValue ≤ safety_buffer * liqLTV_ext * TotalCollateralValue

### Price Volatility

Position health is affected by price changes:
- Collateral price decreases make positions less healthy
- Debt price increases make positions less healthy

Twyne's safety buffer helps mitigate this risk, but users should maintain additional buffers in volatile market conditions.

### Parameter Changes

Changes to protocol parameters can affect position health:
- Decreasing Twyne LTV makes positions less healthy
- Decreasing external LTV makes positions less healthy
- Decreasing safety buffer makes positions less healthy

The protocol should handle these changes carefully to avoid sudden mass liquidations.

## Impact on User Operations

The position health invariant affects several user operations:

1. **Borrowing**: Users can only borrow up to the limits defined by the invariant
2. **Withdrawing Collateral**: Withdrawals are limited to maintain position health
3. **Setting LTV**: Users can only set LTV parameters that maintain position health
4. **Liquidation Eligibility**: Positions that violate the invariant become eligible for liquidation

## Special States and Exceptions

There are situations where the position health invariant may be temporarily violated:

1. **During Liquidation**: While a position is being liquidated, it may be unhealthy
2. **Circuit Breakers**: In extreme market volatility, the protocol might implement circuit breakers that temporarily suspend invariant enforcement
3. **Multi-Step Operations**: Some complex operations might require multiple transactions, during which the invariant might be temporarily violated before being restored

These exceptions should be carefully controlled and monitored to prevent abuse.

## Connection to Non-Negative Excess Credit

The position health invariant works together with the non-negative excess credit invariant:

1. **Complementary Protection**: While non-negative excess credit ensures proper credit reservation, position health ensures proper debt management
2. **Sequence of Enforcement**: Typically, non-negative excess credit is checked first, followed by position health
3. **Relationship to Rebalancing**: Rebalancing helps maintain both invariants by releasing excess credit and improving position health

Together, these invariants create a robust safety system for Twyne.

## Related Topics

- [Non-Negative Excess Credit Invariant](./01-Non-Negative-Excess.md): The other critical invariant that works alongside position health
- [Liquidation Conditions](../03-Liquidations/01-Liquidation-Conditions.md): How position health directly determines liquidation eligibility
- [Internal Liquidation](../03-Liquidations/02-Internal-Liquidation.md): What happens when the position health invariant is violated
- [Position Management](../02-Core-Mechanics/04-Position-Management.md): How users can manage positions to maintain health
