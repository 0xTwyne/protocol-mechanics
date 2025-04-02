# External Liquidation

While Twyne is designed to trigger internal liquidations before external protocols would liquidate positions, external liquidations can still occur in certain scenarios. This document explains how Twyne detects and handles external liquidations, the recovery mechanism, and the distribution of remaining assets.

## External Liquidation Scenarios

External liquidations may occur in the following situations:

1. **Delayed Internal Liquidation**: No one performs an internal liquidation in time, and the position becomes liquidatable by the external protocol.

2. **Extreme Market Volatility**: Rapid price movements cause the position to breach external liquidation thresholds before internal liquidators can act.

3. **Safety Buffer Breach**: The position's health deteriorates quickly, moving directly from healthy to below the safety buffer threshold.

4. **Implementation Errors**: Bugs or edge cases in Twyne's implementation allow a position to become externally liquidatable.

## External Liquidation Detection

Twyne detects external liquidations by comparing the expected assets in the Collateral Vault with the actual balance.

If the actual balance is less than what Twyne is tracking, this indicates that assets have been removed through an external liquidation.

## Liquidation Locking Mechanism

Once an external liquidation is detected, the Collateral Vault enters a "locked" state, preventing normal operations.

The only operation allowed is `handleExternalLiquidation()`, which is used to recover and distribute the remaining assets.

## Recovery Process

### Step 1: External Liquidation Occurs

The external lending protocol liquidates part or all of the position, taking some collateral to repay debt.

### Step 2: Detection and Locking

Twyne detects the liquidation through the balance discrepancy and locks the Collateral Vault.

### Step 3: Handle External Liquidation

Any user can call `handleExternalLiquidation()` to initiate the recovery process.
```

## Asset Distribution Logic

After an external liquidation, Twyne needs to fairly distribute the remaining assets between the borrower, Credit LPs, and potentially the liquidation handler. The distribution follows this general logic:

```python
def distribute_remaining_assets(remaining_collateral, remaining_debt,
                              user_collateral, reserved_credit, twyne_liq_ltv):
    """
    Distribute remaining assets after external liquidation

    Parameters:
    -----------
    remaining_collateral : float
        Collateral remaining after external liquidation
    remaining_debt : float
        Debt remaining after external liquidation
    user_collateral : float
        Original user collateral before liquidation
    reserved_credit : float
        Original reserved credit before liquidation
    twyne_liq_ltv : float
        Twyne liquidation LTV

    Returns:
    --------
    dict
        Asset distribution amounts
    """
    # Calculate how much should go to borrower to maintain the debt
    # based on Twyne LTV
    if remaining_debt > 0:
        borrower_collateral = remaining_debt / twyne_liq_ltv
    else:
        borrower_collateral = 0

    # Ensure we don't assign more than available
    borrower_collateral = min(borrower_collateral, remaining_collateral)

    # Credit LP gets priority for the rest, up to their original contribution
    credit_lp_collateral = min(
        reserved_credit,
        remaining_collateral - borrower_collateral
    )

    # Any remaining goes to protocol
    remaining_to_protocol = remaining_collateral - borrower_collateral - credit_lp_collateral

    return {
        "to_borrower": borrower_collateral,
        "to_credit_lp": credit_lp_collateral,
        "to_protocol": remaining_to_protocol,
        "debt_remaining": remaining_debt
    }
```

### Distribution Principles

The distribution follows these key principles:

1. **Debt Maintenance**: Enough collateral is allocated to maintain the remaining debt at or below the Twyne LTV.

2. **Credit LP Priority**: Credit LPs get priority for the remaining collateral, up to their original contribution.

3. **Borrower Residual**: Any assets left after satisfying the above are returned to the borrower.

This approach aims to:
- Ensure the position remains serviceable if debt remains
- Prioritize returning value to Credit LPs, who had no direct control over the position
- Return any excess value to the borrower

## Example External Liquidation Scenario

Let's walk through a complete external liquidation scenario:

```python
def simulate_external_liquidation():
    """Simulate an external liquidation and Twyne's handling of it"""

    # Initial position
    initial_position = {
        "borrower": "0xBorrower",
        "user_collateral": 1.0,        # 1.0 ETH
        "reserved_credit": 0.5,        # 0.5 ETH
        "total_collateral": 1.5,       # 1.5 ETH
        "borrowed": 1.0,               # 1.0 ETH
        "twyne_liq_ltv": 0.85,         # 85%
        "ext_lending_ltv": 0.75,       # 75%
    }

    # External liquidation occurs
    # Typically, external protocols liquidate a portion of the position
    # Let's say 50% of the debt is repaid by taking collateral + liquidation penalty
    debt_repaid = initial_position["borrowed"] * 0.5  # 0.5 ETH
    liquidation_penalty = 0.1  # 10% penalty
    collateral_taken = debt_repaid * (1 + liquidation_penalty)  # 110% of debt repaid

    # After external liquidation
    remaining_debt = initial_position["borrowed"] - debt_repaid
    remaining_collateral = initial_position["total_collateral"] - collateral_taken

    print(f"External liquidation repaid {debt_repaid} ETH debt by taking {collateral_taken} ETH collateral")
    print(f"Remaining debt: {remaining_debt} ETH")
    print(f"Remaining collateral: {remaining_collateral} ETH")

    # Detect external liquidation
    externally_liquidated = remaining_collateral < initial_position["total_collateral"]
    print(f"External liquidation detected: {externally_liquidated}")

    # Handle external liquidation by distributing remaining assets
    if externally_liquidated:
        distribution = distribute_remaining_assets(
            remaining_collateral,
            remaining_debt,
            initial_position["user_collateral"],
            initial_position["reserved_credit"],
            initial_position["twyne_liq_ltv"]
        )

        print("\nAsset distribution:")
        print(f"To borrower: {distribution['to_borrower']} ETH")
        print(f"To Credit LP: {distribution['to_credit_lp']} ETH")
        print(f"To Protocol: {distribution['to_protocol']} ETH")
        print(f"Debt remaining: {distribution['debt_remaining']} ETH")

    return {
        "initial_position": initial_position,
        "collateral_taken": collateral_taken,
        "debt_repaid": debt_repaid,
        "remaining_collateral": remaining_collateral,
        "remaining_debt": remaining_debt,
        "distribution": distribution if externally_liquidated else None
    }

