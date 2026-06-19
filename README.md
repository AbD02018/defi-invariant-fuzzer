<div align="center">

# 🧪 defi-invariant-fuzzer

### *Foundry-based property-testing framework for DeFi protocol invariants.*

[![Foundry](https://img.shields.io/badge/Foundry-required-000000?style=flat-square)](https://book.getfoundry.sh/)
[![Solidity](https://img.shields.io/badge/Solidity-0.8.x-363636?style=flat-square&logo=solidity&logoColor=white)](https://soliditylang.org/)
[![Templates](https://img.shields.io/badge/protocol%20templates-11-blueviolet?style=flat-square)](#-available-templates)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

</div>

---

## 🎯 Why

> *"What property must NEVER break?"*

That's the only question that matters when you're auditing a DeFi protocol. **defi-invariant-fuzzer** turns the answer to that question into a Foundry invariant test — and lets the fuzzer try to break it.

This is the same workflow I use to find real bugs on forked mainnet:
1. Identify the protocol's critical invariants (token conservation, solvency, liquidation bounds, etc.)
2. Encode them as Foundry invariant handlers
3. Generate random call sequences (deposit → borrow → liquidate → …)
4. Fuzz until the invariant breaks — or until you have confidence it can't

When the fuzzer finds a counterexample, you get:
- 🔍 **The call sequence** that broke the invariant
- 📊 **State diff** at the point of failure
- 🧬 **A reproducible Foundry test** that re-runs the same sequence
- 📝 **A candidate bug report** with measurable impact

---

## ✨ Features

- 🎯 **11 protocol-specific invariant templates** — see [Available Templates](#-available-templates)
- 🚀 **Foundry-invariant native** — Built on top of `forge invariant`, no new DSL to learn
- 🔍 **Automatic call sequence generation** — Multi-step state transitions (deposit → borrow → liquidate)
- 📊 **Failure analysis** — Renders counterexamples in human-readable form with state diff
- 🐳 **Docker-ready** — One-liner for CI/CD pipelines
- 🔌 **Composable** — Mix and match templates per protocol

---

## 📦 Installation

```bash
forge install AbD02018/defi-invariant-fuzzer --no-commit
cp -r lib/defi-invariant-fuzzer/templates/* test/invariants/
```

### From source (for development)

```bash
git clone https://github.com/AbD02018/defi-invariant-fuzzer
cd defi-invariant-fuzzer
forge install
forge test
```

### Requirements

- Foundry (`forge`, `cast`, `anvil`) — see [installation guide](https://book.getfoundry.sh/getting-started/installation)
- Solidity 0.8.x
- Python 3.10+ (optional, for the report generator)

---

## 🚀 Quick Start

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "defi-invariant-fuzzer/templates/ERC20/Conservation.sol";
import "defi-invariant-fuzzer/templates/Lending/Solvency.sol";

contract MyProtocolInvariants is ConservationInvariant, SolvencyInvariant {
    MyToken token;
    MyLendingMarket market;

    constructor(MyToken _token, MyLendingMarket _market) {
        token = _token;
        market = _market;
    }

    function _totalSupply() internal view override returns (uint256) {
        return token.totalSupply();
    }

    function _totalBalance() internal view override returns (uint256) {
        return token.balanceOf(address(this)) + token.balanceOf(market);
    }

    // ...implement remaining hooks (typically 3–5 lines per template)
}
```

Then in `foundry.toml`:

```toml
[invariant]
runs = 1000
depth = 50
fail_on_revert = false
```

Run:

```bash
forge test --match-contract MyProtocolInvariants -vvvv
```

If a counterexample is found, the trace is saved automatically.

---

## 📚 Available Templates

| # | Template | Class | What it asserts |
|---|---|---|---|
| 01 | **ERC20Conservation** | Token | `totalSupply == sum(balanceOf)` |
| 02 | **LendingSolvency** | Lending | `sum(deposits) >= sum(borrows) + reserves` |
| 03 | **AMMConstantProduct** | AMM | `x * y >= k` (Uniswap V2 style) |
| 04 | **LiquidationBounds** | Lending | `seized_collateral <= liquidatable_collateral` |
| 05 | **VaultShareSolvency** | Vault | `idle + adapters >= total_assets` (ERC-4626) |
| 06 | **OracleFreshness** | Oracle | `last_update <= staleness_threshold` |
| 07 | **GovernanceQuorum** | Governance | `votes_for >= quorum && votes_against < quorum` |
| 08 | **AccessControlMonotonicity** | Access | `role_assignments` only grow via `grantRole` |
| 09 | **BridgeMessageUniqueness** | Bridge | `processed_messages` only grows |
| 10 | **StakingRewardConservation** | Staking | `rewards_paid <= rewards_accrued` |
| 11 | **ProxyImplementationMonotonicity** | Proxy | `implementation` only changes via `upgradeTo` |

> **Need a new template?** Open an issue — most protocol-specific invariants are compositions of these 11.

---

## 🎓 When To Use This

✅ **Use defi-invariant-fuzzer when:**
- You're auditing a DeFi protocol and want to catch logic bugs static analysis misses
- You have a fork-based Foundry project and want to fuzz behavior
- You want to verify an audit fix actually works
- You want a "did I break anything?" test suite for a refactor

❌ **Don't use it when:**
- The bug is in the EVM/solc layer (use [Echidna](https://github.com/crytic/echidna) or [Medusa](https://github.com/crytic/medusa) instead)
- The bug is in off-chain logic (this is a Solidity tool)
- You don't have a Foundry project yet (start with [solidity-forge-template](https://github.com/AbD02018/solidity-forge-template))

---

## 🤝 Contributing

PRs welcome for:
- New protocol-specific templates
- More efficient fuzzer targets (call-sequence generators)
- Better counterexample analysis (post-processing traces)
- Documentation improvements

See [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution guide.

---

## 📄 License

MIT — see [LICENSE](LICENSE).

---

## 🙏 Acknowledgments

- [Foundry](https://github.com/foundry-rs/foundry) — the engine
- [Trail of Bits](https://github.com/crytic) — `Echidna`, `slither`, `manticore` — the inspiration
- [Ethereum Foundation](https://ethereum.foundation/) — for funding the security tooling ecosystem

---

<div align="center">
  <sub>Built by <a href="https://github.com/AbD02018">@AbD02018</a> · Smart contract security researcher</sub>
</div>
