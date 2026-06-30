ATM Machine - Low Level Design. 
State design pattern was required to be incorporated.

# ATM Transaction Workflow

This repository implements a simplified ATM transaction workflow using the **State Design Pattern**.  
The core idea is that the ATM behavior changes depending on its current state, and each state decides which operations are valid at that moment.

## Project Overview

The system is organized into the following modules:

- `atm/` — contains the main ATM facade
- `state/` — defines the state machine and state-specific behavior
- `card_manager/` — handles card-type-specific validation and withdrawal logic
- `data/` — defines enums and data transfer objects
- `db/` — provides persistence-like access for transaction and state management

---

## Main Components

### 1. ATM

The `ATM` class acts as the central controller for the machine.

It:
- holds the current state of the ATM
- delegates operations such as `init()`, `cancel()`, `read_card()`, and `read_withdrawal_details()` to the active state
- updates the stored state in the database whenever the state changes

### 2. State Machine

The ATM supports the following states:

- `READY`
- `CARD_READING`
- `WITHDRAWAL_DETAILS_READING`
- `CASH_DISPENSING`
- `CARD_EJECTING`

The state transitions are managed through `StateFactory`, which returns the appropriate concrete state based on the current `ATMState`.

---

## State Flow

### READY State
- The ATM starts in `READY`.
- When `init()` is called:
  - a new transaction is created
  - the ATM transitions to `CARD_READING`

### CARD_READING State
- Accepts card details for validation.
- If the card is valid:
  - card details are stored
  - the ATM transitions to `WITHDRAWAL_DETAILS_READING`
- If the card is invalid:
  - the transaction is disapproved
  - the ATM returns to `READY`

### WITHDRAWAL_DETAILS_READING State
- Accepts withdrawal amount and transaction information.
- If the withdrawal is valid:
  - the ATM transitions to `CASH_DISPENSING`
  - the withdrawal is marked as approved
- If the withdrawal is invalid:
  - the transaction is disapproved
  - the ATM returns to `READY`

### CASH_DISPENSING State
- Executes the withdrawal operation.
- After cash is dispensed, the transaction is marked as executed.
- The ATM remains in the cash dispensing flow until the transaction completes.

### CARD_EJECTING State
- Used when a card needs to be ejected.
- After ejection, the ATM returns to `READY`.

---

## Card Manager

The `CardManagerFactory` selects the appropriate card handler based on the card type:

- `CreditCardManager`
- `DebitCardManager`

Each card manager provides:
- `validate_card()`
- `validate_withdrawal()`
- `execute_withdrawal()`

This abstraction allows the ATM to support different card types without changing the state flow.

---

## Data Model

### `CardDetails`
Stores:
- card type
- card number
- PIN
- cardholder name

### `ATMState`
Represents the current state of the ATM:
- `READY`
- `CARD_READING`
- `WITHDRAWAL_DETAILS_READING`
- `CASH_DISPENSING`
- `CARD_EJECTING`

### `TransactionStatus`
Tracks transaction outcomes:
- `APPROVED`
- `NOT_APPROVED`
- `EXECUTED`
- `CANCELLED`

---

## Database Access Layer

The `DBAccessor` acts as the persistence interface for the simulation.

It is responsible for:
- getting the current ATM state
- creating a fresh transaction
- updating ATM state
- storing card details
- storing withdrawal details
- marking transactions as approved / executed / cancelled

---

## Design Principles Used

- **State Design Pattern** for ATM behavior control
- **Factory Pattern** for creating card managers and states
- **Separation of concerns** across ATM, state, card manager, and DB layers

---

## Example Workflow

1. ATM is in `READY`
2. `init()` is called
3. ATM moves to `CARD_READING`
4. User inserts card details
5. Card is validated
6. ATM moves to `WITHDRAWAL_DETAILS_READING`
7. User enters withdrawal amount
8. Withdrawal is validated
9. ATM moves to `CASH_DISPENSING`
10. Cash is dispensed
11. ATM returns to `READY` after card ejection or completion

---

## Notes

This repository is a low-level design example and uses simplified in-memory-style persistence behavior through `DBAccessor`.  
The implementation is structured to demonstrate how state transitions can control ATM behavior in a clean and extensible way.