# zkVVM on Arc Testnet

A working deployment and local launch guide for the zkVVM stack on **Arc Testnet (Chain ID 5042002)** using:

- `EVVM-org/ZkVaultService-contracts` (smart contracts)
- `EVVM-org/zkVaultService-demo` (Next.js dApp)

This repository summarizes what was deployed, how the app was configured for Arc, and how to run it locally.

## What This App Does

The application demonstrates private and compliant USDC flows with a shielded-pool model:

- **Deposit:** Convert visible USDC into a private note commitment
- **Withdraw:** Redeem a note to a destination address using a zero-knowledge proof
- **Split:** Split one note into four smaller commitments

The frontend runs locally and interacts with Arc Testnet through EVVM Core and a deployed ShieldedPool.

## Deployed Contracts (Arc Testnet)

- **WithdrawFromPoolVerifier**  
  `0xe54D1758301a09b3323903f17762Fba86FCFd5AB`
- **SplitNoteVerifier**  
  `0x51b83270BF60d203D43222fd5F6b81514aF0A647`
- **ShieldedPool**  
  `0xab47C6B86B9BfE2E82Bc8aeB44c4a2Ce0581f2ad`

Root relayer configured on ShieldedPool:

- **Root Relayer EOA**  
  `0x5d1058aa18823642717De6A37669cad0e41d75e1`

## Arc + EVVM Parameters Used

- **Arc RPC:** `https://rpc.testnet.arc.network`
- **Arc Explorer:** `https://testnet.arcscan.app`
- **Chain ID:** `5042002`
- **EVVM Core:** `0x8828A715795877dcfE0d12d190B21596bEDA8870`
- **EVVM Staking:** `0x0753A6702861B77e844Cb426Fbf102D3e9Dd7550`
- **USDC Token:** `0x3600000000000000000000000000000000000000`

## Local dApp Configuration

The demo app was adapted from Sepolia to Arc by:

1. Adding a custom Arc chain definition (`defineChain`) 
2. Replacing Sepolia in wagmi provider config with Arc Testnet
3. Replacing Sepolia in API route wallet clients (`/api/fisher`, `/api/shielded-pool/register-root`)
4. Updating UI labels from Sepolia to Arc Testnet

## Required Environment Variables

### Contracts repo (`ZkVaultService-contracts/.env`)

```bash
DEPLOYER_PRIVATE_KEY=<funded arc key>
EVVM_CORE_ADDRESS=0x8828A715795877dcfE0d12d190B21596bEDA8870
EVVM_STAKING_ADDRESS=0x0753A6702861B77e844Cb426Fbf102D3e9Dd7550
EVVM_USDC_TOKEN_ADDRESS=0x3600000000000000000000000000000000000000
ROOT_RELAYER_ADDRESS=<address derived from fisher private key>
ARC_TESTNET_RPC_URL=https://rpc.testnet.arc.network
```

### Demo repo (`zkVaultService-demo/.env`)

```bash
NEXT_PUBLIC_ZKVAULT_ADDRESS=0xab47C6B86B9BfE2E82Bc8aeB44c4a2Ce0581f2ad
NEXT_PUBLIC_EVVM_CORE_ADDRESS=0x8828A715795877dcfE0d12d190B21596bEDA8870
NEXT_PUBLIC_USDC_TOKEN_ADDRESS=0x3600000000000000000000000000000000000000
FISHER_PRIVATE_KEY=<private key whose address == ROOT_RELAYER_ADDRESS>
```

## Run Locally

```bash
# frontend demo
cd zkVaultService-demo
npm install
npm run dev
```

Then open `http://localhost:3000` and connect a wallet on Arc Testnet.

## Verification Status

All deployed contracts were verified on Arc explorer:

- `https://testnet.arcscan.app/address/0xe54d1758301a09b3323903f17762fba86fcfd5ab`
- `https://testnet.arcscan.app/address/0x51b83270bf60d203d43222fd5f6b81514af0a647`
- `https://testnet.arcscan.app/address/0xab47c6b86b9bfe2e82bc8aeb44c4a2ce0581f2ad`

## Notes

- Root relayer is an EOA, not a contract (so it is configured, not contract-verified).
- On Arc, gas is paid with USDC native token semantics; ensure deployer/fisher accounts are funded before deployment and runtime actions.
