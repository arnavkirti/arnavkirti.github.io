---
title: A Practical Guide to Writing Gas-Efficient Solidity Smart Contracts
date: Jun 19, 2025
tags: Solidity, Smart Contracts, Gas Optimization, Foundry, Blockchain
---

## 🧠 Why Gas Optimization Matters

Every operation in Ethereum consumes gas, a unit that represents computational cost. Efficient contracts cost users less to interact with, improve UX, and help your dApp scale. Plus, gas optimization often aligns with better logic and cleaner code.

In this blog, we'll walk through:

- Common gas inefficiencies and how to fix them
- Tools like Foundry for gas benchmarking
- Real Solidity code examples
- Practical optimization patterns

## ⚒️ Setup: Foundry for Gas Benchmarking

We'll use Foundry, a powerful tool for Ethereum development, to benchmark gas usage. Install it with:

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

Create a new project:

```bash
forge init GasOptimization
cd GasOptimization
```

## ⚠️ Common Gas Pitfalls and Fixes

### 🔁 Avoid Unbounded Loops

```solidity
// ❌ Bad
function sum(uint[] memory arr) public pure returns (uint) {
    uint total;
    for (uint i = 0; i < arr.length; i++) {
        total += arr[i];
    }
    return total;
}
```

**Why it's bad:** The loop depends on input size. Gas costs become unbounded.

✅ **Fix:** Avoid loops for storage/memory writes. Offload to frontends or batch with limits.

### 📦 Use uint256 Over Smaller Types

```solidity
// ❌ Bad
uint8 a = 1;
uint8 b = 2;
```

**Why it's bad:** EVM reads in 32-byte words. Mixing types causes inefficient packing.

✅ **Fix:** Use `uint256` everywhere. It's the EVM's native word size and avoids packing issues.

```solidity
uint256 a = 1;
uint256 b = 2;
```

### 💾 Pack Storage Variables

```solidity
// ❌ Unpacked
uint256 a;
bool b;
uint256 c;
```

**Why it's bad:** Each storage slot costs 20,000 gas. Unpacked variables waste space.

✅ **Fix:** Pack variables into fewer slots. Use structs or arrays to group related data.

```solidity
// ✅ Packed
uint256 a;
uint256 b;
bool c;
```

### 🗂️ Use calldata for External Function Parameters

```solidity
// ❌ Memory
function process(uint[] memory data) external {}

// ✅ Calldata
function process(uint[] calldata data) external {}
```

`calldata` is cheaper for external functions since it avoids memory allocation.

### 🧠 Short-Circuit Booleans

```solidity
// ❌ Evaluates both
if (checkA() && checkB()) {}

// ✅ Short-circuits if checkA() is false
if (!checkA()) return;
if (checkB()) {}
```

## 🧪 Benchmarking With Foundry

Let's create a simple test to compare gas.

### Example Contract

```solidity
// SPDX-License-Identifier: MIT
// src/GasExample.sol
pragma solidity ^0.8.24;

contract GasExample {
    // Inefficient
    function sumMemory(uint[] memory arr) external pure returns (uint total) {
        for (uint i = 0; i < arr.length; i++) {
            total += arr[i];
        }
    }

    // Efficient (calldata)
    function sumCalldata(uint[] calldata arr) external pure returns (uint total) {
        for (uint i = 0; i < arr.length; i++) {
            total += arr[i];
        }
    }
}
```

### Test File

```solidity
// SPDX-License-Identifier: MIT
// test/GasExample.t.sol
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import "../src/GasExample.sol";

contract GasExampleTest is Test {
    GasExample gasExample;

    function setUp() public {
        gasExample = new GasExample();
    }

    function testGasSumMemory() public view {
        uint[] memory arr = new uint[](5);
        for (uint i = 0; i < 5; i++) {
            arr[i] = i;
        }
        uint result = gasExample.sumMemory(arr);
        // Expected result: 0 + 1 + 2 + 3 + 4 = 10
        assertEq(result, 10);
    }

    function testGasSumCalldata() public view {
        uint[] memory arr = new uint[](5);
        for (uint i = 0; i < 5; i++) {
            arr[i] = i;
        }
        uint result = gasExample.sumCalldata(arr);
        // Expected result: 0 + 1 + 2 + 3 + 4 = 10
        assertEq(result, 10);
    }
}
```

### Run Benchmark

```bash
forge test --gas-report
```

You will see gas usage for each function. Compare the results to see the impact of optimizations.

| Function Name      | Max  |
|--------------------|------|
| testGasSumCalldata | 2226 |
| testGasSumMemory   | 3699 |

`calldata` saved us over 1,000 gas compared to `memory`!

## ✅ General Optimization Checklist

- Use the latest Solidity version for optimizations
- Avoid unbounded loops and recursion
- Use `calldata` for external function parameters
- Pack storage variables efficiently
- Short-circuit boolean expressions
- Minimize state changes in functions
- Leverage libraries like OpenZeppelin for common patterns

## 🧠 Final Thoughts

Gas optimization isn't about micro-obsessing — it's about knowing where it matters. If your contract will be called millions of times or touches expensive opcodes, optimize. Otherwise, prioritize readability and security first.

## 📚 Further Reading

- [Ethereum.org — Gas and fees](https://ethereum.org/en/developers/docs/gas/)
- [RareSkills — Gas optimization](https://www.rareskills.io/post/gas-optimization)
- [Foundry documentation](https://book.getfoundry.sh/) for testing and benchmarking