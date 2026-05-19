# Demystifying Ethereum Transaction Execution: A Yellow Paper Deep Dive (Shanghai Version)

Ethereum is fundamentally a transaction-based state machine. In the official [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf), **Section 6** defines the absolute core of this engine: the mathematical and logical process of **Transaction Execution**. 

This article deconstructs the rigorous mathematical formulas of Section 6 (Equations 64 through 91) into an accessible, highly detailed, and practical guide. Whether you are a beginner seeking a mental model, an advanced engineer learning EVM mechanics, or a core developer building consensus clients, this deep dive is designed for you.

---

## 1. Visualizing the Execution Pipeline

To understand the mathematics, we must first visualize the state changes. An Ethereum transaction transitions the global state from $\sigma$ to a final state $\sigma''$ through five distinct mathematical checkpoints:

```mermaid
graph TD
    subgraph State Transitions
        S0["Global State ( σ )"] -->|1. Validation & Upfront Fee| S1["Checkpoint State ( σ' )"]
        S1 -->|2. EVM Execution ( Λ or Θ )| SP["Provisional State ( σᴾ )"]
        SP -->|3. Gas Refund & Fee Distribution| SStar["Pre-Final State ( σ* )"]
        SStar -->|4. State Self-Destruct Cleanup| SDoubleStar["Final State ( σ'' )"]
    end

    subgraph EVM Substates
        AStar["Initial Substate ( A* )<br/>- Access List Warmup"] -.->|Injected into| SP
    end
```

---

## 2. Glossary & Variable Mapping
The Yellow Paper uses rigorous Greek and script notation. This table maps those formal symbols to their modern **JSON-RPC** and **Go-Ethereum (Geth)** source code equivalents:

| Yellow Paper Symbol | Concept | JSON-RPC / Transaction Field | Geth Source Variable (`core/types/transaction.go`) |
| :--- | :--- | :--- | :--- |
| $\sigma$ | World State | — | `state.StateDB` |
| $T$ | Transaction | — | `types.Transaction` |
| $S(T)$ | Sender Address | `from` (derived via ECDSA recovery) | `types.Sender(signer)` |
| $T_t$ | Recipient Address | `to` (if contract creation, $T_t = \emptyset$) | `tx.To()` |
| $T_n$ | Transaction Nonce | `nonce` | `tx.Nonce()` |
| $T_g$ | Gas Limit | `gas` | `tx.Gas()` |
| $T_v$ | Value (in wei) | `value` | `tx.Value()` |
| $T_i$ | Input Data / Initcode | `input` or `data` | `tx.Data()` |
| $T_p$ | Legacy Gas Price | `gasPrice` (Type 0 / 1) | `tx.GasPrice()` |
| $T_m$ | Max Fee per Gas | `maxFeePerGas` (Type 2) | `tx.GasFeeCap()` |
| $T_f$ | Max Priority Fee per Gas | `maxPriorityFeePerGas` (Type 2) | `tx.GasTipCap()` |
| $T_A$ | Access List | `accessList` (Type 1 / 2) | `tx.AccessList()` |
| $H_f$ | Block Base Fee | — (Block Header Base Fee) | `header.BaseFee` |
| $g_0$ | Intrinsic Gas | — | Calculated via `IntrinsicGas()` |
| $v_0$ | Upfront Cost | — | Upfront balance check in `preCheck()` |
| $p$ | Effective Gas Price | — | `vm.EVM.GasPrice` |
| $f$ | Validator Priority Fee | — | Derived tip credited to fee recipient |

---

## 3. High-Level Perspectives

### The Beginner's Intuition
Think of executing a transaction like renting a self-driving taxi:
1. **The Booking (Pre-execution validation):** The taxi check your wallet to ensure you have enough money to cover the longest possible trip (`gasLimit` × `maxFee`). You are billed this maximum amount upfront.
2. **The Ride (EVM Execution):** The taxi drives. It consumes fuel (`gas`) as it makes turns and climbs hills (executing opcodes). If it creates a new destination (contract creation), it incurs a surcharge.
3. **The Refund (Finalization):** You arrive at your destination early. The taxi calculates the actual fuel used, refunds you the difference for the unused gas, and gives you a bonus discount if you cleaned up trash in the car (clearing storage slots/self-destructs).
4. **The Payout:** The base rate of the fuel is burned (destroyed forever to combat inflation), and the driver gets a tip.

