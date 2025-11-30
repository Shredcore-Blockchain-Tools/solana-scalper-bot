# shredcore-scalper-bot

A high-performance Solana high-frequency scalping bot written in Rust for maximum efficiency. This bot automatically identifies and trades short-term price movements on PumpFun and PumpSwap platforms, re-entering positions when market conditions are favorable.

## Why Rust?

This bot is written in Rust, a systems programming language known for its exceptional performance, memory safety, and low latency. Rust's zero-cost abstractions and efficient execution make it ideal for high-frequency trading where every millisecond counts. The bot can process market events and execute trades faster than bots written in interpreted languages, giving you a competitive edge in the fast-paced world of scalping.

## Performance

This is one of the **speediest bot on the market**. With optimal server, RPC, and gRPC provider configuration, the bot can achieve **0 to 3 blocks landing speed** after signal detection. This exceptional speed is critical for high-frequency scalping where rapid entry and exit execution directly impacts profitability.

## Durable Nonce Technology

This bot uses **Durable Nonce** technology, which is essential for its operation. Here's why:

When trading at high speeds, the bot sends the same transaction to multiple SWQoS (Solana Quality of Service) providers simultaneously - including Jito, Nozomi (Temporal), and Astralane. This "spam sending" strategy dramatically improves transaction inclusion speed and success rates by ensuring your transaction reaches validators through multiple paths.

However, without durable nonces, sending the same transaction to multiple providers could result in duplicate executions if multiple providers include it in the same block. Durable nonces solve this by ensuring each transaction can only be executed once, even if it's submitted through multiple channels. This allows the bot to safely spam transactions to all available SWQoS providers for maximum speed and inclusion probability, while preventing accidental double-spends.

## Supported Platforms

- **PumpFun** - The original bonding curve platform
- **PumpSwap** - After migration from bonding curves
... Many more to come

## Features

### High-Frequency Scalping

- **Automatic Re-entry**: Closed positions stay in watchlist and can be automatically re-entered when dip/flow conditions are favorable
- **Scalping Mode**: Enables aggressive re-entry behavior for short-term profit capture
- **Entry Loop**: Continuously monitors market conditions and enters trades when opportunities arise
- **Buy Throttling**: Prevents rapid re-entry on the same token with configurable cooldown periods

### Market Analysis & Scoring

- **Interest Scoring**: Calculates opportunity scores based on volume, trading activity, and market flow
- **Volume Filters**: Requires minimum trading volume across multiple time windows (5s, 2m, 5m, etc.)
- **Trade Activity Filters**: Filters out dead coins by requiring minimum trade counts
- **Data Freshness**: Rejects entries if market data is stale
- **Rug Protection**: Detects and avoids tokens that are rapidly declining

### Entry Strategy

- **Dip Entry Gates**: Only enters when price is in a favorable dip range from recent highs
- **Buy Flow Requirements**: Ensures sufficient buy pressure (minimum buy share percentage) before entering
- **Entry Dip Weights**: Configurable weighting across multiple timeframes (1s, 5s, 15s, 30s, 2m, 5m, etc.) for precise entry timing
- **Rebound Confirmation**: Optional feature to wait for price/flow recovery before entering (anti-knife-catch protection)
- **Global Drawdown Filter**: Prevents entering tokens that are far down from all-time high

### Risk Management

- **Stop Loss**: Automatically sell if your position drops below a configured loss percentage
- **Take Profit Levels**: Set multiple profit targets with partial sell percentages
- **Dynamic Trailing Stop Loss (DTSL)**: As your profit increases, the stop loss floor automatically raises to lock in gains
- **Time-Based Exits**: Force exits after maximum hold time, negative PnL duration, or minimum profit target duration
- **Position Limits**: Control maximum concurrent positions
- **Portfolio Exposure Limits**: Limit total capital deployed across all positions

### Advanced Trading Features

- **Dollar Cost Averaging (DCA)**: Automatically add to losing positions to average down entry price
  - Configurable DCA triggers based on loss percentage and duration
  - Dip and flow requirements for DCA entries
  - Rebound confirmation for safer DCA execution
- **Transaction Retries**: Automatic retry logic for failed transactions with exponential backoff
- **Blacklist**: Permanently block specific token mints from trading

### Execution Features

- **SWQoS Integration**: Simultaneously sends transactions to multiple providers (Jito, Nozomi, Astralane) for maximum inclusion speed
- **Priority Fees**: Configurable fees to encourage faster validator inclusion
- **High Slippage Tolerance**: Configured for aggressive entry/exit to ensure trades execute
- **Real-Time Market Data**: Uses gRPC (Yellowstone) or WebSocket streams for instant market updates

## Setup and Installation

### Prerequisites

- Solana CLI tools (for nonce setup)
- A Solana wallet with SOL for trading
- A license key
- RPC endpoint (high-performance private RPC recommended)
- gRPC endpoint (Yellowstone gRPC) or WebSocket endpoint

### Quick Start

1. **Configure the bot**:
   ```bash
   ./start.sh
   ```
   
   The interactive setup script will guide you through:
   - License key entry
   - RPC and gRPC/WebSocket URL configuration
   - Wallet private key (Base58 encoded)

2. **Setup Durable Nonce**:
   The setup script automatically runs `setup_nonce.sh` if no nonce account exists. This creates a durable nonce account that's required for safe multi-provider transaction sending.

