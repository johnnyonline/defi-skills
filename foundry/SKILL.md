# Foundry

Ethereum toolkit. Docs: https://book.getfoundry.sh

## Install

```bash
curl -L https://foundry.paradigm.xyz | bash
export PATH="$HOME/.foundry/bin:$PATH"
foundryup
```

## cast — Read

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

## cast — Write

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

## cast — Conversions

```bash
# ETH to wei
cast to-wei 1.5

# Wei to ETH
cast from-wei 1500000000000000000

# Hex to decimal
cast to-dec 0x4c1e57

# Decimal to hex
cast to-hex 4988503

# ABI encode
cast abi-encode "transfer(address,uint256)" <to> <amount>

# ABI decode
cast abi-decode "balanceOf(address)(uint256)" <data>

# Keccak256
cast keccak "Transfer(address,address,uint256)"
```

## cast — Wallet

```bash
# Generate new keypair
cast wallet new

# Get address from private key
cast wallet address --private-key $PK
```

## cast — ENS

```bash
cast resolve-name vitalik.eth --rpc-url $RPC
cast lookup-address <address> --rpc-url $RPC
```

## cast — Logs

```bash
cast logs --from-block <block> --to-block <block> --address <contract> <topic0> --rpc-url $RPC
```

## Tips

- `cast call` = read (free), `cast send` = write (costs gas)
- For ERC20 amounts, mind the decimals (USDC = 6, most tokens = 18)
- Use `cast --help` or `cast <command> --help` for full reference
