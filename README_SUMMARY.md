# 🧪 DeFi Invariant Fuzzer — Base Invariant Framework

Foundry-native property-testing framework for DeFi protocol invariants. Catch billion-dollar bugs before deployment.

## Quick Start

```bash
# Install
forge install AbD02018/defi-invariant-fuzzer --no-commit

# Use a template
cp lib/defi-invariant-fuzzer/src/templates/LendingSolvency.sol test/invariants/MyLendingInvariants.sol

# Run
forge invariant --match-contract MyLendingInvariants --depth 50
```

## Why Invariant Fuzzing?

Most DeFi hacks share a common pattern: a **property that must NEVER break** got broken.
- **Cream Finance** ($130M): borrowing power miscalculation
- **Harvest Finance** ($34M): oracle price manipulation
- **Badger DAO** ($120M): function permission misconfiguration

These are not complex exploits. They're invariants that the protocol failed to enforce or check.

## Templates Included

| Template | Property |
|---|---|
| `LendingSolvency` | `Σ(collateral_value) >= Σ(debt_value)` for all accounts |
| `LendingLiquidation` | Liquidation never makes protocol insolvent |
| `AMMConstantProduct` | `x * y = k` for all pool states |
| `StakingRewards` | `Σ(rewards_owed) <= rewards_balance` |
| `VaultShareConservation` | `Σ(user_shares) == totalSupply()` |
| `OracleNoStaleness` | All prices updated within N seconds |
| `BridgeMessageUniqueness` | No double-spend in cross-chain |
| `GovernanceQuorum` | `Σ(votes) <= totalSupply()` |
| `OrderbookConservation` | Bid+Ask fills cancel out |
| `LendingAccountHealth` | Health factor monotonic under partial liquidations |
| `ERC20Conservation` | `Σ(balance) == totalSupply()` for all users |

## Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "defi-invariant-fuzzer/src/templates/LendingSolvency.sol";

contract MyLendingInvariantTest is LendingSolvency {
    MyLending lending;
    
    function setUp() public {
        lending = new MyLending();
        _registerProtocol(address(lending));
    }
    
    /// @custom:invariant Protocol is always solvent
    function invariant_protocol_solvency() public {
        assertTrue(_isSolvent(lending), "Protocol insolvent");
    }
}
```

## Configuration

```toml
# foundry.toml
[invariant]
runs = 256
depth = 50
fail_on_revert = false
call_override = false
dictionary_weight = 80
include_storage = true
include_push_bytes = true
```

## Architecture

```
BaseInvariant (abstract)
├── targets[]      # Protocol contracts under test
├── actors[]       # Fuzzed user accounts
├── ghost_vars     # Internal accounting for invariant checks
└── handlers[]     # Action selectors the fuzzer can call
```

Handlers define the **state transitions** the fuzzer explores. Invariants define the **properties** that must hold after every transition.

## License

MIT
