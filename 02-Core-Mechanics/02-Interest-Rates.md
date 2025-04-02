# Interest Rates

Twyne integrates three different interest rate mechanisms: external lending yield, external borrowing interest, and Twyne-specific credit LP rates. This document explains how these interest rates work, their mathematical models, and implications for users.

## Interest Rate Overview

Twyne manages three distinct interest rates that affect user positions:

1. **External Lending Protocol Yield**: Earned on all assets deposited in the external lending protocol
2. **External Lending Protocol Borrow Rate**: Paid on borrowed assets from the external protocol
3. **Credit LP Supply Rate**: Paid by borrowers to Credit LPs for using their borrowing power

This creates a complex but efficient yield and interest system that compensates all parties appropriately for the risks they take.

## External Protocol Interest Rates

### Lending Yield

All assets deposited in the external lending protocol (both borrower's collateral and Credit LP deposits) earn the standard yield offered by that protocol. This yield is automatically accrued to the receipt tokens (e.g., aTokens from Aave) held by the respective vaults.

### Borrow Interest

Borrowers pay the standard interest rate charged by the external lending protocol on their borrowed assets. This interest accrues on the debt over time and must be repaid when the loan is closed.

These external protocol rates are not controlled by Twyne but are instead determined by the external protocol's interest rate model.

## Twyne-Specific Interest Rate

The core interest rate unique to Twyne is the rate paid by borrowers to Credit LPs for using their borrowing power. This is called the **siphoning rate** because it "siphons" value from the borrower's collateral to the Credit LP's balance.

### Interest Rate Model

Twyne employs a curved interest rate model that depends on the utilization rate of Credit LP assets:

$$IR(u) = \frac{IR_0}{u_0} \cdot u + \left(IR_{max} - \frac{IR_0}{u_0}\right) \cdot u^\gamma$$

Where:
- $u$ = Utilization rate of Credit LP assets ($C_{LP} / C_{total}^{LP}$)
- $IR_0$ = Base interest rate at utilization threshold
- $u_0$ = Utilization threshold
- $IR_{max}$ = Maximum interest rate
- $\gamma$ = Curve exponent (greater than 1)

Subject to these constraints:
- $IR_{max} > \frac{IR_0}{u_0} > 0$
- $\gamma > 1$
- $1 > u_0 > 0$

### Rate Behavior and Rationale

This curved interest rate model:

1. Behaves approximately linearly for utilization rates below $u_0$
2. Accelerates rapidly for utilization rates above $u_0$
3. Reaches $IR_{max}$ as utilization approaches 100%

This design encourages optimal utilization by:
- Providing reasonable rates at moderate utilization levels
- Strongly disincentivizing extreme utilization that could lead to liquidity issues
- Smoothly transitioning between these states without sharp kinks

## Flow of Interest in Twyne

### Borrower Perspective

From a borrower's perspective, they experience:

1. **Positive Flow**: External lending yield earned on their collateral
2. **Negative Flow**: External borrowing interest on their debt
3. **Negative Flow**: Siphoning rate paid to Credit LPs

The net cost to borrowers is the combination of these three components.
#### Siphoning Rate
The key formula from the borrower's perspective is the **siphoning rate**:

$$r_C = \frac{C_{LP} \cdot IR(u)}{C}$$

Where:
- $r_C$ = Siphoning rate (paid to Credit LPs in units of collateral asset)
- $C_{LP}$ = Credit reserved from Intermediate Vault
- $IR(u)$ = Interest rate from the curved model
- $C$ = User collateral

The borrower might be more interested in the net rate:

$$r_C^{net} = \frac{C_{LP} \cdot IR(u)}{C - B} = \frac{r_C}{1 - \lambda_{twyne}}$$

Where:
- $B$ = Borrowed amount
- $\lambda_{twyne}$ = Twyne LTV ($B/C$)

### Credit LP Perspective

From a Credit LP's perspective, they experience:

1. **Positive Flow**: External lending yield earned on their deposits
2. **Positive Flow**: Siphoning rate received from borrowers

The net return to Credit LPs is the combination of these two components. The key formula from the Credit LP's perspective is the **net rate**:

$$r_{LP}^{net} = \frac{C_{LP} \cdot IR(u)}{C_{total}^{LP}} = u \cdot IR(u)$$

Where:
- $r_{LP}^{net}$ = Net rate earned by Credit LPs
- $C_{total}^{LP}$ = Total credit LP assets supplied
- $u$ = Utilization rate ($C_{LP} / C_{total}^{LP}$)

## Interest Accrual Implementation

### Interest Accrual and Balance Updates

Interest accrues continuously in Twyne, but is accounted for at specific moments:

1. **On Deposit/Withdrawal**: When users deposit or withdraw assets
2. **On Borrow/Repay**: When borrowers borrow or repay loans
3. **On Rebalance**: When excess credit is released back to the Intermediate Vault
4. **On Explicit Touch**: When users call functions that update interest

The accrual process:

1. Calculate interest owed since the last update
2. Reduce the borrower's collateral by the interest amount
3. Increase the Credit LP's share by the same amount
4. Update the timestamp of the last interest accrual

## Practical Examples

### Example 1: Basic Interest Calculation

Consider the following scenario:
- Credit LP deposits 10 ETH in the Intermediate Vault
- Borrower has 5 ETH collateral and reserves 2 ETH credit
- Utilization rate is 20% (2 ETH out of 10 ETH)
- Parameter values: $IR_0 = 0.05$, $u_0 = 0.8$, $IR_{max} = 0.5$, $\gamma = 2$

The interest rate would be:
$$IR(0.2) = \frac{0.05}{0.8} \cdot 0.2 + \left(0.5 - \frac{0.05}{0.8}\right) \cdot 0.2^2 = 0.0125 + 0.475 \cdot 0.04 = 0.0125 + 0.019 = 0.0315$$

So the interest rate is 3.15% APR.

The siphoning rate for the borrower would be:
$$r_C = \frac{2 \cdot 0.0315}{5} = \frac{0.063}{5} = 0.0126$$

So 1.26% of the borrower's collateral would go to Credit LPs annually.

### Example 2: Complex Scenario with Multiple Interest Flows

Now let's consider a more complete example:
- External lending yield: 2% APR
- External borrowing rate: 3% APR
- Twyne siphoning rate: 1.5% APR on collateral
- Borrower has 10 ETH collateral and 7 ETH borrowed (70% LTV)

Annual flows for the borrower:
- +0.2 ETH from external yield (2% of 10 ETH)
- -0.21 ETH from external borrowing (3% of 7 ETH)
- -0.15 ETH from siphoning rate (1.5% of 10 ETH)

Net annual cost: -0.16 ETH

That translates to an effective cost of borrowing:
$$\frac{0.16}{7} = 0.023 \text{ or } 2.3\% \text{ APR}$$

This compares favorably to the 3% APR that would be paid by borrowing directly from the external protocol, despite using a much higher LTV.

## Rebalancing and Interest

As interest accrues, the borrower's collateral decreases and the Credit LP's share increases. This creates excess credit that should be released back to the Intermediate Vault through rebalancing.

The relationship between interest and rebalancing is explained in detail in the [Rebalancing](./03-Rebalancing.md) document.

## Security Implications

The interest rate mechanism has several security implications:

1. **Position Deterioration**: Interest accrual gradually deteriorates borrower positions, potentially leading to liquidation if not managed properly
2. **Precision Issues**: Fixed-point arithmetic in interest calculations can lead to rounding errors if not carefully implemented
3. **Extreme Utilization Handling**: The model must correctly handle edge cases like 0% or 100% utilization

## Related Topics

- [Rebalancing](./03-Rebalancing.md): How excess credit from interest accrual is released
- [Position Management](./04-Position-Management.md): How users manage their positions, including interest considerations
- [Internal Liquidation](../03-Liquidations/02-Internal-Liquidation.md): What happens when interest accrual leads to unhealthy positions
