# E-Commerce Risk: LLM SQL Co-Pilot

Part of a dual-platform e-commerce governance analytics portfolio. This repo covers the LLM-powered text-to-SQL co-pilot layer. The SQL fraud/risk auditing notebook and the Tableau/Power BI dashboards live in the companion repo: [dual-platform-ecommerce-governance-analytics](https://github.com/bernard-omondi/dual-platform-ecommerce-governance-analytics).

## Contents

| File | What it is |
|---|---|
| `03_LLM_CoPilot_SQL_Generation.ipynb` | A text-to-SQL co-pilot: schema-grounded prompting, a self-healing sanitizer for messy LLM output, a schema-hallucination guardrail, and a live pipeline tested against both the Anthropic and Google Gemini APIs |
| `sql_copilot_console.html` | An interactive browser demo of the same co-pilot pattern (see note below) |
| `synthetic_ecommerce_order_risk_dataset.csv` | The source 12,000-row, 23-column dataset the notebook builds its local SQLite database from |

## Data

12,000 synthetic e-commerce orders with risk labels (`Normal`, `Return Risk`, `Fraud Risk`) and governance signals (`high_risk_ip`, `late_delivery_risk`, `address_mismatch`, `customer_support_contacts`, etc). Fraud rate is ~3.7% -- intentionally imbalanced to reflect real-world fraud detection conditions. This is the same source dataset used across all three repos in this portfolio (SQL audits, dashboards, and this LLM co-pilot), so all three read from a single, consistent source of truth rather than diverging datasets.

## Running the notebook

The notebook builds a local SQLite database (`ecommerce_analytics.db`) from the CSV in its first code cell. The live LLM sections additionally need:

```bash
export ANTHROPIC_API_KEY="your-key-here"
export GEMINI_API_KEY="your-key-here"
jupyter-lab
```

Without these set, the notebook still displays the outputs from the original run -- the key-check cells will print a "not found" warning, and the live API cells raise a clear error rather than failing silently.

## The browser demo (`sql_copilot_console.html`)

**Important:** this file's live API call only works when opened inside an Anthropic-hosted artifact sandbox (e.g. within a Claude.ai conversation). Opened as a plain downloaded file in a regular browser, the API call will fail -- browsers can't authenticate directly to Anthropic's API without a server in between, and this demo has none. It's included here as a portfolio artifact showing the interaction design, not as a standalone deployable app.

## Key engineering decisions

- **Full 23-column schema grounding** for the production LLM pipeline, not the trimmed 6-column version used in early teaching cells -- an incomplete schema either causes hallucination or makes some legitimate questions unanswerable.
- **Self-healing sanitizer**: tries multiple recovery strategies in sequence (tagged fence, untagged fence, no fence with trailing prose, stray leading `sql` word) rather than assuming one fixed LLM output shape.
- **Schema guardrail is a token scan, not a full parser** -- deliberately: it catches the specific, common failure of a hallucinated column name cheaply, without the complexity of validating full SQL semantics.
- **Multi-provider by design**: the sanitizer and guardrail are provider-agnostic. The same functions validate output from both Claude and Gemini, proving the safety layer isn't coupled to one vendor's response shape.

## Portfolio context

This is one of three linked pieces:
1. **SQL fraud/risk auditing + Power BI / Tableau dashboards** -- see [dual-platform-ecommerce-governance-analytics](https://github.com/bernard-omondi/dual-platform-ecommerce-governance-analytics)
2. **This repo (LLM SQL co-pilot)** -- the AI-tooling layer on top of that same governance logic
3. **Predictive ML (in progress)** -- supervised classification for live fraud/return probability scoring, extending this same dataset's `is_fraud` / `is_returned` labels
