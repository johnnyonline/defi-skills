# Foundry

Ethereum toolkit for onchain interactions. Docs: https://book.getfoundry.sh

## Install

```bash
curl -L https://foundry.paradigm.xyz | bash
export PATH="$HOME/.foundry/bin:$PATH"
foundryup
```

## Tools

- `cast` — CLI for onchain reads, writes, conversions
- `forge` — build, test, deploy Solidity contracts
- `anvil` — local testnet node
- `chisel` — Solidity REPL

## cast — Common Operations

### Read

```bash
# ETH balance
cast balance <address> --rpc-url $RPC --ether

# Call view function
cast call <contract> "balanceOf(address)(uint256)" <address> --rpc-url $RPC

# Get storage slot
cast storage <contract> <slot> --rpc-url $RPC

# Block number
cast block-number --rpc-url $RPC

# Get tx receipt
cast receipt <tx_hash> --rpc-url $RPC
```

### Write

```bash
# Send ETH
cast send <to> --value <amount_wei> --rpc-url $RPC --private-key $PK

# Call contract function
cast send <contract> "transfer(address,uint256)" <to> <amount> --rpc-url $RPC --private-key $PK

# ERC20 approve
cast send <token> "approve(address,uint256)" <spender> <amount> --rpc-url $RPC --private-key $PK

# Send raw calldata
cast send <to> <calldata> --value <wei> --rpc-url $RPC --private-key $PK
```

### Conversions

```bash
# ETH to wei
cast to-wei 1.5    # 1500000000000000000

# Wei to ETH
cast from-wei 1500000000000000000    # 1.5

# Hex to decimal
cast to-dec 0x4c1e57    # 4988503

# Decimal to hex
cast to-hex 4988503

# To checksummed address
cast to-check-sum-address <address>

# ABI encode
cast abi-encode "transfer(address,uint256)" <to> <amount>

# ABI decode
cast abi-decode "balanceOf(address)(uint256)" <data>

# Keccak256
cast keccak "Transfer(address,address,uint256)"
```

### Wallet

```bash
# Generate new keypair
cast wallet new

# Get address from private key
cast wallet address --private-key $PK
```

### ENS

```bash
# Resolve ENS name
cast resolve-name vitalik.eth --rpc-url $RPC

# Reverse resolve
cast lookup-address <address> --rpc-url $RPC
```

### Logs & Events

```bash
# Get logs by topic
cast logs --from-block <block> --to-block <block> --address <contract> <topic0> --rpc-url $RPC
```

## forge — Common Operations

```bash
# Init new project
forge init my-project

# Build
forge build

# Test
forge test -vvv

# Deploy
forge create src/Contract.sol:Contract --rpc-url $RPC --private-key $PK

# Verify on Etherscan
forge verify-contract <address> src/Contract.sol:Contract --etherscan-api-key $KEY --chain mainnet
```

## anvil — Local Testnet

```bash
# Start local node
anvil

# Fork mainnet
anvil --fork-url $RPC

# Fork at specific block
anvil --fork-url $RPC --fork-block-number 12345678
```

## Tips

- Always use `--rpc-url` (no default)
- `cast call` = read (free), `cast send` = write (costs gas)
- For ERC20 amounts, mind the decimals (USDC = 6, most tokens = 18)
- Use `cast --help` or `cast <command> --help` for full reference