### The Advanced Engineer's Architecture
For contract developers and protocols, Section 6 establishes absolute invariants:
* **Irrevocability:** Nonce increment and upfront fee deductions are *immediate and irrevocable*. Once validation passes, these state changes occur even if the actual EVM execution reverts due to an `OOG` (Out of Gas) or `REVERT` opcode.
* **Deterministic Execution:** There are no "invalid" transactions during runtime. Errors inside the EVM are safely captured, returning a status code $z \in \{0, 1\}$. State mutations inside the execution are rolled back to the checkpoint state $\sigma'$, but the fees are still paid.
* **Economic Defense:** The entire validation system is designed to prevent denial-of-service (DoS) attacks on nodes by ensuring that every computational and storage resource is paid for *before* a node commits CPU cycles to run it.

---

## 4. Phase-by-Phase Mathematical Deep Dive

### Phase 1: Pre-Execution Validation & Upfront Charges

Before the EVM executes a single opcode, the transaction must pass a gauntlet of structural and economic checks.

#### 1. Intrinsic Gas ($g_0$) — Equation (64) & (65)
Intrinsic gas is the absolute minimum gas required to submit a transaction. It represents the cost of payload storage, transaction overhead, contract initialization, and access-list warming:

$$g_0 \equiv \sum_{i \in T_i, T_d} \begin{cases} G_{txdatazero} & \text{if } i = 0 \\ G_{txdatanonzero} & \text{otherwise} \end{cases} + \begin{cases} G_{txcreate} + R(\|T_i\|) & \text{if } T_t = \emptyset \\ 0 & \text{otherwise} \end{cases} + G_{transaction} + \sum_{j=0}^{\|T_A\|-1} \left(G_{accesslistaddress} + \|T_A[j]_s\| \times G_{accessliststorage}\right)$$

* **Calldata Costs:** Each zero byte of transaction data ($T_i$ or $T_d$) costs **4 gas** ($G_{txdatazero}$). Each non-zero byte costs **16 gas** ($G_{txdatanonzero}$).
* **Base Fee:** $G_{transaction}$ is the static cost of processing any transaction, set at **21,000 gas**.
* **Contract Creation Surcharges ($T_t = \emptyset$):**
  * $G_{txcreate}$ is a flat fee of **32,000 gas**.
  * **Initcode Word Cost $R(x)$ — Equation (65):**
    $$R(x) \equiv G_{initcodeword} \times \left\lceil \frac{x}{32} \right\rceil$$
    Introduced in **EIP-3860** (Shanghai Upgrade), this charges $G_{initcodeword} = 2$ gas per 32-byte word (rounded up) of the contract initialization code. This prevents DoS attacks using massive, un-executed initcode blobs.
* **EIP-2929 Access Lists Warmup:** If the transaction includes an EIP-2930 access list ($T_A$), it warms up addresses and storage keys upfront. Each warmed address costs $G_{accesslistaddress} = 2,400$ gas, and each warmed storage slot key ($T_A[j]_s$) costs $G_{accessliststorage} = 1,900$ gas.

---

#### 2. Pricing Mechanics (EIP-1559) — Equations (66) & (67)
The effective gas price $p$ (measured in wei per unit of gas) determines the actual billing rate for the transaction signer. It depends on whether the transaction is Legacy/Access List ($T_x = 0 \text{ or } 1$) or EIP-1559 Type 2 ($T_x = 2$):

$$\text{Effective Gas Price: } p \equiv \begin{cases} T_p & \text{if } T_x = 0 \lor T_x = 1 \\ f + H_f & \text{if } T_x = 2 \end{cases}$$

Where the priority fee $f$ (tip) rewarded to the validator is calculated as:

$$\text{Priority Fee: } f \equiv \begin{cases} T_p - H_f & \text{if } T_x = 0 \lor T_x = 1 \\ \min\left(T_f, T_m - H_f\right) & \text{if } T_x = 2 \end{cases}$$

* **For Type 2 Transactions:** The sender sets a max fee cap ($T_m$) and a tip cap ($T_f$). The protocol charges the absolute minimum required: the block's base fee ($H_f$) plus the tip cap ($T_f$), capped strictly by the max fee ($T_m$). 

---

#### 3. Upfront Cost ($v_0$) and Sanity Checks — Equations (68) & (69)
The upfront cost $v_0$ is the total capital that must be immediately available in the sender's account:

$$v_0 \equiv \begin{cases} T_g T_p + T_v & \text{if } T_x = 0 \lor T_x = 1 \\ T_g T_m + T_v & \text{if } T_x = 2 \end{cases}$$

