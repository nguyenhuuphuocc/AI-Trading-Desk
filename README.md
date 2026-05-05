# AI Trading Chatbot Platform

Full-stack trading assistant with a FastAPI backend, React/Tailwind dashboard, live market data adapters, strategy engine, risk management, WebSocket updates, paper trading, alerts, and backtesting.

This is research software, not financial advice. Trade execution is intentionally paper-mode only unless you add broker execution controls.

## Project Structure

```text
backend/
  app/
    api/              FastAPI routes and WebSocket endpoints
    services/         market data, chatbot, alerts, paper trading
    db_models.py      SQLAlchemy persistence models
    main.py           API entrypoint
  tests/
frontend/
  src/
    components/      dashboard, chart, chat, signal, backtest UI
    services/        API client
models/              optional ML predictor interface
strategies/          indicators, ICT concepts, strategy engine, backtester
```

## Features

- Conversational trading chatbot via `/api/chat`
- Real-time and historical crypto data through Kraken/Binance-compatible `ccxt`
- Stock data through Alpaca when keys are set, with Yahoo Finance fallback
- Multi-timeframe support: `1m`, `5m`, `15m`, `1h`, `4h`, `1d`
- ICT-inspired BOS, FVG, liquidity, supply/demand zones
- RSI, MACD, EMA, ATR, and volume analysis
- Position sizing, daily loss limit metadata, 1:2 minimum risk/reward
- Backtesting with equity curve, win rate, max drawdown, trade history
- PostgreSQL-ready SQLAlchemy models with local SQLite fallback
- WebSocket quote and signal streams
- Telegram/email alert hooks
- React + Tailwind dashboard with TradingView widget and Chart.js visuals

## Backend Setup

```powershell
cd C:\Users\nguye\Documents\Codex\2026-05-05\you-are-a-senior-ai-engineer
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r backend\requirements.txt
Copy-Item .env.example .env
python -m uvicorn backend.app.main:app --reload --host 127.0.0.1 --port 8000
```

For PostgreSQL:

```powershell
docker compose up -d postgres
```

Keep `DATABASE_URL=postgresql+psycopg://postgres:postgres@localhost:5432/tradingbot` in `.env`. If you remove `DATABASE_URL`, the API uses local SQLite.

## Frontend Setup

```powershell
cd frontend
npm install
Copy-Item .env.example .env
npm run dev
```

Open [http://127.0.0.1:5173](http://127.0.0.1:5173).

## API Examples

```bash
curl -X POST http://127.0.0.1:8000/api/analyze \
  -H "Content-Type: application/json" \
  -d "{\"symbol\":\"BTC/USDT\",\"asset_type\":\"crypto\",\"timeframe\":\"15m\"}"
```

```bash
curl -X POST http://127.0.0.1:8000/api/chat \
  -H "Content-Type: application/json" \
  -d "{\"message\":\"Analyze ETH and give me a high probability trade\"}"
```

```bash
curl -X POST http://127.0.0.1:8000/api/backtest \
  -H "Content-Type: application/json" \
  -d "{\"symbol\":\"BTC/USDT\",\"asset_type\":\"crypto\",\"timeframe\":\"1h\",\"limit\":650}"
```

## Environment Variables

Use `.env.example` as the template.

- `OPENAI_API_KEY`: enables LLM-written chat responses. Without it, the bot returns deterministic strategy explanations.
- `OPENAI_MODEL`: model used by the chatbot path.
- `ALPACA_API_KEY` / `ALPACA_API_SECRET`: enables Alpaca stock bars. Without them, Yahoo Finance is used.
- `DEFAULT_CRYPTO_EXCHANGE`: defaults to `kraken`; Binance and Binance US are included as failover options.
- `DATABASE_URL`: PostgreSQL URL. Local SQLite fallback is used when omitted.
- `MAX_RISK_PER_TRADE`: default `0.01`.
- `MAX_DAILY_LOSS`: default `0.03`.
- `MINIMUM_RR`: default `2`.

## Deployment

Backend on Render or AWS:

- Build: `pip install -r backend/requirements.txt`
- Start: `python -m uvicorn backend.app.main:app --host 0.0.0.0 --port $PORT`
- Add environment variables from `.env.example`
- Provision PostgreSQL and set `DATABASE_URL`

Frontend on Vercel:

- Root directory: `frontend`
- Build command: `npm run build`
- Output directory: `dist`
- Set `VITE_API_URL` to the deployed backend URL
- Set `VITE_WS_URL` to the deployed backend WebSocket URL

## Production Notes

- Add authentication before exposing user chats, paper orders, or alerts publicly.
- Use a migration tool such as Alembic before schema changes in production.
- Replace the compact `models/PricePredictor` with a trained/persisted LSTM or transformer model if ML prediction becomes a primary signal.
- Add broker-side kill switches before enabling real auto-trading.
