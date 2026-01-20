# Hummingbot Condor Development Instructions

Instructions for AI coding assistants working on Hummingbot Condor.

## Project Vision

Condor is a **Telegram bot** for monitoring and trading with Hummingbot via the Backend API. It provides a complete trading interface for both centralized exchanges (CEX) and decentralized exchanges (DEX) directly through Telegram commands.

**Current Focus:** Supporting ETCswap connector for Ethereum Classic trading via Gateway.

**Architecture:**
```
Telegram → Condor Bot → Hummingbot Backend API → Trading Bots
                     ↘ Gateway → ETCswap DEX
```

## Tech Stack

**Framework & Runtime:**
- Python: 3.12+
- python-telegram-bot: Bot framework with job-queue
- hummingbot-api-client: API client library
- pydantic-ai: AI/LLM integration

**Data & Visualization:**
- Pandas: Data manipulation
- Plotly + Kaleido: Chart generation

**Market Data:**
- geckoterminal-py: DEX pool data

## Quick Start

```bash
# Clone and setup
git clone https://github.com/etcswap/hummingbot-condor.git
cd hummingbot-condor
git checkout etcswap

# Option 1: Local Python
make install     # Interactive setup + conda environment
make run         # Start the bot

# Option 2: Docker
make setup       # Interactive configuration
make deploy      # Start with Docker Compose
```

> **Note:** This uses the ETCswap fork. Once merged upstream, use `https://github.com/hummingbot/condor.git`.

## Core Architecture

### Project Structure

```
hummingbot-condor/
├── main.py                     # Application entry point
├── config_manager.py           # Unified config (servers, users, permissions)
├── handlers/                   # Telegram command handlers
│   ├── __init__.py             # Command registration
│   ├── portfolio.py            # /portfolio command
│   ├── bots/                   # /bots - Bot monitoring
│   ├── cex/                    # /trade - CEX trading
│   │   ├── trade.py            # Order placement
│   │   ├── orders.py           # Order management
│   │   └── positions.py        # Position tracking
│   ├── dex/                    # /swap, /lp - DEX trading
│   │   ├── swap.py             # Token swaps
│   │   ├── liquidity.py        # LP positions
│   │   └── pools.py            # Pool discovery
│   ├── config/                 # /config - Configuration
│   │   ├── servers.py          # API server management
│   │   ├── api_keys.py         # Exchange credentials
│   │   └── gateway/            # Gateway configuration
│   ├── routines/               # /routines - Automation
│   └── admin/                  # Admin panel
├── routines/                   # User automation scripts
│   ├── arb_check.py            # Arbitrage monitoring
│   ├── lp_monitor.py           # LP position monitoring
│   ├── lp_tpsl.py              # LP take-profit/stop-loss
│   └── price_monitor.py        # Price alerts
├── utils/                      # Utilities
│   ├── auth.py                 # @restricted, @admin_required
│   ├── portfolio_graphs.py     # Chart generation
│   └── telegram_formatters.py  # Message formatting
└── flows/                      # Flow documentation
```

### Key Files

| File | Purpose |
|------|---------|
| `main.py` | Application entry point, bot initialization |
| `config_manager.py` | Configuration management |
| `handlers/dex/swap.py` | DEX swap functionality |
| `handlers/dex/liquidity.py` | LP position management |
| `utils/auth.py` | Access control decorators |

## Commands

| Command | Description |
|---------|-------------|
| `/portfolio` | Portfolio dashboard with PNL indicators |
| `/bots` | All active bots with status |
| `/trade` | CEX trading (spot & perpetual) |
| `/swap` | DEX swaps via Gateway |
| `/lp` | DEX liquidity pool management |
| `/routines` | Automation scripts |
| `/config` | Configuration menu |

## ETCswap Integration

### How ETCswap Works in Condor

ETCswap is accessed through the `/swap` and `/lp` commands, which communicate with Gateway via the Backend API.

