# SwingAI Revolut X Bot

Cloudflare Worker z pełną logiką AI do handlu kryptowalutami na **Revolut X** (Ed25519 auth). Dane rynkowe z Gate.io (publiczne API).

## Strategia

- **Multi-TF swing trading**: Daily + 4H + 1H
- **Wskaźniki**: RSI(14), MACD, Bollinger Bands(20), EMA50/200, VWAP, S/R pivoty, RSI dywergencje
- **AI ensemble**: Naive Bayes + Gradient Boosting + Q-Learning
- **Risk**: Kelly criterion, circuit breaker (−15% drawdown → blokada BUY 24h), portfolio heat <10%, daily loss limit −5%
- **Formacje**: Hammer, Engulfing, Morning Star, Doji, Piercing Line, Three White Soldiers, Shooting Star, Bearish Engulfing
- **Pary**: BTC, ETH, SOL, XRP, DOGE, ADA, AVAX, LINK (vs USDC)

## Dashboard

Otwarty w przeglądarce: [https://tomekfalek-cyber.github.io/swingai-revolut/](https://tomekfalek-cyber.github.io/swingai-revolut/)

Folder `dashboard/index.html` — pełna wersja do GitHub Pages (połącz z własnym Worker URL w konfiguracji).

## Setup — krok po kroku

### 1. Wygeneruj klucz Ed25519

```bash
# Linux/Mac:
openssl genpkey -algorithm ed25519 -out private_key.pem
openssl pkey -in private_key.pem -pubout -out public_key.pem

# Lub online (tylko do testów): generuj lokalnie!
```

Zawartość `public_key.pem` wgraj w panelu Revolut X (Settings → API Keys).

### 2. Utwórz KV Namespace w Cloudflare

W panelu Cloudflare Workers:
- **Storage → KV** → Create Namespace: `SWINGAI_REVOLUT_KV`
- Skopiuj **Namespace ID**

### 3. Wdróż Worker

W Cloudflare Workers → Create Worker:
1. Wklej zawartość `worker.js`
2. Przejdź do **Settings → Variables → KV Namespace Bindings**:
   - Variable name: `SWINGAI_REVOLUT_KV`
   - Namespace: wybierz `SWINGAI_REVOLUT_KV`
3. Zapisz i wdróż
4. Skopiuj URL Workera (np. `https://swingai-revolut.TWOJ-NICK.workers.dev`)

### 4. GitHub Pages (Dashboard)

1. W ustawieniach repo: **Settings → Pages → Source: main branch / folder: /dashboard**
2. Dashboard dostępny na: `https://tomekfalek-cyber.github.io/swingai-revolut/`

### 5. Konfiguracja Bota

Otwórz dashboard → ⚙️ Konfiguracja:
- **Worker URL**: `https://swingai-revolut.TWOJ-NICK.workers.dev`
- **Auth token**: `swingai-revolut-2024` (domyślny)
- **Revolut X API Key**: klucz z panelu Revolut X
- **Private Key PEM**: zawartość `private_key.pem` (cały plik)
- **Tryb**: Paper (test) lub Live

### 6. Uruchom

W dashboardzie kliknij **START PAPER** (najpierw przetestuj!) lub **START LIVE**.

## API Endpoints (Worker)

| Endpoint | Opis |
|----------|------|
| `GET /` | Redirect do GitHub Pages dashboard |
| `GET /status?auth=TOKEN` | Stan bota (JSON) |
| `GET /start-paper?auth=TOKEN` | Uruchom paper mode |
| `GET /start-live?auth=TOKEN&key=KEY&priv=PEM_B64&tp=12&sl=5...` | Uruchom live mode |
| `GET /stop?auth=TOKEN` | Zatrzymaj bota |
| `GET /run?auth=TOKEN` | Wymuś jeden cykl |
| `GET /balance?auth=TOKEN` | Saldo USDC z Revolut X |

## Zmienne środowiskowe (opcjonalne)

| Zmienna | Opis | Domyślna |
|---------|------|---------|
| `AUTH_SECRET` | Token autoryzacyjny | `swingai-revolut-2024` |

## Różnice vs Bot MEXC

| Cecha | MEXC Bot | Revolut X Bot |
|-------|----------|---------------|
| Auth | HMAC-SHA256 | Ed25519 |
| API key | `apiKey` + `apiSecret` | `revxApiKey` + `revxPrivKey` (PEM) |
| Pair format | `BTCUSDC` | `BTC/USDC` |
| KV namespace | `SWINGAI_KV` | `SWINGAI_REVOLUT_KV` |
| Live mode | `cfg.mode = 'mexc'` | `cfg.mode = 'live'` |

## Ostrzeżenie

Ten bot jest oprogramowaniem eksperymentalnym. Handel kryptowalutami wiąże się z ryzykiem utraty całości kapitału. Używaj na własną odpowiedzialność. Najpierw testuj w trybie paper.