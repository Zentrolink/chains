# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **ethereum-lists/chains** repository - the canonical source for EVM-based blockchain chain metadata. It contains 2300+ chain definitions used by wallets, explorers, and dApps across the Ethereum ecosystem.

Data is served at:
- https://chainid.network/chains.json (full)
- https://chainid.network/chains_mini.json (minified)

## Repository Structure

```
_data/
├── chains/          # Chain JSON files (eip155-{chainId}.json)
├── icons/           # Icon metadata JSON files
└── iconsDownload/   # Downloaded icon files
processor/           # Kotlin validation processor
tools/               # Utility tools
.github/workflows/   # CI workflows
```

## Chain File Format

Chain files are stored in `_data/chains/` with CAIP-2 naming: `eip155-{chainId}.json`

Required fields:
- `name`: Unique chain name
- `chain`: Chain symbol (e.g., "ETH")
- `chainId`: Unique numeric chain ID
- `networkId`: Network ID (usually same as chainId)
- `shortName`: Unique short identifier
- `nativeCurrency`: Object with `name`, `symbol`, `decimals`
- `rpc`: Array of RPC endpoint URLs
- `faucets`: Array of faucet URLs (can be empty)
- `infoURL`: Chain info website

Optional fields:
- `icon`: Reference to icon in `_data/icons/`
- `explorers`: Array of block explorer objects
- `features`: Array of supported features (e.g., EIP155, EIP1559)
- `parent`: For L2/shard chains, reference to parent chain
- `status`: "active" (default), "deprecated", or "incubating"
- `slip44`: SLIP-44 coin type
- `ens`: ENS registry configuration

## Common Commands

### Validate all chains
```bash
./gradlew run
```

### Validate a single chain file
```bash
./gradlew clean run --args="verbose singleChainCheck _data/chains/eip155-1.json"
```

### Format JSON files with Prettier
```bash
npx prettier --write '_data/*/*.json'
```

### Check JSON formatting
```bash
npx prettier --check '_data/*/*.json'
```

## Key Constraints

1. **Uniqueness**: `shortName` and `name` must be globally unique across all chains
2. **No deletions**: Chains cannot be deleted, only deprecated (prevents replay attacks)
3. **Parent validation**: If referencing a parent chain, it must exist in the repo
4. **Icon requirements**:
   - Must be IPFS URLs (publicly retrievable via `ipfs get`)
   - Size must be less than 250kb
   - Format: png, jpg, or svg
   - Corresponding JSON must exist in `_data/icons/`
5. **ChainID collision**: First PR gets the chainID; reassignment only possible if old chain is deprecated

## CI Checks

PRs must pass:
1. **Build workflow**: Runs `./gradlew run` validation
2. **Prettier check**: JSON formatting validation
3. **Single chain check**: Validates only changed files

## Adding a New Chain

1. Create `_data/chains/eip155-{chainId}.json` with required fields
2. If using an icon, create corresponding `_data/icons/{iconName}.json`
3. Run `./gradlew run` to validate
4. Run `npx prettier --write '_data/chains/eip155-{chainId}.json'` to format
5. Submit PR

## Deprecating a Chain

Add `"status": "deprecated"` to the chain JSON. Never delete chain files.
