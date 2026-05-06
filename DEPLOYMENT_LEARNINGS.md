# Arc Deployment Learnings, Successes, Failures, and Best Practices

This document captures practical lessons from deploying and running zkVVM components on Arc Testnet.

## Scope

Work covered:

- Deploying `ZkVaultService-contracts` on Arc Testnet
- Patching `zkVaultService-demo` from Sepolia to Arc
- Verifying contracts on Arc explorer
- Running the frontend locally against Arc

## Key Learnings

### 1) Arc integration requires explicit chain replacement in app code

The demo was hardcoded for Sepolia in multiple places (wagmi provider and API routes). A complete migration required updating all chain references, not just environment variables.

### 2) Root relayer alignment is critical

`ROOT_RELAYER_ADDRESS` must match the EOA derived from `FISHER_PRIVATE_KEY`. If they do not match, `registerRoot` calls fail with authorization errors.

### 3) Contract deployment script was chain-agnostic and reusable

The existing Foundry deploy script worked on Arc without logic changes once RPC + addresses were supplied.

### 4) Arc USDC endpoint behaved as expected for ERC20 calls

The provided USDC address (`0x3600...0000`) responded to `decimals/symbol/name`, which de-risked deposit flow assumptions.

### 5) Verification path on Arc is Blockscout-compatible

Using Foundry verification with Blockscout-style endpoint and watch polling succeeded for all contracts.

## Successes

- Successfully deployed all three contracts to Arc Testnet:
  - WithdrawFromPoolVerifier
  - SplitNoteVerifier
  - ShieldedPool
- Successfully configured root relayer on-chain
- Successfully migrated frontend + API chain config to Arc
- Successfully launched frontend locally (`http://localhost:3000`)
- Successfully verified all deployed contracts on Arc explorer

## Failures / Friction Points

### 1) NPM peer dependency conflict in contracts repo

Initial `npm install` failed due to peer conflicts between `@evvm/testnet-contracts` and OpenZeppelin versions. Workaround used: `npm install --legacy-peer-deps`.

### 2) Foundry install flag mismatch

`forge install ... --no-commit` was not supported in installed Forge version. Resolved by manually cloning `forge-std` tag into `lib/forge-std`.

### 3) Deployment blocked until faucet funding

Broadcast deployment initially could not proceed because deployer balance was zero; funding from Arc faucet was required first.

### 4) Plan/intent mismatch on “4 contracts” wording

Only three contracts were deployed; the fourth item in the list was root relayer EOA (not contract). This caused verification expectation ambiguity.

### 5) Turbopack workspace-root behavior can break Tailwind resolution

With multiple lockfiles on the machine, Next/Turbopack inferred the workspace root from `/Users/openclaw/package-lock.json`. When a `turbopack.root` override was present in this setup, module resolution intermittently failed with `Can't resolve 'tailwindcss' in '/Users/openclaw/Documents/zkVVM'`.

### 6) Server-side localStorage shim conflict caused runtime crash

The runtime error `localStorage.getItem is not a function` was caused by a malformed `globalThis.localStorage` object in the server runtime context. A guarded shim in `next.config.ts` ensured `getItem` exists and stabilized rendering.

## Best Practices

### Deployment best practices

1. **Run dry-run before broadcast** using `forge script` without `--broadcast`.
2. **Check deployer balance first** on Arc before spending time on final deploy step.
3. **Validate external dependencies** (EVVM Core/Staking addresses and bytecode) before deployment.
4. **Capture deploy outputs immediately** and store in docs/env files.

### Configuration best practices

1. **Use a dedicated chain module** (single source of truth for chain id, RPC, explorer).
2. **Keep env files explicit** and avoid hidden defaults for chain-dependent values.
3. **Use same key for fisher/root relayer in dev** to reduce config mismatch risk.

### Runtime/operational best practices

1. **Perform smoke checks** (`cast call decimals/symbol/name`) on token addresses.
2. **Use contract verification as final acceptance gate** after deploy.
3. **Treat API private keys as secrets**; never commit `.env` values.
4. **Document explorer links** for all deployed artifacts for easy audit and debugging.
5. **Track server/runtime globals while debugging** (especially `globalThis.localStorage`) when browser-like APIs appear in SSR errors.
6. **Use A/B config testing for build issues** (toggle one config at a time, verify with logs, then keep only proven changes).

## Suggested Improvements for Next Iteration

- Add Arc network as a first-class option in upstream demo (instead of manual replacement).
- Add startup validation that confirms chain ID and required env variables at boot.
- Add a deployment output JSON artifact committed to repo docs (without secrets).
- Add script to auto-generate `.env` templates from deployment outputs.

## Final Outcome

The Arc deployment and local launch were successful end-to-end, with contract verification complete and the frontend running against Arc Testnet.