# Expected output:
# External liquidation repaid 0.5 ETH debt by taking 0.55 ETH collateral
# Remaining debt: 0.5 ETH
# Remaining collateral: 0.95 ETH
# External liquidation detected: True
#
# Asset distribution:
# To borrower: 0.5882 ETH
# To Credit LP: 0.3618 ETH
# Debt remaining: 0.5 ETH
```

## Mitigating External Liquidations

Twyne includes several mechanisms to minimize the occurrence of external liquidations:

1. **Safety Buffer**: The safety buffer (typically 95%) creates a margin between Twyne's liquidation threshold and the external protocol's threshold.

2. **Early Internal Liquidation**: Triggering internal liquidations when positions are still relatively healthy reduces the chance of rapid deterioration leading to external liquidation.

3. **Incentives for Internal Liquidators**: Making internal liquidations economically attractive encourages timely liquidations.

4. **Position Monitoring**: Tools to monitor position health help users take action before liquidation becomes necessary.

## Handling External Liquidation Events

When an external liquidation occurs, several events and actions are triggered:

1. **Liquidation Detection**: Any interaction with the Collateral Vault will check if an external liquidation has occurred.

2. **Event Emission**: An event is emitted when external liquidation is detected and handled.

3. **Position Locking**: The Collateral Vault is locked for normal operations until the external liquidation is handled.

4. **Recovery Process**: The `handleExternalLiquidation()` function distributes remaining assets.

5. **Vault Reset**: After distribution, the Collateral Vault state is reset to prevent further interactions with the externally liquidated position.

## Technical Implementation Details

### Balance Tracking

For external liquidation detection to work properly, Twyne must accurately track the expected balance. A variable is updated whenever collateral asset enters or leaves the collateral vault.

### Contract Interactions

The Collateral Vault interacts with several contracts during external liquidation:

1. **Asset Token Contract**: To check the actual balance and transfer remaining assets
2. **External Protocol Contract**: Indirectly, as the external protocol triggers the liquidation
3. **Intermediate Vault**: To notify of the liquidation and handle credit recovery

### Gas Optimization

External liquidation handling involves complex calculations and multiple transfers, which can be gas-intensive. Implementations should optimize for gas efficiency while maintaining correct distribution logic.

## Risk Considerations

### External Protocol Liquidation Rules

Different external protocols have different liquidation mechanisms. Twyne must account for these differences when detecting and handling external liquidations.

For example:
- Some protocols liquidate the entire position at once
- Others liquidate a portion of the position
- Some have variable liquidation penalties
- Some protocols might allow partial repayments during liquidation

### Griefing Attacks

There's a potential risk of "griefing" attacks where a malicious user intentionally triggers external liquidation to cause disruption. This risk is mitigated by ensuring the liquidation detection and handling are robust and efficient.

### Liquidation Handling Incentives

To encourage timely handling of external liquidations, Twyne could include an incentive for users who call `handleExternalLiquidation()`. This could be a small portion of the remaining assets or a fixed fee.

## Special Cases

### Complete Liquidation

If the external protocol completely liquidates the position (all debt repaid), the distribution becomes simpler:

- Credit LPs receive their pro-rata share of remaining collateral
- The borrower receives any leftovers

### Zero Remaining Debt

If no debt remains after external liquidation, there's no need to maintain collateral for debt servicing, simplifying the distribution formula.

### No Remaining Collateral

In extreme cases, an external liquidation might leave no collateral in the vault. In this case, the `handleExternalLiquidation()` function simply resets the vault state.

## Related Topics

- [Liquidation Conditions](./01-Liquidation-Conditions.md): When positions become eligible for liquidation
- [Internal Liquidation](./02-Internal-Liquidation.md): Twyne's primary liquidation mechanism
- [Position Management](../02-Core-Mechanics/04-Position-Management.md): How users can manage positions to avoid liquidation
- [Protocol Invariants](../04-Protocol-Invariants/01-Non-Negative-Excess.md): Key invariants that affect liquidation risk
