# n8n-currency-alerts
Automated USD/NGN exchange rate tracker and Slack alert pipeline built with n8n.


# 💱 Automated FX Rate Tracker & Alert System (n8n)

An automated workflow built with **n8n** that monitors daily **USD/NGN** exchange rates, logs historical data to Google Sheets for tracking, and triggers real-time buy-window alerts in Slack when the rate drops to or below a target threshold.

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=flat-square&logo=n8n)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-Data_Logging-34A853?style=flat-square&logo=googlesheets)
![Slack](https://img.shields.io/badge/Slack-Real--Time_Alerts-4A154B?style=flat-square&logo=slack)
![REST API](https://img.shields.io/badge/ExchangeRate--API-REST-0052CC?style=flat-square)

---

## ⚙️ Workflow Architecture

The pipeline executes automatically every day at **9:00 AM** and follows a 5-stage process:

1. **Schedule Trigger**: Initiates the workflow daily at `09:00`.
2. **Fetch Live FX Rates (`HTTP Request`)**: Queries the open ExchangeRate-API (`https://open.er-api.com/v6/latest/USD`) with built-in retry logic (`retryOnFail`, 5000ms interval) for resilience against network drops.
3. **Data Normalization (`Edit Fields / Set`)**: Extracts and structures the payload into clean variables:
   - `pair`: `"USD/NGN"`
   - `rate`: Live NGN rate extracted from `$json.rates.NGN`
   - `date`: ISO timestamp (`$now.toISO()`)
   - `target`: `1700` (configurable threshold)
4. **Parallel Processing**:
   - **Historical Logging (`Google Sheets`)**: Appends every daily check (`Date`, `Pair`, `Rate`, `Target`, `Alerted`) into the **FX History** spreadsheet for long-term trend analysis.
   - **Threshold Evaluation (`IF Node`)**: Evaluates whether the live rate meets the buy condition (`rate <= target`).
5. **Instant Notification (`Slack`)**: If the condition is met (`true`), dispatches a formatted alert to the `#finance-budget` channel:
   > *Buy Window:* USD/NGN is [rate] (target ≤ 1700) on [date]

---

## 🛠️ Tech Stack & Integrations

* **Orchestration:** [n8n](https://n8n.io/)
* **Data Source:** [Open Exchange Rates API](https://open.er-api.com/)
* **Database / Logging:** Google Sheets API (OAuth2)
* **Notification Engine:** Slack API (OAuth2)

---

## 🚀 How to Import & Use This Workflow

### Prerequisites
* An active **n8n** instance (Self-hosted or Cloud)
* A **Google Cloud** OAuth2 credential configured for Google Sheets
* A **Slack** OAuth2 credential with `chat:write` permissions

### Setup Steps
1. **Download the Workflow:** Clone this repository or download the `Currency Alerts.json` file directly.
2. **Import into n8n:**
   * Open your n8n dashboard and create a new workflow.
   * Click the **three dots (`...`)** in the top-right corner → **Import from File** → select `Currency Alerts.json`.
3. **Configure Google Sheets:**
   * Create a Google Sheet with the following column headers in `Sheet1`: `Date`, `Pair`, `Rate`, `Target`, `Alerted`.
   * Open the **Append row in sheet** node, connect your Google Sheets account, and select your spreadsheet.
4. **Configure Slack:**
   * Open the **Send a message** node, connect your Slack OAuth2 account, and select your target channel.
5. **Customize Thresholds (Optional):**
   * Open the **Edit Fields** node and adjust the `target` value (default is `1700`) or currency pair to match your requirements.
6. **Activate:** Toggle the workflow to **Active** in the top-right corner.

---

## 📂 Repository Structure

```text
├── Currency Alerts.json   # Exported n8n workflow template
├── README.md              # Project documentation
└── assets/                # (Optional) Workflow screenshots