### Using /swap for ETCswap

1. Type `/swap` in Telegram
2. Select "Gateway Balances" to check ETC wallet
3. Select "Swap Quote" to get a price
4. Select "Execute Swap" to trade

### Using /lp for ETCswap

1. Type `/lp` in Telegram
2. Select "LP Positions" to view positions
3. Select "Pool Info" for pool details
4. Manage positions (add, close, collect fees)

### Configuration for ETCswap

In `/config`:
1. Add Hummingbot API server (with Gateway enabled)
2. Ensure Gateway is configured with ETCswap connector
3. Add ETC wallet to Gateway

### Example Swap Flow

```
User: /swap
Bot: [Shows swap menu]

User: [Selects "Swap Quote"]
Bot: Enter connector (e.g., etcswap):

User: etcswap
Bot: Enter base token:

User: ETC
Bot: Enter quote token:

User: USC
Bot: Enter amount:

User: 1.0
Bot: [Shows quote with price, fees, etc.]
     [Execute] [Cancel]

User: [Clicks Execute]
Bot: Swap executed! TX: 0x...
```

### Key Tokens

| Token | Symbol | Address |
|-------|--------|---------|
| Wrapped ETC | WETC | `0x1953cab0E5bFa6D4a9BaD6E05fD46C1CC6527a5a` |
| Classic USD | USC | `0xDE093684c796204224BC081f937aa059D903c52a` |

## Configuration

### .env

```bash
TELEGRAM_TOKEN=your_bot_token
ADMIN_USER_ID=123456789
OPENAI_API_KEY=sk-...  # Optional, for AI features
```

### config.yml (auto-created)

```yaml
servers:
  main:
    host: localhost
    port: 8000
    username: admin
    password: admin
default_server: main
admin_id: 123456789
users: {}
```

## Security Model

- **Admin Whitelist** - Only `ADMIN_USER_ID` has initial access
- **Role-Based Access** - Admin, User, Pending, Blocked
- **@restricted Decorator** - Applied to all command handlers
- **Secret Masking** - Passwords hidden in UI

## Protected Files

Do not modify without explicit request:
- `main.py` - Application entry point
- `config_manager.py` - Configuration management
- `utils/auth.py` - Authentication decorators

## Coding Style

- Python 3.12+ type hints
- Async/await for handlers
- python-telegram-bot patterns
- Black formatter

```bash
# Format code
black .
isort .
```

## Validation Requirements

**Before Any Commit:**
```bash
# Format check
black --check .
isort --check-only .

# Test bot locally
make run
```

## Commit Format

```
<scope>: <description>

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Scopes:**
- `feat:` - New features
- `fix:` - Bug fixes
- `handler:` - Handler changes
- `docs:` - Documentation

**Examples:**
```
feat: add ETCswap to swap connector list
handler: add ETC network selection in /swap
docs: add ETCswap trading guide
```

## Common Tasks

### Adding New DEX Connector

1. Update `handlers/dex/swap.py` connector list
2. Update `handlers/dex/liquidity.py` if LP supported
3. Add to documentation
4. Test via Telegram

### Adding New Routine

1. Create script in `routines/`
2. Inherit from `BaseRoutine`
3. Define Pydantic config model
4. Script auto-discovered by `/routines`

## Ecosystem Context

This is part of the ETCswap/Hummingbot integration:

- **hummingbot-condor** (THIS PROJECT): Telegram bot interface
- **hummingbot-api**: REST API backend
- **hummingbot-mcp**: MCP server for AI assistants
- **hummingbot**: Python trading bot
- **hummingbot-gateway**: TypeScript DEX middleware (ETCswap connector)

**Related Repos:**
- `hummingbot-api` - Backend API this bot connects to
- `hummingbot-gateway` - Gateway for DEX trading

**Branch:** This ETCswap integration is on the `etcswap` branch until merged upstream.

---

Last updated: 2025-01-20
