# Minimal Account Abstraction (Foundry project)

This repository demonstrates a minimal account abstraction implementation (smart contract accounts) built and tested with Foundry. It includes two example account implementations and a test harness for simulating EIP-4337-style user operations and zkSync Era transactions.

Short summary:
- `MinimalAccount` (EVM / EIP-4337-like) — simple owner-based smart contract account that works with an `EntryPoint` via PackedUserOperations.
- `ZkMinimalAccount` (zkSync Era) — same owner-based idea adapted to zkSync Era system contracts & transaction format.

This repository is primarily for learning and experimentation — to show how simple smart contract accounts can validate and execute signed operations in both standard EVM and zkSync Era environments.

---

## Structure — where to look

- `src/ethereum/MinimalAccount.sol` — EIP-4337-style, validates PackedUserOperation signatures and pays pre-fund to an EntryPoint.
- `src/zksync/ZkMinimalAccount.sol` — zkSync Era account implementing `validateTransaction`, `executeTransaction` and account lifecycle hooks.
- `test/ethereum/MinimalAccountTest.t.sol` — tests for the Ethereum/EIP-4337 account and operation handling.
- `test/zksync/ZkMinimalAccountTest.t.sol` — tests for the zkSync Era account (uses Foundry-era utilities & system-mode concepts).
- `script/DeployMinimal.s.sol` — deployment script for `MinimalAccount` used in tests and examples.
- `script/SendPackedUserOp.s.sol` — helper that creates and signs a `PackedUserOperation` and calls the EntryPoint `handleOps` (simulates a real user op flow).
- `lib/*` — external libraries used (OpenZeppelin, account-abstraction/EntryPoint, foundry-era contracts and test utilities).

---

## Key behavior (quick explainer)

- MinimalAccount (EIP-4337 style):
	- Uses `owner()` as the signer for the smart account.
	- Implements `validateUserOp` which:
		- Recovers the signer from the userOp hash (EIP-191 / `toEthSignedMessageHash`) via `ECDSA.recover`.
		- Returns a validation result and optionally pays missing pre-fund amounts to the entry point.
	- `execute` method allows the `entryPoint` or `owner` to call arbitrary functions on other contracts.

- ZkMinimalAccount (zkSync Era):
	- Works with zkSync Era's system contracts and transaction structure (`Transaction` struct, NonceHolder, Bootloader, Deployer).
	- `validateTransaction` must increment the nonce (uses the NonceHolder system contract) and verify signatures on the transaction hash.
	- `executeTransaction` performs low level calls and can route deployer-system-contract calls via the system-call helper.

---

## Tests — what’s covered

- `test/ethereum/MinimalAccountTest.t.sol` covers:
	- Owner-only `execute` behavior (owner can call; others cannot).
	- Proper signature recovery from a signed PackedUserOperation (using `SendPackedUserOp` helper).
	- `validateUserOp` returning expected validation code when called from EntryPoint with missing pre-fund paid.
	- Full flow where EntryPoint `handleOps` is called and the account op mints tokens for the account.

- `test/zksync/ZkMinimalAccountTest.t.sol` covers:
	- The owner can call `executeTransaction` to perform actions like minting an ERC20.
	- `validateTransaction` verifies the transaction signature and returns `ACCOUNT_VALIDATION_SUCCESS_MAGIC`.
	- Helpers demonstrate how zkSync transactions are constructed and signed in tests.

Notes on running zk tests: zkSync Era tests use special Foundry-era utilities and some tests use a chain checker (see `ZkSyncChainChecker`) — some tests may require running in system-mode or a zk-enabled environment (Foundry flags like `--system-mode=true` or a local zk test environment).

---

## Quick commands

Build the project:
```bash
forge build
```

Run the full test suite (all tests):
```bash
forge test
```

Run just the Ethereum tests (examples):
```bash
forge test --match-path test/ethereum
```

Run zkSync-specific tests (you may need system-mode / zk tools):
```bash
forge test --match-path test/zksync --ffi --system-mode=true
```

Example deployment / run script (will use configured network settings):
```bash
forge script script/DeployMinimal.s.sol:DeployMinimal --rpc-url <RPC_URL> --private-key <KEY>
```

Example: send a signed PackedUserOperation via script (the script uses `DevOpsTools.get_most_recent_deployment`):
```bash
forge script script/SendPackedUserOp.s.sol:SendPackedUserOp --rpc-url <RPC_URL> --private-key <KEY>
```

---

## Recommended next steps / ideas

- Add a `README` section describing which EntryPoint or network the project expects in `HelperConfig` to make local testing and deployment easier for new contributors.
- Add CI (GitHub Actions) to run `forge test` for quick feedback.
- Extend account behavior with support for paymasters, multi-sig owners, or modules to better illustrate account abstraction features.

---

If you want, I can also:
- Run the full `forge test` here and report results (or help debug any failing tests).
- Add a shorter / beginner-friendly README snippet at the project root (quick start) if you'd like a condensed version for newcomers.

Happy to continue — tell me which of the suggested next steps you'd like me to take. ✅
