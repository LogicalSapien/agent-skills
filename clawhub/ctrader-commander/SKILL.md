---
name: ctrader-commander
description: Place and manage cTrader orders (market, limit, stop), check open positions, fetch live quotes and OHLC candles, and query account balance and equity via a local HTTP proxy. No credentials or token required at call time.
homepage: https://github.com/LogicalSapien/agent-skills/tree/main/clawhub/ctrader-commander
metadata: {"openclaw": {"emoji": "\ud83d\udcc8", "requires": {"bins": ["curl"]}, "homepage": "https://github.com/LogicalSapien/agent-skills/tree/main/clawhub/ctrader-commander"}}
---

# cTrader Commander

Use when the user wants to place trades, check positions or balance, get live prices, fetch candles, or manage orders on a cTrader account.

All calls go to `http://localhost:9009` — credentials live in `.env` on the server, never passed by callers.

> **Proxy repo:** https://github.com/LogicalSapien/ctrader-openapi-proxy
> Clone it, add your `.env`, and run `make run` to start the proxy before using this skill.

Full reference: `{baseDir}/endpoints.md`

## Check proxy is running

```bash
curl -s "http://localhost:9009/get-data?command=ProtoOAVersionReq"
```

If it fails, start the proxy: `cd ~/ctrader-openapi-proxy && make run`

## Find symbol IDs (do this first)

Symbol IDs are broker-specific — look them up before placing orders or fetching data:

```bash
curl -s "http://localhost:9009/get-data?command=ProtoOASymbolsListReq"
```

Returns `symbol[]` with `symbolId` and `symbolName`. Note the ID for your instrument.

## Place a market order

```bash
curl -s -X POST http://localhost:9009/api/market-order \
  -H "Content-Type: application/json" \
  -d '{"symbolId": 158, "orderType": "MARKET", "tradeSide": "BUY", "volume": 1000}'
```

Volume is in **units**: `1000` = 0.01 lot · `10000` = 0.1 lot · `100000` = 1 lot.
Add `"relativeStopLoss": 200, "relativeTakeProfit": 350` (pips, market orders only).

## Place a limit or stop order

```bash
curl -s -X POST http://localhost:9009/api/market-order \
  -H "Content-Type: application/json" \
  -d '{"symbolId": 158, "orderType": "LIMIT", "tradeSide": "BUY", "volume": 1000, "price": 0.62500}'
```

`orderType`: `MARKET` · `LIMIT` · `STOP` — `tradeSide`: `BUY` · `SELL`

## Get OHLC candles

```bash
NOW_MS=$(python3 -c "import time; print(int(time.time()*1000))")
FROM_MS=$(python3 -c "import time; print(int(time.time()*1000) - 3600000)")
curl -s -X POST http://localhost:9009/api/trendbars \
  -H "Content-Type: application/json" \
  -d "{\"fromTimestamp\": $FROM_MS, \"toTimestamp\": $NOW_MS, \"period\": \"M5\", \"symbolId\": 158}"
```

Periods: `M1 M2 M3 M4 M5 M10 M15 M30 H1 H4 H12 D1 W1 MN1`

## Get live quote (tick data)

```bash
curl -s -X POST http://localhost:9009/api/live-quote \
  -H "Content-Type: application/json" \
  -d '{"symbolId": 158, "quoteType": "BID", "timeDeltaInSeconds": 60}'
```

`quoteType`: `BID` or `ASK`

## Open positions and pending orders

```bash
curl -s "http://localhost:9009/get-data?command=ProtoOAReconcileReq"
```

## Close a position

```bash
curl -s "http://localhost:9009/get-data?command=ClosePosition%20123456%201000"
# ClosePosition <positionId> <volumeInUnits>
```

## Cancel a pending order

```bash
curl -s "http://localhost:9009/get-data?command=CancelOrder%20789"
```

## Account info (balance, equity, leverage)

```bash
curl -s "http://localhost:9009/get-data?command=ProtoOATraderReq"
```


   curl -s "http://localhost:9009/get-data?command=ProtoOASymbolsListReq" | python3 -c "
   import sys, json
   data = json.load(sys.stdin)
   [print(s['symbolId'], s['symbolName']) for s in data.get('symbol', []) if 'EURUSD' in s['symbolName']]
   "
   ```
2. Check your account details:
   ```bash
   curl -s "http://localhost:9009/get-data?command=ProtoOATraderReq"
   ```
3. Place a market buy:
   ```bash
   curl -s -X POST http://localhost:9009/api/market-order \
     -H "Content-Type: application/json" \
     -d '{"symbolId": 1, "orderType": "MARKET", "tradeSide": "BUY", "volume": 1000}'
   ```
4. Check open positions:
   ```bash
   curl -s "http://localhost:9009/get-data?command=ProtoOAReconcileReq"
   ```
