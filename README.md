# Krista LLM Access Cost Calculator

An interactive, single-file web calculator that estimates how much an organization can save by routing routine LLM work to the private Krista LLM instead of sending everything to a frontier model. It pairs a five-question fit assessment with a live cost model, then gates a printable Cost & Fit Report behind a HubSpot lead form.

The whole tool is one HTML file with inline CSS and vanilla JavaScript. No build step, no framework, no dependencies. It is designed to drop into a WordPress/Elementor page as an HTML widget or run standalone.

## What it does

1. **Fit assessment.** Five multiple-choice questions score the visitor's situation (ungoverned usage, routine workload share, sensitive data exposure, team size, current spend) and return a fit tier: Strong fit, Good fit, or Worth a look.
2. **Cost model.** Three sliders (monthly token volume, percent of routine work, input:output token ratio) drive a live comparison of "all on a frontier model" versus "with Krista routing."
3. **Model comparison.** The visitor picks a comparison model from 11 commercial options across Anthropic, OpenAI, xAI, and Google. Prices blend at the selected input/output mix.
4. **Quality and speed panel.** For models with published Krista benchmark data, a table shows pass rate and latency by task type, head to head with the Krista LLM.
5. **Lead capture.** "Get My Full Report" opens an email gate (work email, first name, last name, company). Submission posts to the HubSpot Forms API along with a plain-text summary of every answer and result.
6. **Printable report.** After the gate, a formatted Cost & Fit Report renders on the page with a Print / Save as PDF button.

## How the math works

### Blended price per million tokens

Every model has separate input and output list prices. The calculator blends them using the input:output ratio slider (default 6:1):

```
input_share  = ratio / (ratio + 1)
output_share = 1 / (ratio + 1)
blended      = input_share * input_price + output_share * output_price
```

The 6:1 default reflects Krista's own production traffic, roughly 1,310 input tokens to 217 output tokens per request.

### Monthly cost and savings

```
frontier_cost = tokens_M * frontier_blended
krista_cost   = tokens_M * routine% * krista_blended
              + tokens_M * (1 - routine%) * frontier_blended
savings       = max(frontier_cost - krista_cost, 0)
savings_pct   = savings / frontier_cost
annual        = savings * 12
multiple      = frontier_blended / krista_blended
```

Routine work routes to the Krista LLM. Everything else stays on the selected frontier model.

### Worked example (defaults)

50M tokens per month, 80% routine, 6:1 mix, compared against Opus 4.8:

| | Value |
|---|---|
| Krista LLM blended rate | $0.17 / M tokens |
| Opus 4.8 blended rate | $7.86 / M tokens |
| All on Opus 4.8 | ~$393 / month |
| With Krista routing | ~$85 / month |
| Savings | ~$308 / month (78%), ~$3,690 / year |
| Price multiple | ~46x |

## Fit scoring

Each answer carries a point value from 0 to 3. The five answers are summed (maximum 15):

| Score | Tier | Message focus |
|---|---|---|
| 11 to 15 | Strong fit | Ungoverned usage, sensitive data, real spend, large routine workload |
| 7 to 10 | Good fit | Real routine volume plus governance gaps |
| 0 to 6 | Worth a look | Complex-reasoning workload or existing governance; cost math still applies |

Answering the routine vs. complex question also resets the routine % slider (85%, 50%, or 25%) so the cost estimate follows the assessment.

## Pricing data

### Krista LLM

| Input | Output |
|---|---|
| $0.15 / M | $0.30 / M |

### Comparison models (list price per million tokens)

| Model | Vendor | Input | Output |
|---|---|---|---|
| Fable 5 | Anthropic | $10.00 | $50.00 |
| Opus 4.8 (default) | Anthropic | $5.00 | $25.00 |
| Sonnet 5 | Anthropic | $3.00 | $15.00 |
| Haiku 4.5 | Anthropic | $1.00 | $5.00 |
| GPT-5.6 Sol | OpenAI | $5.00 | $30.00 |
| GPT-5.6 Terra | OpenAI | $2.50 | $15.00 |
| GPT-5.6 Luna | OpenAI | $1.00 | $6.00 |
| Grok 4.5 | xAI | $2.00 | $6.00 |
| Grok 4.3 | xAI | $1.25 | $2.50 |
| Gemini 3.5 Flash | Google | $2.70 | $16.20 |
| Gemini 3.1 Flash-Lite | Google | $0.45 | $2.70 |

Ultra-cheap open-source models (for example DeepSeek V4 Flash) are intentionally excluded. Their blended rate falls below Krista's, so they would not illustrate a routing comparison.

