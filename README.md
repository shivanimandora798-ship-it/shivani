To understand an expense splitter from a **theoretical** perspective, we move away from code and look at the mathematical logic, game theory, and distributed systems architecture that make it work.

Here is the "Theory Version" of the Soroban Expense Splitter.

---

# Theoretical Framework: On-Chain Group Settlement

## 1. Mathematical Logic (The "Fair Share" Model)
The contract operates on a **zero-sum balance ledger**. In any group of $n$ participants, the sum of all balances must always equal zero.

### The Settlement Formula
When a participant ($P_i$) pays an amount ($A$), the contract executes a state transition based on the following logic:

1.  **Individual Share ($s$):** The cost attributed to every member.
    $$s = \frac{A}{n}$$
2.  **Credit for Payer ($B_{payer}$):** The payer is credited the portion of the expense covered for others.
    $$\Delta B_{payer} = A - s$$
3.  **Debt for Others ($B_{other}$):** Every other participant ($P_j$ where $j \neq i$) incurs a debit.
    $$\Delta B_j = -s$$

### System Invariant
At any point $t$, the sum of all balances ($B$) in the contract remains:
$$\sum_{i=1}^{n} B_i = 0$$

---

## 2. Distributed Systems Architecture
Unlike a centralized database (like Splitwise), the theory here relies on **State Consistency** across a decentralized network.

* **Immutable Ledger:** Every "Add Expense" call is a state-changing transaction. Once recorded on the Stellar ledger, the debt cannot be deleted or altered without a counter-transaction.
* **Atomic State Transitions:** The contract ensures that the "Credit" to the payer and the "Debit" to the participants happen in a single atomic operation. It is impossible for a payer to get credit without the others simultaneously receiving the debt.

---

## 3. Game Theory & Trust Models
The contract addresses the **"Free Rider Problem"** and the **"Trust Gap"** in social financial interactions.

* **Verification vs. Trust:** In a social setting, you trust your friends to pay you back. In a smart contract setting, you replace that trust with **verification**. The "Truth" is the balance stored in the contract's `DataKey::Balances`.
* **Authorization Enforcement:** Using Stellar's `require_auth` provides **Non-Repudiation**. A user cannot later claim, "I didn't agree to pay for that," because the transaction required their unique cryptographic signature to execute (if they were the payer) or the group's consensus on the participant list (if they are a debtor).

---

## 4. Economic Efficiency
The theory of Soroban contracts focuses on **Resource Pricing (Gas)**.

* **Computational Complexity:** The "Add Expense" function has a complexity of $O(n)$, where $n$ is the number of participants. As the group size grows, the "Footprint" (storage and CPU) increases linearly.
* **State Rent:** On Stellar, "Living" data costs money. The theory of this contract suggests that once a group is fully settled (all balances return to zero), the contract state should ideally be archived to reclaim "rent" or minimize the network's long-term storage burden.

---

## 5. Settlement Theory (The End State)
The goal of the contract is to reach an **Equilibrium State** where:
$$B_1, B_2, ... B_n = 0$$

Settlement occurs through a **Direct Transfer**. In a more advanced theoretical model, the contract would facilitate a payment from $P_{debtor}$ to $P_{creditor}$ using a path-payment or a direct XLM transfer, which would then update the internal balances back to zero.
