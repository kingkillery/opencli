# 新浪财经 (Sina Finance)

**Mode**: 🌐 Public / 🔐 Browser · **Domain**: `finance.sina.com.cn`

## Commands

| Command | Description | Mode |
|---------|-------------|------|
| `opencli sinafinance news` | Sina Finance 7×24 real-time news flash | 🌐 Public |
| `opencli sinafinance rolling-news` | Sina Finance rolling news feed | 🔐 Browser |
| `opencli sinafinance stock` | Sina Finance market data (A-shares / HK stocks / US stocks) | 🌐 Public |

## Usage Examples

### news - 7×24 real-time news flash

```bash
# Latest financial news
opencli sinafinance news --limit 20

# Filter by type
opencli sinafinance news --type 1   # A-shares
opencli sinafinance news --type 2   # Macro
opencli sinafinance news --type 6   # International

# JSON output
opencli sinafinance news -f json
```

### rolling-news - rolling news feed

```bash
# Rolling news feed
opencli sinafinance rolling-news

# JSON output
opencli sinafinance rolling-news -f json
```

### stock - stock market data

```bash
# Search and view A-share stock
opencli sinafinance stock 贵州茅台 --market cn

# Search and view HK stock
opencli sinafinance stock 腾讯控股 --market hk

# Search and view US stock
opencli sinafinance stock aapl --market us

# Auto-detect market (searches cn, hk, us in order)
opencli sinafinance stock 招商证券

# JSON output
opencli sinafinance stock 贵州茅台 -f json
```

## Options

### news

| Option | Description |
|--------|-------------|
| `--limit` | Max results, up to 50 (default: 20) |
| `--type` | News type: `0`=All, `1`=A-shares, `2`=Macro, `3`=Companies, `4`=Data, `5`=Markets, `6`=International, `7`=Views, `8`=Central Bank, `9`=Other |

### stock

| Option | Description |
|--------|-------------|
| `--market` | Market: `cn`, `hk`, `us`, `auto` (default: auto). When `auto`, searches in cn, hk, us order |

## Prerequisites

- `news` & `stock`: No browser required — uses public API
- `rolling-news`: Chrome running and **logged into** `finance.sina.com.cn`
- For `rolling-news`: [Browser Bridge extension](/guide/browser-bridge) installed

## Notes

- `news` and `stock` use public APIs — no browser or login needed
- `stock` supports Chinese names, Chinese codes, and ticker symbols; auto-detects market
- Market priority for auto-detection: cn (A-shares) → hk (HK stocks) → us (US stocks)
- US stock `High`/`Low` columns show 52-week range; A-shares/HK stocks show today's range
