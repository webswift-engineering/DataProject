---
title: "project"
slug: project
createdAt: 2026-05-28T02:47:27.198Z
---

# Project Brief: E-Commerce Revenue Diagnostic & Growth Strategy

**Prepared for:** [Junior Analyst Name]  
**Role:** Independent Data Consultant  
**Client:** Online Retail Company (simulated using Google Merchandise Store data)  
**Duration:** 10 working days  
**Start Date:** ___________

> **New here? Start with `step_by_step_guide.md`.** This brief tells you *what* to do and *why*. The walkthrough tells you exactly *how* — every click, every query, every common mistake — and includes a **Day 0** setup section to get BigQuery, Python, and Looker Studio working before Day 1.

---

## 1. Business Context

You have been engaged by the leadership team of an online retail company. They have **three months of session-level data (Nov 2020 – Jan 2021)** and have noticed that **revenue per session dropped sharply in January** after a strong holiday peak. They are not sure how much of that drop is just normal post-holiday seasonality versus a structural problem they should worry about. They want answers to two questions:

1. **What is driving the post-holiday revenue/session drop?** — Decompose the decline into seasonal vs. structural components, and identify the largest structural contributor with data evidence.
2. **Where are the untapped growth opportunities?** — Find actionable segments or behaviors within existing customers that can be leveraged.

Your engagement ends with a **board-ready presentation** and a **self-serve analytics dashboard** the marketing and product teams can use going forward.

> **Note on data window:** The dataset covers exactly 92 days (2020-11-01 → 2021-01-31). All "trend" comparisons in this project are framed as **trailing 4 weeks vs. prior 4 weeks**, not "QoQ" or "YoY." If a stakeholder asks about longer trends, the honest answer is "we don't have the data — here's what I'd ask for next."

---

## 2. Dataset

**Source:** Google Analytics 4 sample dataset on BigQuery

```
bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*
```

| Resource | Link |
|----------|------|
| BigQuery Public Dataset Page | https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=ga4_obfuscated_sample_ecommerce&page=dataset |
| GA4 BigQuery Export Schema Docs | https://support.google.com/analytics/answer/7029846 |
| Google Merchandise Store (live site) | https://shop.googlemerchandisestore.com/ |

This contains real (obfuscated) user behavior data: page views, product interactions, purchases, traffic sources, device info, and geographic data.

---

## 3. Scope & Boundaries

| In Scope | Out of Scope |
|----------|-------------|
| Traffic source mix analysis | External market/competitor analysis |
| Device & platform behavior | App data (web only) |
| Conversion funnel diagnostics | Backend system performance |
| Customer segmentation on behavioral data | PII or individual user targeting |
| One experiment proposal | Running the actual experiment |

---

## 4. Day-by-Day Tasks

### Day 1 — Data Onboarding & Schema Mastery

- Set up GCP project and BigQuery **Sandbox** access (no credit card required — see `step_by_step_guide.md` Day 0 if you have not done this yet)
- Explore the GA4 schema: understand `event_params`, `user_properties`, `items` (all nested/repeated)
- Write profiling queries: date range coverage, event type distribution, null/completeness audit
- Produce a **data dictionary** for the 17 fields you'll use across the project

**The 17 fields you must document on Day 1:**

`event_date`, `event_timestamp`, `event_name`, `user_pseudo_id`, `ga_session_id` (from `event_params`), `page_location` (from `event_params`), `device.category`, `device.operating_system`, `geo.country`, `traffic_source.source`, `traffic_source.medium`, `traffic_source.name`, `items.item_id`, `items.item_name`, `items.item_category`, `items.price`, `items.quantity`, plus `ecommerce.purchase_revenue` and `ecommerce.transaction_id` for purchase events.

> **Important — channel attribution caveat:** This dataset is from 2020–2021. The cleaner field `session_traffic_source_last_click` was only added by Google in **July 2024** and is therefore **not populated** for this dataset. Use `traffic_source.source` and `traffic_source.medium` (user-level first-touch) as your channel dimension. To approximate session-level channel, derive a session id from `event_params` where `key = 'ga_session_id'`.

> **Data quality note:** Google states this dataset is obfuscated and "internal consistency might be somewhat limited." Document at least **3 inconsistencies** you find (e.g., purchase events whose `ecommerce.purchase_revenue` ≠ sum of `items.price * items.quantity`, sessions with no `session_start`, etc.). Real consultants always audit before they analyze.

**Output:** `data_dictionary.md` — field name, type, description, sample values, notes on quality + at least 3 documented data-quality findings