This represents the maximum possible gas fee ($T_g \times T_p$ or $T_g \times T_m$) plus the value of native ether ($T_v$) to be transferred. 

```mermaid
flowchart TD
    Start([Check Transaction Validity]) --> CheckSender{Sender exist & is EOA?<br/>σ[S(T)]c == KEC(∅)}
    CheckSender -- No --> Reject([Reject Tx])
    CheckSender -- Yes --> CheckNonce{Nonce match?<br/>σ[S(T)]n == Tn}
    CheckNonce -- No --> Reject
    CheckNonce -- Yes --> CheckGas{Gas Limit >= Intrinsic Gas?<br/>Tg >= g0}
    CheckGas -- No --> Reject
    CheckGas -- Yes --> CheckBalance{Balance >= Upfront Cost?<br/>σ[S(T)]b >= v0}
    CheckBalance -- No --> Reject
    CheckBalance -- Yes --> CheckInitcode{Initcode length <= 49152?<br/>n <= 49152}
    CheckInitcode -- No --> Reject
    CheckInitcode -- Yes --> CheckBlockGas{Tg + Block prior gas <= Block gas limit?<br/>Tg + ℓ(BR)u <= BHl}
    CheckBlockGas -- No --> Reject
    CheckBlockGas -- Yes --> Accept([Accept & Begin Execution])
```

* **The EOA Check ($\sigma[S(T)]_c = \text{KEC}(\emptyset)$):** An EOA is defined by having **no smart contract code**. If the sender account has code deployed (its code hash is not the hash of empty bytes), execution is immediately aborted. *Only EOAs can initiate transactions on Ethereum.*
* **Initcode Limit Check ($n \le 49,152$ bytes):** Equation (71) defines $n$ as the length of the initcode for contract creations ($T_t = \emptyset$). EIP-3860 enforces that this length must not exceed **49,152 bytes** (exactly $1.5 \times$ the maximum contract code size of 24,576 bytes defined by EIP-170).

---

### Phase 2: Checkpoint State & EVM Computation

Once validity is established, the node creates the first checkpoint state $\sigma'$ to lock in fees and nonces.

#### 1. The Checkpoint State ($\sigma'$) — Equations (73) - (75)
The world state is modified by incrementing the sender's nonce by 1 and deducting the maximum upfront gas fee:

$$\sigma'[S(T)]_b \equiv \sigma[S(T)]_b - T_g p$$
$$\sigma'[S(T)]_n \equiv \sigma[S(T)]_n + 1$$

> [!NOTE]
> Even if the transaction fails due to an out-of-gas error 1 millisecond later, these changes are fully written to the state database. This guarantees that validators are compensated for checking and broadcasting transactions, and prevents transaction replay attacks.

#### 2. EVM Instantiation & The Warmed-up Access List — Equations (76) - (81)
The EVM now starts. The execution depends on whether the destination is empty ($T_t = \emptyset$, denoting contract creation) or a message-call. We define the tuple of the provisional state $\sigma^P$, remaining gas $g'$, accrued substate $A$, and status code $z$:

$$(\sigma^P, g', A, z) \equiv \begin{cases} \Lambda_4\left(\sigma', A^*, S(T), S(T), g, p, T_v, T_i, 0, \emptyset, \text{true}\right) & \text{if } T_t = \emptyset \\ \Theta_4\left(\sigma', A^*, S(T), S(T), T_t, T_t, g, p, T_v, T_v, T_d, 0, \text{true}\right) & \text{otherwise} \end{cases}$$

* **Initial Remaining Gas ($g$):** $g \equiv T_g - g_0$ (Equation 81). The EVM starts with the transaction's gas limit minus the upfront intrinsic cost.
* **Warm Access List Initialization ($A^*$) — Equation (78):**
  $$A^*_K \equiv \bigcup_{E \in T_A} \left\{ \forall i < \|E_s\|, i \in \mathbb{N} : (E_a, E_s[i]) \right\}$$
  $$A^*_a \equiv \begin{cases} a \cup T_t & \text{if } T_t \neq \emptyset \\ a & \text{otherwise} \end{cases}$$
  Where $a \equiv A_0 \cup S(T) \cup H_c \cup \bigcup_{E \in T_A} \{E_a\}$. 
  
  The substate $A^*$ is populated prior to execution to track "warmed" accounts and storage slots. By default, the sender $S(T)$, the block proposer $H_c$ (beneficiary), and the recipient/created contract are marked warm. Furthermore, any address and storage key pre-specified in the transaction's access list ($T_A$) are added. Opcodes like `SLOAD` or `CALL` targeting these accounts/keys will enjoy cheaper "warm" gas costs (100 gas instead of 2,100 gas for accounts, and 100 gas instead of 2,100 gas for storage slots).

---

### Phase 3: Gas Refunds & Payouts

After the EVM halts, we must compute gas refunds, distribute the fees to the validator, and burn the base fee.

#### 1. Gas Refund Cap ($g^*$) — Equation (82)
Gas refunds are granted to developers who free up state capacity (e.g., clearing storage slots by setting them to zero). However, to prevent users from abusing refunds to construct gas-arbitrage tokens, **EIP-3529 (London Fork)** reduced the maximum refund limit from $1/2$ to $1/5$ (or 20%) of the total gas used:

$$g^* \equiv g' + \min\left( \left\lfloor \frac{T_g - g'}{5} \right\rfloor, A_r \right)$$

