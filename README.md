# 💰 Personal Finance Dashboard

> Clean, private personal finance dashboard — your data never leaves your machine.

A Python-based personal finance dashboard that turns CSV-stored transactions into interactive charts and indicators. Runs locally via Docker, with **no external APIs and no telemetry**.

> 🤖 Looking for the AI-powered version that adds automatic insights from a local LLM? See **[ai-powered-financial-dashboard](https://github.com/cassianorcarneiro/ai-powered-financial-dashboard)**.

<p align="center">
  <img alt="Stack" src="https://img.shields.io/badge/Stack-Dash%20%2B%20Plotly-blue?style=for-the-badge">
  <img alt="License" src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-ready-2496ED?style=for-the-badge&logo=docker&logoColor=white">
</p>

---

## 📦 Features

- 📊 **Interactive charts** — spending by category, payment method, monthly trends, installments starting/finishing
- 💳 **Payment method management** — handle credit cards (with statement close / payment days) and debit accounts
- 📅 **Installment-aware** — splits multi-installment purchases into per-month records with automatic payment date computation
- 🔒 **Privacy-first** — all data stays on your machine; no external APIs, no telemetry
- 💾 **Persistent data** — your CSVs live on the host filesystem, untouched by container restarts

---

## 🏗️ Architecture

```
┌─────────────────────────┐
│   financial-dashboard   │
│   (Dash + Plotly)       │
│   port 8050             │
└────────────┬────────────┘
             │
             ▼
       ┌───────────┐
       │  ./data/  │   ← CSVs persisted on the host
       └───────────┘
```

Single container; your data lives in `./data/` and is mounted into the container.

---

## 📋 Prerequisites

- **Docker Desktop** (Windows/macOS) or **Docker Engine + Compose plugin** (Linux)
  - Verify: `docker --version` and `docker compose version`
- **~2 GB free disk** (for the Python image and dependencies)
- **Internet** for the first run (pulling the Docker image)

> Don't have Docker? Get it at [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop).

---

## 🚀 Quick start

### 1. Clone the repository

```bash
git clone https://github.com/cassianorcarneiro/personal-finance-dashboard.git
cd personal-finance-dashboard
```

### 2. Prepare your data folder

```
.
├── app.py
├── config.py
├── loading.html
├── requirements.txt
├── docker-compose.yaml
├── Dockerfile
├── .dockerignore
├── .env.example
└── data/
    ├── data.csv
    ├── categories.csv
    └── payments_methods.csv
```

The three CSV files inside `data/` **must exist**, even if empty. The app reads them at startup. Minimal headers expected:

| File | Required columns |
|------|------------------|
| `data.csv` | `Transaction Date;Payment Date;Label;Category;Amount;Installment;Payment Method;Hash;Record Timestamp;Ignore Entry` |
| `categories.csv` | `Name` |
| `payments_methods.csv` | `Name;Close Date;Payment Date;Type` |

> Use `;` as separator and UTF-8-with-BOM encoding (Excel-friendly).

### 3. (Optional) Configure environment

```bash
cp .env.example .env
# edit .env to change the port or timezone
```

Defaults: port `8050`, timezone `America/Sao_Paulo`.

### 4. Build and start

```bash
docker compose up -d --build
```

This will:

1. Build the dashboard image
2. Start the container

The first run takes a couple of minutes to install Python dependencies. Subsequent runs are fast.

Verify it's up:

```bash
docker compose ps
```

### 5. Open the dashboard

Visit **http://localhost:8050** in your browser.

---

## ⚙️ Configuration

All runtime configuration is done through environment variables, exposed via `.env`:

| Variable | Default | Purpose |
|----------|---------|---------|
| `DASHBOARD_PORT` | `8050` | Host port mapped to the dashboard |
| `TZ` | `America/Sao_Paulo` | Timezone for the "last update" timestamp |
| `REQUEST_PASSWORD` | `0` | Set to `1` to require basic auth (users defined in `config.py`) |

---

## 💾 Data persistence

| What | Where | Persisted? |
|------|-------|------------|
| Your CSVs | `./data/` (bind mount) | ✅ on the host filesystem |
| Container filesystem | inside the container | ❌ rebuilt each time |

Edit your CSVs directly with any tool (Excel, VS Code, `pandas`) — the dashboard re-reads them on every interaction.

To **stop and remove** the container (data stays):

```bash
docker compose down
```

To **completely reset**:

```bash
docker compose down -v
```

---

## 🛠️ Common operations

### View logs

```bash
docker compose logs -f dashboard
```

### Restart only the dashboard (after editing the code)

```bash
docker compose up -d --build dashboard
```

### Stop everything

```bash
docker compose down
```

### Start again

```bash
docker compose up -d
```

---

## 🔧 Troubleshooting

**🔴 Container keeps restarting**

Check the logs to see the actual error:

```bash
docker compose logs dashboard --tail 50
```

Most common cause: a Python package missing in `requirements.txt`. Add the missing package and rebuild:

```bash
docker compose up -d --build dashboard
```

**🔴 Port 8050 is already in use**

Set a different port in `.env`:

```bash
DASHBOARD_PORT=8080
```

Then `docker compose up -d` and visit http://localhost:8080.

**🔴 Empty charts / "No data available"**

Make sure your `data/data.csv` has rows within the date filter range. The default filter is the current calendar year — adjust the date inputs at the top of the page.

**🔴 CSV changes don't appear in the dashboard**

The file is read on each callback, but the date filter may be excluding new rows. Reset filters with the ↺ button.

**🔴 "No such file or directory" on startup**

The `data/` folder or one of the CSVs is missing. Create them with the headers shown in step 2.

---

## 📁 Project structure

```
.
├── app.py                  # Dash app, callbacks, layout
├── config.py               # Colors, fonts, db paths
├── loading.html            # Splash page opened on startup
├── requirements.txt        # Python dependencies
├── Dockerfile              # Image definition
├── docker-compose.yaml     # Single-service stack
├── .dockerignore           # Excludes .git, __pycache__, venv from the build context
├── .env.example            # Template for runtime configuration
└── data/                   # ← your CSVs live here (bind-mounted into the container)
    ├── data.csv
    ├── categories.csv
    └── payments_methods.csv
```

---

## 🔐 Privacy

This project is designed to keep your financial data on your machine:

- ✅ CSVs never leave the host filesystem
- ✅ The dashboard makes no outbound HTTP requests during normal operation
- ⚠️ The first run **does** require internet to pull the Docker image and install Python dependencies

After the initial setup, you can disconnect from the internet and the dashboard will keep working.

---

## 🛣️ Roadmap

- [ ] Excel import (in addition to CSV)
- [ ] Budget vs. actual tracking per category
- [ ] Recurring transaction detection
- [ ] Forecast next month's expenses based on installments already scheduled
- [ ] Multi-currency support
- [ ] Mobile-friendly layout

---

## 📜 License

MIT — see `LICENSE` file.

## 👤 Author

**Cassiano Ribeiro Carneiro** — [@cassianorcarneiro](https://github.com/cassianorcarneiro)

---

### 🤖 AI Assistance Disclosure

The codebase architecture, organizational structure, and stylistic formatting of this repository were refactored and optimized leveraging [Claude](https://www.anthropic.com/claude) by Anthropic. All core business logic and intellectual property remain the work of the repository authors and are governed by the project's license.

---

> *A clean, private financial dashboard — nothing more, nothing less.*
