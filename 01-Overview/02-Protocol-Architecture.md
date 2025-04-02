# Protocol Architecture

Twyne's architecture consists of two primary vault types - Intermediate Vaults and Collateral Vaults - that work together to enable credit delegation while maintaining security.

## System Components

### Intermediate Vaults

- **Purpose**: Hold assets deposited by Credit LPs and lend borrowing power to Collateral Vaults
- **Deployment**: Deployed by Twyne (not by users)
- **Implementation**: Use unmodified EVault implementations 
- **Characteristics**:
  - Accept deposits from Credit LPs
  - Manage the delegation of borrowing power
  - Track interest accrued by Credit LPs
  - One Intermediate Vault per supported collateral type

### Collateral Vaults

- **Purpose**: Hold borrower collateral assets, reserve credit from Intermediate Vaults, and borrow from external lending protocols
- **Deployment**: Deployed by individual borrowers through the Twyne factory
- **Characteristics**:
  - Store user-deposited collateral
  - Reserve credit (borrowing power) from Intermediate Vaults
  - Maintain connection to the external lending protocol
  - Handle interest payments and rebalancing
  - Manage liquidation processes
  - User-specific parameters (like liquidation LTV)

### External Lending Protocols

- **Role**: The source of liquidity and the ultimate lender
- **Integration**: Twyne integrates with established lending protocols without requiring any changes to them
- **Relationship**: Both Collateral Vaults and Intermediate Vaults interact with the external lending protocol

## Data Flow Overview
```
┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                            External Lending Protocol                                                     │
│                                                                    (Euler)                                                               │
└────────────────────────────────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────┘
    ▲             │                                  │                                                                 │                 ▲
    │             │                                  │ Borrows                                                         │                 │
    │Deposits     │                                  │                                                                 │                 │ Deposits
    │Underlying   │                                  ▼                                                                 │                 │ Underlying
    │             │                  ┌─────────────────────────────┐               ┌─────────────────────────────┐     │                 │
    │             │                  │                             │    Reserves   │                             │     │                 │ 
    │             │                  │     Collateral Vault        │◄──────────────┤    Intermediate Vault       │     │ Receives        │
    │             │Receives          │    (Deployed by Borrower)   │               │     (Deployed by Twyne)     │     │ Receipt         │
    │             │Receipt           │                             │               │                             │     │ Token           │
    │             │Token             └─────────────────────────────┘               └─────────────────────────────┘     │                 │
    │             │                                ▲                                     ▲                             │                 │
    │             │           Deposits Collateral  │                                     │ Deposits Credit Assets      │                 │ 
    │             │         _______________________│                                     │________________________     │                 │
    │             │        │                                                                                      │    │                 │
    │             ▼        │                                                                                      │    ▼                 │
┌─────────────────────────────┐                                                                                ┌─────────────────────────────┐
│                             │                                                                                │                             │
│          Borrower           │                                                                                │        Credit LP            │
│                             │                                                                                │                             │
└─────────────────────────────┘                                                                                └─────────────────────────────┘
```
## Interaction Flow

1. **Credit LP Deposit**:
   - Credit LPs deposit their lending protocol receipt tokens (e.g., aTokens from Aave) into an Intermediate Vault
   - They receive shares of the Intermediate Vault in return
   - Their unused borrowing power becomes available for delegation

2. **Borrower Setup**:
   - Borrowers deploy a Collateral Vault through the Twyne factory
   - They specify their desired liquidation LTV (which should be higher than the external protocol's LTV)
   - The Collateral Vault is connected to both the appropriate Intermediate Vault and the external lending protocol

3. **Credit Reservation**:
   - When a borrower deposits collateral, their Collateral Vault automatically reserves the appropriate amount of credit from the Intermediate Vault
   - This credit amount is calculated based on the borrower's collateral and desired liquidation LTV

4. **External Borrowing**:
   - With combined collateral (borrower's deposits + reserved credit), the Collateral Vault can borrow assets from the external lending protocol
   - The borrower can borrow at a higher LTV (relative to their own collateral) than would be possible directly on the external protocol

5. **Interest and Yield**:
   - The borrower pays interest on their external debt (to the external protocol)
   - The borrower also pays a siphoning rate to the Credit LPs for the use of their borrowing power
   - Both borrowers and Credit LPs continue to earn yield from the external protocol

6. **Rebalancing and Maintenance**:
   - As interest accrues, the borrower's collateral decreases and the Credit LP's share increases
   - Excess credit is released back to the Intermediate Vault through rebalancing (anyone can trigger this)
   - The protocol enforces invariants to ensure system stability

## Smart Contract Architecture

The protocol consists of the following key contract components:

1. **CollateralVaultFactory**: Creates new Collateral Vaults for borrowers
2. **IntermediateVault**: Manages Credit LP deposits and credit delegation
3. **CollateralVaultBase**: Holds borrower collateral and manages credit reservation
4. **Interest Rate Model**: Determines the rates paid by borrowers to Credit LPs

Each of these components is designed to be modular and upgradeable, allowing for future improvements and expansions.

## Next Sections

- [User Types](./03-User-Types.md): Learn more about the different user roles within Twyne.
- [Credit Reservation](../02-Core-Mechanics/01-Credit-Reservation.md): Understand the mathematical model behind credit reservation.
