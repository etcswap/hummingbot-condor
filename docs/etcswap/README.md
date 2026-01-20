# ETCswap Integration for Condor (Telegram Bot)

This guide explains how to use ETCswap with Condor for trading on Ethereum Classic via Telegram.

## Overview

ETCswap is the leading decentralized exchange on Ethereum Classic. Through Condor, you can:

- Execute swaps on ETCswap directly from Telegram
- Manage V3 liquidity positions
- Monitor token balances
- Set up automated trading routines

## Prerequisites

1. **Condor Bot** running and connected to your Telegram
2. **Hummingbot Backend API** running
3. **Hummingbot Gateway** running with ETCswap connector
4. **Ethereum Classic wallet** added to Gateway

## Architecture

```
Telegram → Condor Bot → Hummingbot API → Gateway → ETCswap
```

## Setting Up ETCswap

### Step 1: Configure API Server

1. Type `/config` in Telegram
2. Select "API Servers"
3. Add or verify your Hummingbot API server is configured
4. Ensure Gateway URL is set in the API server

### Step 2: Add ETC Wallet

Before trading, add your ETC wallet to Gateway:

1. Via MCP: "Add my ETC wallet with private key ..."
2. Via API: POST to `/gateway/wallet/add`
3. Via Gateway CLI directly

### Step 3: Approve Tokens

Tokens must be approved before trading. Use the `/swap` menu or API to approve USC and other tokens for ETCswap.

## Trading Commands

### /swap - Token Swaps

Execute swaps on ETCswap:

1. Type `/swap`
2. Select "Swap Quote" or "Execute Swap"
3. Enter details:
   - Connector: `etcswap`
   - Network: `classic`
   - Base: `ETC`
   - Quote: `USC`
   - Amount: `1.0`
4. Review quote and execute

**Quick Swap:** After your first swap, use "Quick Swap" to repeat with the same parameters.

### /lp - Liquidity Management

Manage V3 LP positions:

1. Type `/lp`
2. Options:
   - **LP Positions** - View your positions
   - **Pool Info** - Get pool details
   - **Add Liquidity** - Create new position
   - **Close Position** - Remove liquidity
   - **Collect Fees** - Claim earned fees

### Gateway Balances

Check your ETC wallet balances:

1. Type `/swap`
2. Select "Gateway Balances"
3. Select network: `classic`
4. View ETC and token balances

## Example Workflows

### Swap ETC for USC

```
/swap
→ Swap Quote
→ Connector: etcswap
→ Network: classic
→ Base: ETC
→ Quote: USC
→ Amount: 1.0
→ Side: sell
→ [Shows quote]
→ [Execute]
→ Swap completed! TX: 0x...
```

### Add Liquidity to ETC-USC Pool

```
/lp
→ Add Liquidity
→ Connector: etcswap_lp
→ Network: classic
→ Token0: WETC
→ Token1: USC
→ Fee: 3000 (0.3%)
→ Lower Price: 15.0
→ Upper Price: 25.0
→ Amount0: 1.0
→ Amount1: 20.0
→ [Shows preview]
→ [Confirm]
→ Position created! Token ID: 12345
```

### Check LP Positions

```
/lp
→ LP Positions
→ [Shows all positions with:]
  - Token ID
  - Pool pair
  - In-range status
  - Liquidity amount
  - Uncollected fees
```

## Automation Routines

### LP Monitor

Monitor your LP positions for range status:

```
/routines
→ lp_monitor
→ Configure:
  - Network: classic
  - Check Interval: 5 minutes
  - Alert on out-of-range: Yes
→ Start
```

### Price Monitor

Set price alerts for ETC:

```
/routines
→ price_monitor
→ Configure:
  - Connector: etcswap
  - Pair: ETC-USC
  - Alert Price: 20.0
  - Alert Type: above/below
→ Start
```

### Arbitrage Check

Monitor CEX/DEX price differences:

```
/routines
→ arb_check
→ Configure:
  - CEX: gate_io (ETC-USDT)
  - DEX: etcswap (ETC-USC)
  - Min Spread: 1.0%
→ Start
```

## Trading Pairs

| Pair | Description |
|------|-------------|
| ETC-USC | ETC vs Classic USD stablecoin |
| WETC-USC | Wrapped ETC vs USC |

## Key Tokens

| Token | Symbol | Address |
|-------|--------|---------|
| Wrapped ETC | WETC | `0x1953cab0E5bFa6D4a9BaD6E05fD46C1CC6527a5a` |
| Classic USD | USC | `0xDE093684c796204224BC081f937aa059D903c52a` |

## Connector Names

| Connector | Use Case |
|-----------|----------|
| `etcswap` | Swaps via Universal Router |
| `etcswap_lp` | V3 LP positions |

## Networks

| Network | Chain ID | Description |
|---------|----------|-------------|
| `classic` | 61 | Ethereum Classic mainnet |
| `mordor` | 63 | Mordor testnet |

## USD Stablecoin Info

**USC (Classic USD)** is 1:1 with USDC through [Brale Platform](https://brale.xyz).

This enables arbitrage between:
- Coinbase: ETC/USDC
- ETCswap: ETC/USC

## Troubleshooting

### "DEX features unavailable"

- Ensure Gateway is running
- Check Gateway is configured in API server
- Verify ETCswap connector is enabled

### "Wallet not found"

- Add wallet to Gateway first
- Use correct network (`classic`)

### "Insufficient balance"

- Check ETC balance for gas
- Check token balance for swap

### "Swap failed"

- Increase slippage tolerance
- Check token approvals
- Verify pool has liquidity

### "Connection refused"

- Check API server is running
- Verify host:port in `/config`

## Bot Commands Reference

| Command | Description |
|---------|-------------|
| `/portfolio` | View portfolio with ETC holdings |
| `/swap` | Execute DEX swaps |
| `/lp` | Manage LP positions |
| `/config` | Configure servers, wallets |
| `/routines` | Automation scripts |
| `/bots` | Monitor trading bots |
| `/trade` | CEX trading (for ETC on CEXs) |

## Resources

- [ETCswap Website](https://etcswap.org)
- [Brale Platform](https://brale.xyz) - USC/USDC bridge
- [Classic USD](https://classicusd.com)
- [Gateway ETCswap Docs](../../hummingbot-gateway/docs/etcswap/)
