# Hyperliquid Trading (perp trading) Bot

Hyperliquid trading(Perp trading) Bot with Rust is an experimental trading system built with
[`hyperliquid_rust_sdk`](https://github.com/hyperliquid-dex/hyperliquid-rust-sdk). It manages multiple
markets on the Hyperliquid exchange and places trades based on signals from
user-selected indicators.

The repository currently ships a React/TS UI and actix server, follow the getting started steps.

## Features

- Connect to Hyperliquid mainnet, testnet or localhost.
- Manage several markets concurrently with configurable margin allocation.
- Customisable strategy (risk, style, stance).
- Indicator engine where each indicator is bound to a timeframe.
- Asynchronous design using `tokio` and `flume` channels.

## Getting started

After cloning the repo

1. Install a recent Rust toolchain.
2. Create a `.env` file in the root directory`:

   ```env
   PRIVATE_KEY=<your API private key> -> https://app.hyperliquid.xyz/API
   AGENT_KEY=<optional agent api public key>
   WALLET=<public wallet address>
   ```

3. Run the app:

   ```bash
   ./run.sh
   ```

## Strategy

The bot uses `CustomStrategy` (see `src/strategy.rs`). It combines indicators
such as RSI, StochRSI, EMA crosses, ADX and ATR. Risk level (`Low`, `Normal`,
`High`), trading style (`Scalp` or `Swing`) and market stance (`Bull`, `Bear` or
`Neutral`) can be set. Signals are generated when multiple indicator conditions
agree—for example an oversold RSI with a bullish StochRSI crossover may trigger a
long trade.

## Indicators

Indicators are activated with `(IndicatorKind, TimeFrame)` pairs. Available kinds
include:

- `Rsi(u32)`
- `SmaOnRsi { periods, smoothing_length }`
- `StochRsi { periods, k_smoothing, d_smoothing }`
- `Adx { periods, di_length }`
- `Atr(u32)`
- `Ema(u32)`
- `EmaCross { short, long }`
- `Sma(u32)`

Each pair is wrapped in an `Entry` together with an `EditType` (`Add`, `Remove` or
`Toggle`). The snippet below (from `enginetest.rs`) shows how a market can be
created with a custom indicator configuration:

```rust
let config = vec![
    (IndicatorKind::Rsi(12), TimeFrame::Min1),
    (IndicatorKind::EmaCross { short: 21, long: 200 }, TimeFrame::Day1),
];

let market = AddMarketInfo {
    asset: "BTC".to_string(),
    margin_alloc: MarginAllocation::Alloc(0.1),
    trade_params,
    config: Some(config),
};
```

## Project structure

- `src/bot.rs` – orchestrates markets and keeps margin in sync.
- `src/market.rs` – handles a single market: data feed, signal engine and order
  execution.
- `src/signal/` – indicator trackers and strategy logic.
- `src/executor.rs` – sends orders via the Hyperliquid API.
- 'src/strategy.ra' - Implements strategies followed by the signal engine.
- `src/trade_setup.rs` – trading parameters and trade metadata.
- `config.toml` – example strategy configuration.

Supported trading pairs can be found in `src/assets.rs` (`MARKETS`).

## Disclaimer

This code is experimental and not audited. Use at your own risk when trading on
live markets.

## Contact

- **Telegram:** [https://t.me/Kei4650](https://t.me/Kei4650)  
- **Twitter:** [https://x.com/kei_4650](https://x.com/kei_4650)
