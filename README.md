# AI Material Planning Engine

[![n8n](https://img.shields.io/badge/n8n-automation-blue)](https://n8n.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**A hybrid inventory optimization system that combines rule-based logic with LLM-powered text generation.  
Proves: rules for calculation, AI for natural language output.**

---

## 📌 Table of Contents

- [Problem Statement](#problem-statement)
- [System Architecture](#system-architecture)
- [Rule‑based vs AI‑based – A Comparative Study](#rule-based-vs-ai-based--a-comparative-study)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Setup & Installation](#setup--installation)
- [How It Works (Step by Step)](#how-it-works-step-by-step)
- [Results & Impact](#results--impact)
- [Future Work](#future-work)
- [Author](#author)

---

## 🎯 Problem Statement

In manufacturing, inaccurate material demand forecasting leads to:
- **Stockouts** → production halts, revenue loss.
- **Overstock** → high holding costs, cash flow issues.

Most small‑to‑medium factories still rely on **manual Excel planning** or **rigid rule‑based ERP modules**. They either ignore demand volatility or cannot handle natural language outputs (e.g., purchase orders, overstock recommendations).

**Goal:** Build an automated, low‑cost material planning workflow that:
- Forecasts demand using statistical methods (fast & transparent).
- Generates professional purchase orders and overstock alerts **using AI** (natural, adaptive).
- Runs weekly with zero human intervention.

---

## 🧱 System Architecture

The workflow is implemented on **n8n** (open‑source automation) and triggered weekly by a Cron schedule.

![Workflow Overview](https://via.placeholder.com/800x400?text=Workflow+Diagram+-+see+JSON+file](https://github.com/Huy-Nguyen-Chualambo/AI-Material-Planning-Engine/blob/main/Workflow_image.png)

### Main components

| Component | Role | Implementation |
|-----------|------|----------------|
| **Data sources** | Inventory, usage history, supplier list | Google Sheets (3 tabs) |
| **Data merging & grouping** | Combine inventory and usage by material | `Function` node (JavaScript) |
| **Rule‑based forecasting** | Moving average + trend + safety stock | `Function` node (pure math) |
| **Shortage calculation** | Demand during lead time – current stock | `Function` node |
| **Decision branch** | If shortage > 0 → create PO; else check overstock | `IF` node |
| **AI overstock assistant** | Suggests STOP/REDUCE/HOLD actions | LLM (Groq/OpenRouter) + prompt |
| **Email generation** | Human‑friendly purchase request emails | LLM (via AI Agent) |
| **Logging** | Append purchase orders to Google Sheets | Google Sheets node |

> **Note:** The workflow contains **two parallel tracks** – one fully rule‑based (for calculation) and one AI‑based (for text). After benchmarking, the final design **uses rules for all arithmetic** and **AI only for text generation** (see comparison below).

---

## ⚖️ Rule‑based vs AI‑based – A Comparative Study

To decide where to apply AI, I implemented **both approaches** for each task and measured accuracy, speed, cost, and explainability.

| Task | Rule‑based (pure code) | LLM‑based (GPT/Gemini) | **Winner** | Why? |
|------|----------------------|------------------------|------------|------|
| **Demand forecasting** (moving avg + trend) | ✅ Exact, O(1), deterministic | ❌ Slow, token cost, hallucination risk | **Rule** | Math is not AI’s strength. |
| **Safety stock** (std dev × 1.5) | ✅ Exact, auditable | ❌ May output wrong formula | **Rule** | Same reason. |
| **Shortage calculation** | ✅ Simple arithmetic | ❌ Overkill, prone to errors | **Rule** | 2+2 = 4, no AI needed. |
| **Overstock detection** (weeks of cover) | ✅ IF‑ELSE, fast | ❌ Unnecessary complexity | **Rule** | Clear threshold logic. |
| **Recommendation for overstock** (STOP/REDUCE/HOLD) | ❌ Hard to define “extremely high” | ✅ **Natural, context‑aware** | **AI** | LLM understands fuzzy boundaries. |
| **Purchase order email** | ❌ Rigid, robotic text | ✅ **Professional, customisable** | **AI** | Better user experience. |
| **Supplier selection** (by lead time) | ✅ Sort + pick first | ❌ Unreliable sorting | **Rule** | Deterministic algorithm wins. |

### 🔍 Conclusion

> **Use rule‑based for any deterministic calculation (numbers, comparisons, sorting).  
> Use AI only for natural language generation or fuzzy classification (e.g., overstock severity).**

This hybrid approach gives you:
- **Speed & reliability** of rules (no API latency, no cost).
- **Flexibility & readability** of AI (emails, recommendations).

---

## ✨ Key Features

- **Fully automated** – runs every Monday at 8 AM.
- **Hybrid intelligence** – rules for math, LLM for text.
- **Real‑world data** – uses Google Sheets as a lightweight ERP.
- **Actionable outputs**:
  - Sends purchase orders to suppliers via Gmail.
  - Saves PO records to a `purchase_orders` sheet.
  - For overstock items, emails AI‑generated actions (`STOP_BUYING` / `REDUCE_PURCHASE` / `HOLD`).
- **Self‑documenting** – every node includes sticky notes explaining the logic.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| [n8n](https://n8n.io/) | Workflow automation (self‑hosted or cloud) |
| Google Sheets | Inventory, usage history, supplier database |
| Gmail | Sending POs and alerts |
| Groq / OpenRouter | LLM inference (fast, cheap) |
| JavaScript (ES6) | Custom logic in `Function` nodes |

---

## 🚀 Setup & Installation

### Prerequisites
- n8n instance (self‑hosted or n8n.cloud)
- Google account (for Sheets & Gmail)
- API key for Groq or OpenRouter (optional – only for AI nodes)

### Steps

1. **Clone this repository**
   ```bash
   git clone https://github.com/Huy-Nguyen-Chualambo/ai-material-planning-engine.git