3. **Configure Scalping Settings**:
   
   Edit `config.toml` to enable scalping:
   - Set `ALLOW_REENTER = true` to allow re-entering closed positions
   - Set `ENABLE_SCALPING = true` to enable automatic re-entry based on market conditions
   - Adjust `ENTRY_INTEREST_SCORE_MIN` to filter opportunities
   - Configure `BUY_THROTTLE_SECS` to control re-entry frequency

4. **Launch the bot**:
   ```bash
   ./start.sh
   ```
   
   Or if you've already configured:
   ```bash
   ./rust-scalper
   ```

### Manual Configuration

1. Copy the example config:
   ```bash
   cp config.example.toml config.toml
   ```

2. Edit `config.toml` with your settings:
   - `LICENSE_KEY`: Your license key
   - `RPC_URL`: Your Solana RPC endpoint
   - `GRPC_URL`: Your Yellowstone gRPC endpoint (if using gRPC)
   - `WALLET_PRIVATE_KEY_B58`: Your wallet private key in Base58 format
   - Enable scalping: `ALLOW_REENTER = true` and `ENABLE_SCALPING = true`
   - Adjust trading parameters (buy amounts, stop loss, take profit, etc.)
   - Configure scoring and entry filters

3. Setup durable nonce:
   ```bash
   ./setup_nonce.sh
   ```

4. Run the bot:
   ```bash
   ./rust-scalper
   ```

### Configuration File

The `config.toml` file contains all bot settings organized into sections:

- `[config]`: Connection settings, wallet, license, and nonce configuration
- `[trading]`: Trading parameters, risk management, and scalping settings
- `[scoring]`: Market analysis filters, entry gates, and dip weights

See `config.example.toml` for detailed comments on each setting.

## Usage

### Normal Operation

Simply run `./start.sh` or `./rust-scalper` and the bot will:
1. Connect to market data streams
2. Monitor all trading activity on PumpFun and PumpSwap
3. Score opportunities based on volume, flow, and market conditions
4. Automatically execute buy orders when favorable conditions are detected
5. Manage positions with your configured risk management rules
6. Re-enter closed positions when scalping conditions are met (if enabled)
7. Execute sells based on stop loss, take profit, or time-based rules

### Command Line Interface

You can also manually trigger trades:

**Buy a specific token**:
```bash
./rust-scalper --buy <MINT_ADDRESS> [--platform PUMP_FUN|PUMP_SWAP]
```

**Sell a position**:
```bash
./rust-scalper --sell <MINT_ADDRESS>
```

### Logs

Logs are saved to `.logs/solana-trading-bot.log` by default (can be disabled in config). Monitor this file to track bot activity, trades, market analysis, and any issues.

## Configuration Tips

### Scalping Settings

- **Enable Scalping**: Set `ALLOW_REENTER = true` and `ENABLE_SCALPING = true` to enable automatic re-entry
- **Entry Interest Score**: Higher `ENTRY_INTEREST_SCORE_MIN` values filter out lower-quality opportunities
- **Buy Throttle**: Lower `BUY_THROTTLE_SECS` allows more frequent entries but increases risk

### Entry Filters

- **Volume Requirements**: Adjust `MIN_VOL_*` settings to ensure sufficient liquidity
- **Trade Activity**: Set `MIN_TRADES_30S` to filter out dead coins
- **Dip Range**: Configure `ENTRY_DIP_MIN_PCT` and `ENTRY_DIP_MAX_PCT` for optimal entry timing
- **Buy Flow**: Set `MIN_BUY_SHARE_FOR_ENTRY` to require stronger buy pressure

### DCA Configuration

- **DCA Triggers**: Configure `DCA_LOSS_TRIGGER_UNDERWATER_PCT` and `DCA_LOSS_TIMER_SECONDS` for loss-based DCA
- **Dip Requirements**: Set `DCA_DIP_MIN_PCT` and `DCA_DIP_MAX_PCT` to control when DCA is allowed
- **Flow Requirements**: Configure `DCA_MIN_BUY_SHARE` to ensure buy flow before DCA

## Important Notes

- **Durable Nonce is Required**: The bot requires a durable nonce account for safe operation with multiple SWQoS providers. The setup script handles this automatically.

- **High-Performance RPC Recommended**: For best results, use a private, high-performance RPC endpoint. Public RPCs may have rate limits and higher latency.

- **Wallet Security**: Never share your `WALLET_PRIVATE_KEY_B58`. Keep your `config.toml` file secure and never commit it to version control.

- **Start with Small Amounts**: When first using the bot, start with small `BUY_AMOUNT_SOL` values to test your configuration.

- **Simulation Mode**: Use `SIMULATE = true` in your config to test without risking real SOL.

- **Scalping is Aggressive**: Scalping mode can result in many trades. Monitor your portfolio exposure and transaction fees.

## Troubleshooting

- **"Durable nonce file not found"**: Run `./setup_nonce.sh` manually
- **"License validation failed"**: Check your `LICENSE_KEY` in config.toml
- **"Failed to create trade-stream client"**: Verify your `GRPC_URL` or `WS_URL` is correct
- **No trades executing**: Check your entry filters aren't too restrictive
- **Too many trades**: Increase `ENTRY_INTEREST_SCORE_MIN` or adjust entry filters
- **Transactions failing**: Check your wallet has sufficient SOL for trades and fees

## Support

For issues, questions, or feature requests, please contact support through your license provider.