**References for Day 1:**

| Resource | Link |
|----------|------|
| GA4 BigQuery Export Schema (official) | https://support.google.com/analytics/answer/7029846 |
| GA4 Automatically Collected Events | https://support.google.com/analytics/answer/9234069 |
| BigQuery UNNEST & Nested Fields Guide | https://cloud.google.com/bigquery/docs/arrays |
| ydata-profiling — automated data profiling for pandas | https://github.com/ydataai/ydata-profiling |
| Data Dictionary Template (dbt) | https://docs.getdbt.com/docs/collaborate/documentation |
| BigQuery Sandbox setup (free, no credit card) | https://cloud.google.com/bigquery/docs/sandbox |

---

### Day 2 — Metric Framework

- Define your KPI tree (see reference below)
- Write SQL to compute each metric at daily and weekly grain
- Build a **baseline comparison table**: trailing 4 weeks vs prior 4 weeks, with % change

**Output:** `metric_definitions.md` + `baseline_metrics.sql`

**Reference — what a professional metric framework looks like:**

> Think of how consulting firms structure a "KPI tree" — revenue decomposes into sessions × conversion rate × AOV. Each branch can be further split. Your doc should read like a one-page appendix in a McKinsey deck: metric name, formula, granularity, owner, and why it matters.

**References for Day 2:**

| Resource | Link |
|----------|------|
| Amplitude — North Star Playbook (free book) | https://amplitude.com/books/north-star |
| Reforge — Growth loops are the new funnels | https://www.reforge.com/blog/growth-loops |
| Lenny's Newsletter (search "metrics" or "KPI tree") | https://www.lennysnewsletter.com/ |
| GitLab Handbook — KPI definitions (real-world example) | https://handbook.gitlab.com/handbook/company/kpis/ |

---

### Day 3 — Trend Decomposition

- Trend revenue/session weekly, split by: traffic source (organic, paid, direct, referral), device (desktop, mobile, tablet), user type (new vs returning)
- Use `LAG()`, `PERCENT_CHANGE`, moving averages
- Write a **1-page findings memo**: which dimension explains the decline, with supporting numbers

**Output:** `trend_analysis.sql` + `findings_memo.md` (1 page, structured as: Observation → Evidence → Implication)

**Reference style:**

> Model this after an analyst memo at Spotify or Airbnb — short, opinionated, data-backed. Example structure: "Revenue/session declined 18% QoQ. 72% of this decline is attributable to a shift in traffic mix: paid social sessions grew 40% but convert at 0.3% vs 2.1% for organic. Implication: CAC is rising while quality is falling."

**References for Day 3:**

| Resource | Link |
|----------|------|
| BigQuery Window Functions (LAG, LEAD, RANK) | https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts |
| Mode Analytics — SQL Window Functions Tutorial | https://mode.com/sql-tutorial/sql-window-functions |
| Airbnb Engineering blog (search "metrics") | https://medium.com/airbnb-engineering |
| GitLab SQL Style Guide (for clean query writing) | https://handbook.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/ |

---

### Day 4 — Funnel Analysis

- Build conversion funnel: `session_start` → `view_item` → `add_to_cart` → `begin_checkout` → `purchase`
- Compute step-to-step rates overall and by device + channel
- Identify the **single biggest drop-off** and form a hypothesis

**Output:** `funnel_analysis.sql` + annotated funnel table/chart

**Reference:**

> Look at how Amplitude or Mixpanel present funnel reports — horizontal bars showing volume and % at each step, with comparison overlays by segment. Your SQL output should feed directly into this kind of visual.

**References for Day 4:**

| Resource | Link |
|----------|------|
| Amplitude — Funnel Analysis Guide | https://amplitude.com/docs/analytics/charts/funnel-analysis |
| Mixpanel — Understanding Funnels | https://mixpanel.com/blog/introduction-to-analytics-funnels/ |
| Baymard Institute — Cart Abandonment Stats | https://baymard.com/lists/cart-abandonment-rate |
| Google — E-commerce Funnel in GA4 | https://support.google.com/analytics/answer/9327974 |
| Mode — Funnel Analysis with SQL (worked example) | https://mode.com/blog/funnel-analysis-with-rjmetrics/ |
| MotherDuck — Product Analytics SQL Guide (funnel section) | https://motherduck.com/learn/product-analytics-motherduck-duckdb/ |

---

### Day 5 — Customer Segmentation

