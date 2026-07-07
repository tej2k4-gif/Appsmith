# 🍽️ SufraEats Analytics Dashboard

An interactive Streamlit BI dashboard built on real SufraEats order and restaurant
data for Dubai: 47,713 orders (Jan–May 2025) across 230+ restaurants, 8 zones,
and 8 cuisines.

## Project structure

```
sufraeats/
├── app.py                     # Layout & wiring only - single entrypoint
├── requirements.txt
├── README.md
├── .gitignore
├── .streamlit/
│   └── config.toml            # Branded dark theme
├── data/
│   ├── cleaned_orders.csv
│   ├── sufraeats_restaurants_cleaned.csv
│   ├── data_quality_report.csv
│   └── missing_value_report.csv
└── utils/
    ├── load_data.py           # Loading, merging, cleaning flags, @st.cache_data
    ├── calculations.py        # Every metric's definition, in one place
    └── charts.py               # Reusable color maps + Plotly figure builders
```

## Setup (local)

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```

The app expects the four CSVs in `data/` (already included). If a file is
missing or empty, the dashboard shows a friendly error instead of crashing.

## Deploying to Streamlit Community Cloud

1. Push this folder to a public (or private) GitHub repo, including the
   `data/` folder and `.streamlit/config.toml`.
2. Go to [share.streamlit.io](https://share.streamlit.io), sign in with
   GitHub, and click **New app**.
3. Select your repo, branch, and set the main file path to `app.py`.
4. Click **Deploy**. Streamlit Cloud installs `requirements.txt` automatically.
5. Any push to the branch auto-redeploys.

> If your data is sensitive, don't commit real CSVs to a public repo -
> use Streamlit's **Secrets**/private repo instead, or swap in a database
> connection in `utils/load_data.py`.

## Dashboard tabs

| Tab | What it covers |
|---|---|
| 📊 Executive Overview | KPI cards with period-over-period deltas, revenue/order trend, auto-generated summary |
| 🗺️ Zone Intelligence | Dubai investment score, zone profitability, map + heatmap |
| 🍽️ Restaurants & Customers | Restaurant rankings, RFM customer segmentation, repeat-customer trend |
| 🚚 Operations | Delivery performance, hourly load proxy, cancellations/refunds |
| 💰 Financial | Revenue waterfall, Sankey flow, discount effectiveness, profit leakage |
| 🎯 Strategy | Cuisine expansion scoring, zone whitespace, final recommendation |
| 📄 Raw Data | Data-quality panel, searchable table, CSV download |

## Key methodology notes (read before presenting the numbers)

- **Revenue model**: only `Delivered` orders realize commission + delivery-fee
  revenue, minus discounts. `Refunded` orders are treated as a pure delivery-fee
  cost (money was paid out, the rider still ran the trip). `Cancelled` orders
  contribute zero revenue. See the docstring in `utils/load_data.py` if you
  need to change this assumption.
- **Data-quality flags, not silent fixes**: a handful of `delivery_time_min`
  values are negative or absurdly large (999 min), and a few `basket_value`
  entries are extreme outliers. These are flagged (`anomalous_*` columns) and
  excluded only from the specific averages they'd distort - they remain fully
  visible in the Raw Data tab.
- **"AI-based investment/expansion scoring"**: implemented as a transparent,
  weighted composite score (0-100), not a trained machine-learning model. The
  weights are documented and easy to tune in `utils/calculations.py`. Framing
  it this way keeps the methodology auditable and defensible if you're asked
  to explain it.
- **No rider utilization data**: the dataset has no rider ID or fleet-size
  column, so the Operations tab uses hourly order volume per zone as an
  honest proxy for operational load, rather than fabricating a utilization
  metric the data can't support.

## Design choices

- A single `build_color_map()` in `utils/charts.py` generates one color per
  zone/cuisine from a custom terracotta/teal palette (not Plotly/Streamlit
  defaults), so - for example - "Marina" is always the same color on the bar
  chart, the map, and the heatmap.
- Order-status colors are semantic (green = Delivered, red = Cancelled,
  gold = Refunded) rather than palette-cycled, since that mapping should never
  change regardless of data.
- Chart type varies deliberately per tab: line/dual-axis for trends, choropleth-style
  map + heatmap for zones, treemap for segments, waterfall + Sankey for financial
  flow, sunburst + scatter for strategy - so the dashboard doesn't read as one
  bar chart repeated seven times.