Sources: [Price Per Token](https://pricepertoken.com/), [Artificial Analysis](https://artificialanalysis.ai/tools/llm-price-calculator), [BenchLM](https://benchlm.ai/llm-pricing), and vendor pricing pages ([OpenAI](https://developers.openai.com/api/docs/pricing), [Claude](https://claude.com/pricing#api), [Google Gemini](https://ai.google.dev/gemini-api/docs/pricing)).

## Benchmark data

Quality and speed figures come from Krista's study [Breaking the Celebrity LLM Monopoly](https://krista.ai/breaking-the-celebrity-llm-monopoly/): 2,800+ real enterprise records, scored by independent LLM judges (GPT-5.1 and Gemini 3 Pro). Pass rate is the share of responses scoring above the enterprise quality threshold.

| Task type | Krista LLM pass rate | Krista LLM latency |
|---|---|---|
| Routine data processing | 88.5% | 1.49s |
| Agentic automation | 94.9% | 2.68s |
| Knowledge Q&A | 80.8% | 1.15s |
| Dialogue summarization | 97.1% | 14.51s |
| **Overall** | **83.4%** | |

Benchmarked comparison models: GPT-5.2 (85.7%), Claude Sonnet 4 (83.3%), Gemini 3 Pro (87.5%), o3-mini (87.0%).

The quality panel appears only when the selected comparison model has real benchmark data. It is hidden otherwise rather than filled with estimates. The panel labels a result "near-parity" when the overall gap is 5 points or less.

## Configuration

All configuration lives in constants at the top of the `<script>` block.

| Constant | Purpose |
|---|---|
| `KRISTA` | Krista LLM input and output price per million tokens |
| `MODELS` | Selectable comparison models. Set `default:true` on the one to preselect |
| `KRISTA_BENCHMARK` | Krista LLM pass rate and latency by task category |
| `BENCHMARKS` | Comparison model benchmarks, keyed by model `id` (must match an `id` in `MODELS` to display) |
| `HUBSPOT_PORTAL_ID` | HubSpot portal for form submissions |
| `HUBSPOT_FORM_GUID` | HubSpot form that receives submissions |

### Adding or updating a model

Add an entry to `MODELS`:

```js
{ id:'new-model', label:'New Model', vendor:'Vendor', input:2.00, output:8.00 }
```

To show head-to-head quality data for it, add a matching key to `BENCHMARKS` with `overall` and the four `categories` (`genericData`, `agentic`, `qa`, `keu`).

### HubSpot setup

The form posts to:

```
https://api.hsforms.com/submissions/v3/integration/submit/{PORTAL_ID}/{FORM_GUID}
```

Fields sent: `email`, `firstname`, `lastname`, `company`, and `message`. The `message` field carries a full text summary: fit tier, all five answers, slider inputs, costs, savings, and benchmark comparison when available. The HubSpot form must include a `message` property to store it.

To use a different form: HubSpot > Marketing > Lead Capture > Forms, create a form with those fields, then copy the GUID from Actions > Embed.

If submission fails, the visitor sees an error and a "Continue without submitting" link so the report is never blocked by a HubSpot outage.

## Embedding

**Standalone:** open `krista-llm-cost-calculator.html` in a browser or host it on any static server.

**WordPress / Elementor:** paste the contents of the file (the `<style>` block, the markup inside `<body>`, and the `<script>` block) into an HTML widget. All classes use the `klam-` prefix to avoid collisions with theme styles.

**Printing inside Elementor:** before printing, `printReport()` moves the report element to be a direct child of `<body>`, hides every other body child, and restores the original position on `afterprint`. This avoids blank pages and clipped layouts caused by positioned Elementor wrappers.

## File structure

```
krista-llm-cost-calculator.html
├── <style>    Krista-branded styles, responsive two-column grid, print rules
├── <body>
│   ├── Step 1: five-question fit assessment
│   ├── Step 2: token volume, routine %, and I/O ratio sliders
│   ├── Results: fit badge, model picker, cost bars, savings, CTAs
│   ├── What you get with Krista: role-based access, governed MCP
│   ├── Report container (hidden until unlocked)
│   └── Email gate modal
└── <script>   Pricing data, benchmarks, cost math, fit scoring,
               HubSpot submission, report rendering, print handling
```

## Browser support

Any modern evergreen browser. Uses `fetch`, `closest`, CSS grid, `accent-color`, and `clamp()`.

## Disclaimer

Estimates only. Actual savings depend on workload mix, model selection, and routing configuration. Comparison prices are published list rates and change often; review the `MODELS` table periodically against the sources above.
