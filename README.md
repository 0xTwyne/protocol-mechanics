# Twyne Protocol Documentation

The documentation is organized to provide a clear understanding of Twyne's mechanics, mathematical models, and implementation details.

## What is Twyne?

Twyne is a risk-modular credit delegation protocol built for the Ethereum Virtual Machine. It addresses a fundamental inefficiency in current DeFi lending markets: the unused borrowing power of users who deposit assets but do not borrow against them.

By enabling these depositors (Credit LPs) to earn additional yield by making their unused borrowing power available to borrowers, Twyne unlocks higher capital efficiency while maintaining the security constraints of the underlying lending markets.

## Documentation Sections

### [1. Overview](./01-Overview/01-Introduction.md)
High-level introduction to Twyne's purpose, architecture, and user types.
- [Introduction](./01-Overview/01-Introduction.md)
- [Protocol Architecture](./01-Overview/02-Protocol-Architecture.md)
- [User Types](./01-Overview/03-User-Types.md)

### [2. Core Mechanics](./02-Core-Mechanics/01-Credit-Reservation.md)
Detailed explanations of the fundamental mechanisms that power Twyne.
- [Credit Reservation](./02-Core-Mechanics/01-Credit-Reservation.md)
- [Interest Rates](./02-Core-Mechanics/02-Interest-Rates.md)
- [Rebalancing](./02-Core-Mechanics/03-Rebalancing.md)
- [Position Management](./02-Core-Mechanics/04-Position-Management.md)

### [3. Liquidations](./03-Liquidations/01-Liquidation-Conditions.md)
How liquidations work in Twyne, including both internal and external processes.
- [Liquidation Conditions](./03-Liquidations/01-Liquidation-Conditions.md)
- [Internal Liquidation](./03-Liquidations/02-Internal-Liquidation.md)
- [External Liquidation](./03-Liquidations/03-External-Liquidation.md)

### [4. Protocol Invariants](./04-Protocol-Invariants/01-Non-Negative-Excess.md)
The critical invariants that must be maintained for protocol stability.
- [Non-Negative Excess Credit](./04-Protocol-Invariants/01-Non-Negative-Excess.md)
- [Position Health](./04-Protocol-Invariants/02-Position-Health.md)

## Key Invariants

Two critical invariants govern Twyne's operation:

1. **Non-Negative Excess Credit**: A collateral vault must never have negative excess credit, ensuring borrowers don't reserve more credit than they're entitled to.

2. **Position Health**: Positions must maintain healthy LTV ratios according to both Twyne's and the external protocol's criteria (with safety buffer).

## Repository Structure

<pre> <code>
/
├── README.md                         # Main entry point and navigation
├── 01-Overview/                      # High-level overview
│   ├── 01-Introduction.md            # What Twyne is and its value proposition
│   ├── 02-Protocol-Architecture.md   # System architecture and components
│   └── 03-User-Types.md              # Credit LPs and Borrowers
├── 02-Core-Mechanics/                # Fundamental mechanisms
│   ├── 01-Credit-Reservation.md      # How credit is reserved from LPs
│   ├── 02-Interest-Rates.md          # Interest rate models and calculations
│   ├── 03-Rebalancing.md             # Excess credit rebalancing process
│   └── 04-Position-Management.md     # Creating and managing positions
├── 03-Liquidations/                  # Liquidation processes
│   ├── 01-Liquidation-Conditions.md  # When positions can be liquidated
│   ├── 02-Internal-Liquidation.md    # Internal liquidation mechanism
│   └── 03-External-Liquidation.md    # External liquidation handling
├── 04-Protocol-Invariants/           # System invariants
│   ├── 01-Non-Negative-Excess.md     # Non-negative excess credit invariant
│   └── 02-Position-Health.md         # Position health invariant
</code> </pre>
