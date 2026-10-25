---
title: A Practical Guide to Writing Gas-Efficient Solidity Smart Contracts
date: Jun 19, 2025
tags: Solidity, Smart Contracts, Gas Optimization, Foundry, Blockchain
---

Paste the body of your post here, as Markdown. Everything above the second `---` line is the post's title, date and tags; leave that as it is.

```solidity
// Code blocks work like this
function sum(uint256[] calldata xs) external pure returns (uint256 s) {
    for (uint256 i; i < xs.length; ++i) s += xs[i];
}
```