- Pull user-level aggregates from BigQuery into Python (frequency, recency, total revenue, avg session duration, top category)
- Run **bucketed RFM** OR **K-means clustering** to produce 3–5 segments (see notes below)
- Profile each segment with: size (% of users), revenue share (% of total), defining behaviors, recommended marketing action

> **Important — RFM in a 92-day window:** Standard RFM uses a 12-month recency. With this dataset, recency only varies from 0 to ~92 days, so don't use quintiles — they will collapse. Use these explicit buckets instead:
> - **Recency** (days since last event, anchored on 2021-01-31): `R5 = 0–7`, `R4 = 8–21`, `R3 = 22–45`, `R2 = 46–75`, `R1 = 76+`
> - **Frequency** (sessions in window): `F1 = 1`, `F2 = 2`, `F3 = 3+`
> - **Monetary** (total revenue): quintile across **purchasers only** (most users have zero revenue — splitting them by quintile is meaningless)
>
> Alternative: K-means on `{sessions, page_views_per_session, items_viewed, cart_adds, purchases, total_revenue}` with `k=4` (use elbow plot to confirm). This often produces cleaner segments than RFM on this dataset.

**Output:** `segmentation.ipynb` with clear markdown narrative between code cells

**Reference:**

> Give each segment a memorable name ("High-Value Loyalists", "Browse-Only Window Shoppers", "Lapsed Big Spenders"). A stakeholder should understand the segment from the name alone.

**References for Day 5:**

| Resource | Link |
|----------|------|
| Mode Analytics — RFM Analysis Guide | https://mode.com/blog/rfm-analysis |
| scikit-learn — KMeans Clustering | https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html |
| Towards Data Science — Customer Segmentation with Python | https://towardsdatascience.com/customer-segmentation-using-k-means-clustering-d33964f238c3 |
| Shopify — RFM Segmentation Explained | https://www.shopify.com/blog/rfm-analysis |
| Google Colab (free Python notebook environment) | https://colab.research.google.com/ |

---

### Day 6 — Dashboard Architecture & Data Layer

- Sketch wireframe (hand-drawn or Figma/slides — layout of 4-6 charts + filters)
- Create 2-3 BigQuery views optimized for Looker Studio (pre-aggregated to avoid slow queries)
- Connect Looker Studio, validate data loads correctly

