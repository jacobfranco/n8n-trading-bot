# n8n DCA Trading Bot

A fully automated dollar-cost-averaging (DCA) trading workflow built with **n8n**, the **Interactive Brokers (IBKR) TWS API**, and **Postgres**.  
This system was designed to continuously monitor market prices for **KO (Coca-Cola)**, execute defined DCA entries, place take-profit (TP) orders, detect fills, and send notifications — all without manual intervention.

## How the System Works

The workflow implements a structured, rule-based DCA strategy around buying **10-share tranches** of KO and exiting them through a fixed take-profit increment.

### **Take-Profit Logic**

Every 10-share tranche (initial or DCA) is assigned a take-profit level at:

- **+0.09 above the entry price**

When the price reaches +0.09, the bot executes a TP sell for that tranche and records/logs the fill.

### **DCA Levels**

If the price moves against the position, the system automatically buys additional 10-share tranches at predefined drawdown levels:

- **DCA #1 at –0.60**
- **DCA #2 at –0.90**
- **DCA #3 at –1.20**

Each DCA step is executed only once per cycle, and each step creates its own corresponding +0.09 TP target.

### **Fallback Matching**

IBKR sometimes fills TP orders before the workflow detects them.  
To keep everything consistent, the bot:

- Reconciles positions through the **IBKR API**
- Compares fills against historical trades
- Performs fallback matching when fills occur outside n8n’s normal execution path

This ensures the system’s internal state reflects the actual brokerage account.

### **Notifications**

Every buy, TP fill, or reconciliation event triggers a **Gmail notification**, including:

- Action taken (DCA buy, TP sell, fallback match)
- Execution price
- Remaining position
- Which DCA or TP level triggered

### **Data Logging**

A Postgres database stores:

- All executed buys
- TP orders and fills
- Reconciliation events
- Active DCA step states

This makes the workflow resilient across restarts and ensures accurate historical tracking.

## Files

- `workflows/dca-trading-bot.json` — Main n8n workflow export, containing all logic for DCA detection, TP placement, fill handling, fallback matching, and notifications.

## Stack

- n8n
- Interactive Brokers TWS API
- Postgres
- Gmail