* $g'$ is the gas left over at the end of the EVM execution.
* $A_r$ is the refund counter accumulated during EVM execution (e.g., clearing a storage slot refunds 4,800 gas to $A_r$).
* The maximum refund is capped at exactly one-fifth ($20\%$) of the actual gas consumed ($T_g - g'$).

---

#### 2. The Pre-Final State ($\sigma^*$) — Equations (83) - (85)
We transition from the provisional state $\sigma^P$ to the pre-final state $\sigma^*$ by refunding the sender and paying the validator:

$$\sigma^*[S(T)]_b \equiv \sigma^P[S(T)]_b + g^* p$$
$$\sigma^*[BH_c]_b \equiv \sigma^P[BH_c]_b + (T_g - g^*) f$$

```
   Total Upfront Deducted: Tg × p
   ┌───────────────────────────────────────────────┐
   │                                               │
   ▼                                               ▼
Returned to Sender (g* × p)          Consumed Fee: (Tg - g*) × p
                                     ┌─────────────────────────────┐
                                     │                             │
                                     ▼                             ▼
                            Validator Tip (f)           Base Fee Burned (Hf)
                            (Tg - g*) × f               (Tg - g*) × Hf
```

* **Sender Refund:** The sender is credited back the value of all refunded gas $g^*$ calculated at the effective gas price $p$.
* **Validator Payment:** The block beneficiary $BH_c$ (validator address) receives the total gas consumed ($T_g - g^*$) multiplied by the priority fee $f$.
* **The Burn Mechanics:** Notice that the sender paid $(T_g - g^*) \times p$ for the consumed gas. The validator only receives $(T_g - g^*) \times f$. Because EIP-1559 defines the effective gas price as $p = f + H_f$, the remaining amount of $(T_g - g^*) \times H_f$ is debited from the sender's account but is *credited to no one*. It is permanently destroyed (burned) from the world state!

---

### Phase 4: State Cleanup & Final State

The final step is to purge empty or dead accounts from the state database.

#### 1. Self-Destruct & Empty Accounts Cleanup — Equations (86) - (88)
The final state $\sigma''$ is reached by removing accounts slated for deletion:

$$\sigma'' \equiv \sigma^* \text{ except:}$$
$$\forall i \in A_s : \sigma''[i] = \emptyset$$
$$\forall i \in A_t : \sigma''[i] = \emptyset \quad \text{if } \text{DEAD}(\sigma^*, i)$$

* **Self-Destructs ($A_s$):** Any account added to the self-destruct set $A_s$ during execution is deleted from the state database.
* **Empty Accounts ($A_t$):** Any "touched" account in $A_t$ that is defined as `DEAD` (having a balance of 0, a nonce of 0, and no contract code) is cleaned up to prevent state bloat.

#### 2. Emitted Metrics — Equations (89) - (91)
Finally, the transaction emits three core variables used by the client to build receipts:
$$\text{Total Gas Used: } \Upsilon_g \equiv T_g - g^*$$
$$\text{Transaction Logs: } \Upsilon_l \equiv A_l$$
$$\text{Status Code: } \Upsilon_z \equiv z$$

---

## 5. Core Developer's Code Blueprint (Go-Ethereum)

For core developers looking to map Section 6 mathematical equations to active client code, the primary pipeline lives within Geth's **`core/state_transition.go`** file under the `TransitionDb` function.

### Code-to-Equation Mapping

#### 1. Validation & Upfront Cost (Eq 68 - 72)
Geth validates the transaction in `preCheck()` and `Commit()`:
```go
// core/state_transition.go
func (st *StateTransition) preCheck() error {
    // Equation 69: Nonce verification
    if st.msg.Nonce() != st.state.GetNonce(st.msg.From()) {
        return fmt.Errorf("%w: address %v, tx: %d, state: %d", ErrNonceTooHigh, ...)
    }
    // Equation 68: Upfront fee calculation & verification
    // Geth checks: balance >= gasLimit * gasFeeCap + value
    upfront := new(big.Int).Mul(new(big.Int).SetUint64(st.msg.Gas()), st.msg.GasFeeCap())
    upfront.Add(upfront, st.msg.Value())
    if st.state.GetBalance(st.msg.From()).Cmp(upfront) < 0 {
        return fmt.Errorf("%w: address %v have %v want %v", ErrInsufficientFunds, ...)
    }
    return nil
}
```

#### 2. Checkpoint State Creation (Eq 73 - 75)
Fee deduction and nonce increments occur in the execution setup:
```go
// core/state_transition.go
func (st *StateTransition) TransitionDb() (*ExecutionResult, error) {
    if err := st.preCheck(); err != nil {
        return nil, err
    }
    // Equation 74 & 75: Deduct upfront fee and increment nonce
    st.state.BuyGas(st.msg.From(), st.msg.Gas(), st.gasPrice)
    st.state.SetNonce(st.msg.From(), st.state.GetNonce(st.msg.From())+1)
    
    // Warm up accounts & access lists (Equation 78)
    st.state.Prepare(st.rules, st.msg.From(), st.coinbase, st.msg.To(), vm.ActivePrecompiles(st.rules), st.msg.AccessList())
    ...
}
```

#### 3. EVM Computation & Intrinsic Gas (Eq 64 - 65, Eq 76)
```go
// core/state_transition.go
// Equation 64: Verify gasLimit is greater than intrinsic gas (g0)
intrGas, err := IntrinsicGas(st.msg.Data(), st.msg.AccessList(), st.msg.To() == nil, true, st.rules.IsShanghai)
if st.msg.Gas() < intrGas {
    return nil, fmt.Errorf("%w: have %d, want %d", ErrIntrinsicGas, st.msg.Gas(), intrGas)
}

// Equation 76: Launch EVM (Create if To == nil, Call otherwise)
var (
    evm  = st.evm
    vmenv = evm.NewEVM(...)
)
if st.msg.To() == nil {
    ret, _, st.gas, err = vmenv.Create(sender, st.msg.Data(), st.gas, st.msg.Value())
} else {
    ret, st.gas, err = vmenv.Call(sender, *st.msg.To(), st.msg.Data(), st.gas, st.msg.Value())
}
```

#### 4. Refunds & Finalization (Eq 82 - 85)
```go
// core/state_transition.go
func (st *StateTransition) refundGas() {
    // Equation 82: Calculate refund cap (1/5th or 20% since EIP-3529)
    refund := st.gasUsed() / 5
    if refund > st.state.GetRefund() {
        refund = st.state.GetRefund()
    }
    st.gas += refund

    // Equation 84: Refund unused gas to sender
    st.state.RefundGas(st.msg.From(), st.gas, st.gasPrice)
    
    // Equation 85: Pay fee recipient (coinbase) the priority fee (f)
    // effectiveTip = gasPrice - baseFee
    st.state.AddBalance(st.coinbase, new(big.Int).Mul(new(big.Int).SetUint64(st.gasUsed()), st.effectiveTip()))
}
```

---

## 6. Key Takeaways for Core Engineers

1. **The Role of Precompiles ($\pi$):** Precompiled contracts (Equation 63's $\pi$ set) bypass the EVM interpreter completely. Instead of executing bytecode, they run compiled Go/Rust code inside the client itself. This drastically lowers gas consumption for heavy mathematical operations like elliptic curve pairings or SHA256 hashes.
2. **The 20% Refund Cap Safeguard:** EIP-3529 was a turning point for Ethereum state sizing. Before it, GasTokens allowed users to mint gas at low congestion and burn them to receive a 50% refund during peak hours. The reduction to a maximum 20% refund killed this practice, aligning gas dynamics strictly with real execution costs and discouraging speculative state bloat.
3. **EIP-3860's Defense-in-Depth:** Charging 2 gas per word of initcode (Equation 65) closed an elegant exploit vector where attackers could pass extremely long data payloads to contract creations to exhaust node memory without paying for the computational overhead. Enforcing $n \le 49,152$ bytes prevents any client from stack-overflowing or timing out due to excessively long contract deployments.