> **BigQuery Sandbox warning:** Sandbox supports `CREATE OR REPLACE VIEW` (good — that's all this project needs), but **all sandbox tables and views auto-expire after 60 days**, and DML statements (`INSERT`/`UPDATE`/`MERGE`) are blocked. So:
> - Use `CREATE OR REPLACE VIEW`, never `CREATE TABLE AS SELECT` — views recompute each time and don't take storage.
> - If you want your dashboard to keep working past 60 days, click the "Upgrade" button in BigQuery to enable billing. The free tier (1 TB queries/month) will keep this whole project at $0; the upgrade just removes the expiration.

**Output:** Wireframe image + BigQuery view SQL

**Reference — professional dashboard layout:**

> Follow the "inverted pyramid" pattern used by teams at Google and Netflix:
> - **Top row:** 4-5 KPI scorecards with sparklines and vs-prior-period comparison
> - **Middle:** 1-2 trend charts (the "what happened" layer)
> - **Bottom:** Breakdown tables or segment comparisons (the "why" layer)
> - **Filters:** Top-left, always visible — date range, device, channel

**References for Day 6:**

| Resource | Link |
|----------|------|
| Storytelling with Data — Dashboard Design | https://www.storytellingwithdata.com/blog/2019/9/30/how-to-design-a-dashboard |
| Looker Studio — Connect to BigQuery | https://support.google.com/looker-studio/answer/6370296 |
| Figma (free wireframing) | https://www.figma.com/ |
| Dashboard Design Patterns (Stephen Few) | https://www.perceptualedge.com/articles/Whitepapers/Dashboard_Design.pdf |
| Supermetrics — Dashboard Examples | https://supermetrics.com/blog/marketing-dashboard-examples |

---

### Day 7 — Dashboard Build

- Build all components in Looker Studio:
  - KPI scorecards (revenue, sessions, conversion rate, AOV) with period comparison
  - Revenue trend line with channel overlay
  - Funnel visualization
  - Segment performance comparison
- Add interactivity: date filter, device filter, channel filter
- Apply consistent formatting (font, color palette, alignment)

**Output:** Live Looker Studio dashboard (shared link)

**Reference:**

> Your dashboard should pass the "5-second test" — a viewer should understand the main story within 5 seconds of looking at it.

**References for Day 7:**

| Resource | Link |
|----------|------|
| Looker Studio Report Gallery (examples) | https://lookerstudio.google.com/gallery |
| Looker Studio — Calculated Fields | https://support.google.com/looker-studio/answer/6299685 |
| Cole Nussbaumer — Declutter Your Data Viz | https://www.storytellingwithdata.com/blog/2017/3/29/declutter-this-graph |
| Color palette tool (ColorBrewer) | https://colorbrewer2.org/ |
| Google Material Design — Data Visualization Guidelines | https://m3.material.io/styles/color/dynamic/choosing-a-source |

---

### Day 8 — Experiment Design

- Select your strongest insight that implies a testable action (e.g., "mobile checkout UX is causing 60% cart abandonment → simplify mobile checkout flow")
- Write a **1-page experiment brief**:
  - Hypothesis (if we do X, metric Y will improve by Z%)
  - Primary metric + guardrail metrics
  - Control vs treatment description
  - **Baseline conversion rate — pull this directly from your Day 4 funnel output.** Do not invent a number. State the segment (e.g., "mobile checkout completion = 38.2% from Day 4 funnel").
  - **Minimum Detectable Effect (MDE)** — pick a value you can defend (typically 5–10% relative). Justify in one sentence.
  - Sample size calculation (use statsmodels or Evan Miller calculator)
  - Estimated duration (= sample size ÷ daily eligible traffic from Day 1 profiling)
  - Expected revenue impact if successful (give a **range** $X–$Y with assumptions stated, not a single fake-precise number)

**Output:** `experiment_brief.md`

**Reference:**

> Model after Booking.com or Microsoft's experimentation docs. Structure: "We believe that [change] will result in [outcome] because [evidence]. We'll measure success by [metric] and ensure no harm to [guardrails]. At current traffic, we need N sessions over D days to detect a X% MDE at 80% power."

**References for Day 8:**

| Resource | Link |
|----------|------|
| Ronny Kohavi — Trustworthy Online Controlled Experiments | https://experimentguide.com/ |
| Evan Miller — A/B Test Sample Size Calculator | https://www.evanmiller.org/ab-testing/sample-size.html |
| Booking.com — How We Do Experimentation | https://blog.booking.com/how-booking-com-increases-the-power-of-online-experiments.html |
| Microsoft — ExP Platform (experiment design principles) | https://www.microsoft.com/en-us/research/group/experimentation-platform-exp/ |
| Statsmodels — Proportion Power Analysis (Python) | https://www.statsmodels.org/stable/generated/statsmodels.stats.power.NormalIndPower.html |
| VWO — A/B Testing Guide | https://vwo.com/ab-testing/ |

---

### Day 9 — Executive Presentation

Build a **10-slide max** deck:

| Slide | Content |
|-------|---------|
| 1 | Title + engagement summary |
| 2 | Executive summary (the answer in 3 bullets) |
| 3 | Revenue trend — the problem visualized |
| 4 | Root cause: traffic mix shift (with decomposition chart) |
| 5 | Root cause: funnel breakdown by device |
| 6 | Customer segments — who they are and what they're worth |
| 7 | Growth opportunity — which segment, what action |
| 8 | Recommended experiment — hypothesis + expected impact |
| 9 | Impact sizing — **range** $X–$Y if adopted, with explicit assumptions, sensitivity to top-1 unknown, and "reasons this could be wrong" |
| 10 | Next steps + what you'd analyze with more time |

Every slide title is an **insight statement**, not a topic label:
- ❌ "Traffic Source Analysis"
- ✅ "Paid Social Grew 40% But Converts at 1/7th the Rate of Organic"

**Output:** Slide deck (Google Slides or PDF)

**References for Day 9:**

| Resource | Link |
|----------|------|
| The Pyramid Principle (Barbara Minto) | https://www.amazon.com/Pyramid-Principle-Logic-Writing-Thinking/dp/0273710516 |
| HBR — How to Make Charts and Graphs that Aren't Misleading | https://hbr.org/2014/06/the-three-elements-of-successful-data-visualizations |
| Google Slides Templates (free, clean) | https://slidesgo.com/theme/business |
| Edward Tufte — The Visual Display of Quantitative Information | https://www.edwardtufte.com/tufte/books_vdqi |
| Harvard Business Review — How to Present Data | https://hbr.org/2020/02/present-your-data-like-a-pro |

---

### Day 10 — Mock Presentation & Final Polish

- Deliver a **15-minute presentation** (timed)
- Receive and respond to challenge questions:
  - "How confident are you in this root cause?"
  - "What's the downside risk of your recommendation?"
  - "What would you do differently with 2 more weeks?"
  - "How did you handle data quality issues?"
- Revise deliverables based on feedback
- Submit final package

**Output:** Revised deck + all deliverables packaged

**References for Day 10:**

| Resource | Link |
|----------|------|
| Harvard Business Review — How to Handle Tough Questions | https://hbr.org/2018/10/how-to-respond-to-tough-questions-in-a-presentation |
| Consulting Case Interview Frameworks (for structured answers) | https://www.preplounge.com/en/case-interview-basics |
| Jeff Bezos — Writing Culture & 6-Pager Memo Format | https://writingcooperative.com/the-anatomy-of-an-amazon-6-pager-fc79f31a41c9 |
| Presentation Zen (Garr Reynolds) | https://www.presentationzen.com/ |

---

## 5. Final Deliverable Package

```
project/
├── README.md                    # Project overview, how to navigate
├── sql/
│   ├── 01_data_profiling.sql
│   ├── 02_baseline_metrics.sql
│   ├── 03_trend_decomposition.sql
│   ├── 04_funnel_analysis.sql
│   └── 05_dashboard_views.sql
├── python/
│   └── segmentation.ipynb
├── docs/
│   ├── data_dictionary.md
│   ├── metric_definitions.md
│   ├── findings_memo.md
│   └── experiment_brief.md
├── dashboard/
│   └── looker_studio_link.md    # Shared dashboard URL
└── presentation/
    └── final_deck.pdf
```

---

## 6. References & Resources to Study

| Skill Area | Resource | Link |
|-----------|----------|------|
| SQL style guide | GitLab Data Team SQL Style Guide | https://handbook.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/ |
| Metric thinking | Reforge — North Star Metric Framework | https://www.reforge.com/blog/north-star-metric-growth |
| Dashboard design | Storytelling with Data (Cole Nussbaumer Knaflic) | https://www.storytellingwithdata.com/ |
| Dashboard examples | Looker Studio Report Gallery | https://lookerstudio.google.com/gallery |
| Segmentation (RFM) | Mode Analytics — RFM Analysis Guide | https://mode.com/blog/rfm-analysis |
| Experiment design | Ronny Kohavi — Trustworthy Online Controlled Experiments | https://experimentguide.com/ |
| Sample size calculator | Evan Miller A/B Test Calculator | https://www.evanmiller.org/ab-testing/sample-size.html |
| Presentation structure | The Pyramid Principle (Barbara Minto) | https://www.amazon.com/Pyramid-Principle-Logic-Writing-Thinking/dp/0273710516 |
| Funnel visualization reference | Amplitude Funnel Analysis Docs | https://amplitude.com/docs/analytics/charts/funnel-analysis |
| Python clustering | scikit-learn KMeans Documentation | https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html |
| BigQuery SQL reference | BigQuery Standard SQL Functions | https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators |
| GA4 event reference | GA4 Automatically Collected Events | https://support.google.com/analytics/answer/9234069 |

---

## 7. Evaluation Criteria

| Criteria | Weight | What I'm looking for |
|----------|--------|---------------------|
| Advanced SQL | 20% | Correct use of CTEs, window functions, UNNEST; readable and efficient |
| Metric understanding | 15% | Clear definitions, appropriate choices, awareness of limitations |
| Data visualization | 20% | Dashboard is actionable, clean, passes the 5-second test |
| Customer segmentation | 15% | Segments are distinct, sized, and tied to business actions |
| Experiment design | 15% | Statistically sound, clearly scoped, impact is quantified |
| Presentation & communication | 15% | Insight-led, concise, handles questions confidently |

---

## 8. Ground Rules

- **Ask questions early.** If something is ambiguous, clarify on Day 1, not Day 8.
- **Show your work.** Messy intermediate queries are fine — I want to see your thinking process.
- **Timebox.** If you're stuck for more than 2 hours on one thing, flag it. That's what a real consultant does with a client.
- **Quality over quantity.** 3 sharp insights beat 10 shallow ones.
- **Commit daily.** Push your work to a repo (or shared folder) at end of each day so I can review async.

---

## 9. Daily Standup Template

```
1. What did you finish yesterday?
2. What are you working on today?
3. Any blockers or things you're unsure about?
4. On a scale of 1-5, how confident are you in yesterday's output?
```

---

Ready to start. Day 1 begins when you confirm you have BigQuery access and have read this brief end-to-end.