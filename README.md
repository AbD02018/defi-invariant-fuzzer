# 🧪 defi-invariant-fuzzer

> **Foundry-based property-testing framework for DeFi protocol invariants.**

Catch billion-dollar bugs before deployment by writing invariants that must NEVER break — and then letting the fuzzer try to break them.

---

## ✨ Features

- 🎯 **Protocol-specific invariant templates** — ERC20 conservation, lending solvency, AMM constant-product, liquidation bounds
- 🚀 **Foundry-invariant native** — Built on top of `forge invariant`, no new DSL to learn
- 🔍 **Automatic call sequence generation** — Multi-step state transitions (deposit → borrow → liquidate)
- 📊 **Failure analysis** — Renders counterexamples in human-readable form with state diff
- 🐳 **Docker-ready** — One-liner for CI/CD
- 📚 **11 templates** for common DeFi protocol types

---

## 📦 Installation

```bash
forge install AbD02018/defi-invariant-fuzzer --no-commit
cp -r lib/defi-invariant-fuzzer/templates/* test/invariants/
```

### Requirements

- Foundry (`forge`, `cast`, `anvil`)
- Solidity 0.8.x

---

## 🚀 Usage

### Initialize a new invariant suite

```bash
forge invariant-init --target MyLending --suite LendingInvariants
```

This generates `test/invariants/LendingInvariants.sol` with pre-configured handlers for common lending protocol invariants.

### Run invariants

```bash
forge invariant --match-contract LendingInvariants --depth 50
```

### Available templates

| Template | Use For |
|---|---|
| `ERC20Conservation` | Token mint/burn balance checks |
| `LendingSolvency` | Collateral >= Debt at all times |
| `LendingLiquidation` | Liquidation maintains protocol solvency |
| `AMMConstantProduct` | `x * y = k` for AMM pools |
| `StakingRewards` | Rewards owed <= Rewards balance |
| `VaultShareConservation` | Sum of shares == totalSupply |
| `OracleNoStaleness` | Prices updated within N seconds |
| `BridgeMessageUniqueness` | No double-spend in cross-chain bridges |
| `GovernanceQuorum` | Vote counts don't exceed total supply |
| `OrderbookConservation` | Bid+Ask fills cancel out correctly |
| `LendingAccountHealth` | Health factor monotonicity |

---

## 📖 Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "defi-invariant-fuzzer/templates/LendingSolvency.sol";
import "../src/MyLending.sol";

contract MyLendingInvariants is LendingSolvency {
    MyLending internal lending;
    
    function setUp() public {
        lending = new MyLending();
        _registerTarget(address(lending));
    }
    
    /// @custom:invariant Total debt must never exceed total collateral value
    function invariant_solvency() public {
        assertTrue(_isSolvent(lending), "Protocol insolvent");
    }
}
```

Run with:
```bash
forge invariant --match-contract MyLendingInvariants --depth 100
```

When the fuzzer finds a counterexample, the framework outputs:
1. **Call sequence** that broke the invariant
2. **State diff** showing which variables changed
3. **Trace** of internal calls

---

## 🔬 Architecture

```
defi-invariant-fuzzer/
├── src/
│   ├── base/
│   │   └── BaseInvariant.sol       # Abstract invariant contract
│   ├── handlers/
│   │   ├── ERC20Handler.sol        # Random mint/transfer/burn
│   │   ├── LendingHandler.sol      # Deposit/borrow/repay/liquidate
│   │   ├── AMMHandler.sol          # Add/remove liquidity, swap
│   │   └── OracleHandler.sol       # Push prices
│   └── templates/
│       ├── ERC20Conservation.sol
│       ├── LendingSolvency.sol
│       ├── AMMConstantProduct.sol
│       └── ... (11 total)
└── test/
    └── (your test contracts)
```

---

## 🤝 Contributing

1. Add a new invariant template to `src/templates/`
2. Write a unit test in `test/templates/<Name>.t.sol` proving the template catches synthetic bugs
3. Submit PR with bug bounty finds that the template would have caught

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

<div align="center">

*Invariant: A property that must hold for ALL possible inputs, not just the ones you tested.*

</div>
