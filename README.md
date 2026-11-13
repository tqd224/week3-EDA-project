# King County Housing: Where Should Nicole Buy?

Exploratory data analysis (EDA) of one year of house sales in King County, Washington (Seattle and surroundings), with insights and recommendations for a buyer looking for a mid-priced home in a lively, central neighborhood.


| | |
|---|---|
| 📊 **Presentation (PDF)** | [`presentation/TQD_King_County_Nicole_EDA.pdf`](presentation/TQD_King_County_Nicole_EDA.pdf) |
| 📓 **Analysis notebook** | [`04_eda.ipynb`](04_eda.ipynb) |
| 🗺️ **Interactive map** | [`maps/zip_map.html`](maps/zip_map.html) (download and open in a browser) · [open online](https://<your-github-name>.github.io/<repo-name>/maps/zip_map.html) |

---

## Contents

1. [The client](#the-client)
2. [The data](#the-data)
3. [Hypotheses and results](#hypotheses-and-results)
4. [Insights](#insights)
5. [Recommendations](#recommendations)
6. [Repository structure](#repository-structure)
7. [Setup and how to run](#setup-and-how-to-run)
8. [Assumptions and limitations](#assumptions-and-limitations)

---

## The client

**Nicole Johnson, buyer:** wants a lively, central neighborhood in the middle price range and plans to buy within a year.

**Business question:** *Where and when can Nicole find a mid-priced house in a lively, central area of King County?*

Her wishes, translated into the data:

| Wish | Definition used in the analysis |
|---|---|
| Middle price range | Middle 50% of all sale prices (25th–75th percentile): **\$322,000 – \$645,000** |
| Central | Straight-line distance to downtown Seattle (47.6062, -122.3321), in km |
| Lively | Not in the data; approximated by central location and number of sales |
| Fair comparison of prices | Price per square foot (price ÷ living area), so houses of different sizes can be compared |

---

## The data

**King County House Data (KCHD):** 21,597 house sales from **May 2014 to May 2015**, 21 columns (see [`column_names.md`](column_names.md)).

The data is stored in two tables in the `eda` schema of the course's PostgreSQL database and is loaded in [`03_fetching_the_data_eda.ipynb`](03_fetching_the_data_eda.ipynb) with this join:

```sql
SET SCHEMA 'eda';

SELECT kchd.*,
       kchs."date",
       kchs.price
FROM king_county_house_details AS kchd
LEFT JOIN king_county_house_sales AS kchs
       ON kchd.id = kchs.house_id;
```

The `LEFT JOIN` keeps every house. A few houses were sold twice, so there is one row per **sale**. The result is saved as `data/eda.csv`, which is not committed to GitHub.

**Cleaning steps** (explained in the notebook):

| Issue | Treatment |
|---|---|
| `yr_renovated` stored ×10 (e.g. 19910) | Divided by 10; 0 and missing values = never renovated, set to `yr_built` |
| One house with 33 bedrooms | Corrected to 3 (typo: small living area) |
| Missing `sqft_basement` | Recalculated as `sqft_living − sqft_above` |
| Missing `waterfront` and `view` | Set to 0 (no waterfront, no special view) |
| Very expensive houses | Kept (real sales); medians and Spearman correlations limit their influence |

---

## Hypotheses and results

| # | Hypothesis | Result |
|---|---|---|
| 1 | **Where:** The closer a house is to downtown Seattle, the higher its price. Mid-priced houses near the center will therefore be smaller or have a lower grade. | ✅ **Supported.** Price per sq ft falls from \$369 within 5 km to \$168 at 30–50 km. In Nicole's budget, central houses are smaller (median 1,470 sq ft within 10 km vs. 2,415 sq ft at 30–50 km) and slightly lower grade (mean 7.2–7.4 within 20 km vs. 7.8–7.9 farther out). |
| 2 | **When:** Houses sold in winter are cheaper than houses sold in spring and summer. | ⚖️ **Partly supported.** County-wide, winter is 5.6% cheaper per sq ft than spring and summer (April peak ≈\$260, Dec–Jan ≈\$233–235). Within 5 km of downtown there is no seasonal effect (+0.4%). Winter has far fewer sales. |
| 3 | **What:** Older houses near the center are cheaper than newer houses of similar size. | ❌ **Not supported.** Within 5 km, pre-1940 houses cost the most per sq ft (≈\$374); no size group shows older houses as cheaper. |

---

## Insights

1. **Size and build quality drive price.** Grade (Spearman 0.66) and living area (0.64) have the strongest link with price. Bathrooms (0.50) and bedrooms (0.34) add little beyond size.
2. **Location matters most (geographical insight).** Price per sq ft more than doubles from the outskirts to downtown, and the north is pricier than the south (latitude 0.46). Within 5 km of downtown, the north (98119, 98109, 98102, 98112) is the most expensive, while 98122 and 98144 offer the best value.
3. **Spring is the priciest season** county-wide; December–January is about 10% cheaper than the April peak, but with far fewer houses for sale.
4. **Build quality adds value, age does not.** Price per sq ft rises from ≈\$228 (grade 7) to ≈\$312 (grade 11+), while old and new central houses cost about the same.

## Recommendations

1. **Search east of downtown:** 98122 (Central District, next to Capitol Hill) and 98144 (Beacon Hill, Mount Baker) are 2–3 km from downtown, offer the most houses in her budget and have a lower price per sq ft.
2. **Expect a smaller, slightly simpler house:** about 1,500 sq ft within 10 km of downtown, typically grade 7. A grade 8 house costs only about 8% more per sq ft.
3. **Skip spring:** prices peak in March–April; late summer to autumn balances price and choice.
4. **Choose old or new by taste:** older central houses are not cheaper.

---

## Repository structure

```
.
├── README.md                        ← you are here
├── 01_assignment.md                 ← project brief, tasks, deliverables, client list
├── 02_workflow.md                   ← recommended EDA workflow
├── 03_fetching_the_data_eda.ipynb   ← connects to PostgreSQL and exports data/eda.csv
├── 04_eda.ipynb                     ← main analysis (see overview below)
├── column_names.md                  ← data dictionary
├── presentation/
│   └── TQD_King_County_Nicole_EDA.pdf   ← client presentation (PDF export)
├── maps/
│   └── zip_map.html                 ← interactive map of price per sq ft (Plotly)
├── data/                            ← eda.csv goes here (data files are git-ignored)
├── .env.example                     ← template for database credentials
├── .python-version                  ← Python version for uv
├── pyproject.toml                   ← dependencies
├── uv.lock                          ← locked dependency versions
└── LICENSE                          ← MIT license (template by neuefische GmbH)
```

### `04_eda.ipynb` at a glance

| Section | What it does |
|---|---|
| Setup | Imports, plot settings, loading `data/eda.csv` |
| Hypotheses | Client, business question and the three hypotheses |
| Explore | Structure, summary statistics, missing values, boxplots |
| Clean | Cleaning decisions and code (always starts from a fresh copy of the raw data) |
| Feature engineering | `dist_km`, `price_per_sqft`, `month`, `season`, `age`, groups, Nicole's budget |
| Check relationships | Spearman correlations; price vs. living area by bathrooms and grade |
| Nicole's budget | Price per sq ft, distance vs. size, central zip codes, maps |
| Hypotheses 1–3 | Tests, plots and written results |
| Insights and recommendations | Summary for the client, assumptions and limitations |

Every plot is followed by a short "What we see" note.

---

## Setup and how to run

Requires [uv](https://docs.astral.sh/uv/getting-started/installation/) 0.13.5+ and Python 3.13 (see `.python-version`).

1. **Clone and install**

   ```bash
   git clone <copied-ssh-url>
   cd <repo-name>
   uv sync
   ```

2. **Add database credentials**

   ```bash
   cp .env.example .env
   ```

   Fill in your credentials in `.env`.

   > [!CAUTION]
   > `.env` holds secrets and is git-ignored. Never put real credentials in `.env.example`.

3. **Fetch the data:** run the **psycopg2** part of [`03_fetching_the_data_eda.ipynb`](03_fetching_the_data_eda.ipynb), which saves the joined data to `data/eda.csv`.

4. **Run the analysis:** open [`04_eda.ipynb`](04_eda.ipynb) in VS Code from the project root, select the `.venv` kernel, set `source` in the setup cell to your `data` folder, and click **Run All**.

> [!TIP]
> If a Plotly map is blank in VS Code, run `import plotly.io as pio; pio.renderers.default = "vscode"`, or open the saved HTML file in a browser.

---

## Assumptions and limitations

- Prices are from 2014–15: the **patterns**, not the dollar amounts, are the takeaway.
- Only one year of data, so seasonal patterns can't be confirmed across years.
- "Lively" is not measured directly; it is approximated by central location and sales activity.
- Distance is a straight line to downtown, not travel time (e.g. West Seattle lies across the water).
- Some central subgroups contain few sales, so small differences may be due to chance.

---

*Project template and assignment by [neuefische GmbH](https://github.com/neuefische/ds-eda-project-template), MIT license.*
