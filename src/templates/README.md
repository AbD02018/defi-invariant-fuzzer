# 🧪 DeFi Invariant Fuzzer — Base Templates

Production-grade Solidity templates for invariant-based fuzzing of DeFi protocols.

## Core Templates

### `LendingSolvency.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import "./BaseInvariant.sol";

/// @title LendingSolvency
/// @notice Invariant: Total collateral value >= Total debt value
abstract contract LendingSolvency is BaseInvariant {
    /// @notice Register the protocol under test
    function _registerProtocol(address protocol) internal virtual;

    /// @notice Check if the protocol is solvent
    function _isSolvent(address protocol) internal view virtual returns (bool);
}
```

### `AMMConstantProduct.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import "./BaseInvariant.sol";

/// @title AMMConstantProduct
/// @notice Invariant: x * y >= k for all AMM pools
abstract contract AMMConstantProduct is BaseInvariant {
    uint256 internal constant K_TOLERANCE = 100; // wei tolerance for fees
}
```

### `LendingLiquidation.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import "./BaseInvariant.sol";

/// @title LendingLiquidation
/// @notice Invariant: Liquidation never increases protocol debt
abstract contract LendingLiquidation is BaseInvariant {
    function _isLiquidationSafe(address protocol) internal view virtual returns (bool);
}
```

## Usage

See the full README at the repository root.
