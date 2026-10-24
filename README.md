# Uranium price analysis with Python

A Jupyter Notebook workflow exploring how physical investment demand, changes in expected uranium supply and nuclear activity relate to monthly uranium price changes.

The project combines IMF uranium prices, Sprott Physical Uranium Trust acquisitions, Cameco and Kazatomprom production announcements, and Ember nuclear electricity-generation data. It uses simple linear regression to compare the usefulness of the measured factors.

**Main finding:** Sprott purchases produced the lowest model errors over the 23-month evaluation period. Adding the collected supply revisions or nuclear-generation growth did not improve the Sprott-only model. These results describe associations and model performance, not causal influence weights.

## Repository contents

| File | Purpose |
|---|---|
| [external-data.xlsx](external-data.xlsx) | IMF commodity-price workbook. The code reads uranium from `PURAN` on the `External` sheet. |
| Project notebook or Python script | Data preparation, factor construction, model comparisons and plots. Open the code file uploaded to this repository. |
| [uranium_supply_announcements_2021_2026.csv](uranium_supply_announcements_2021_2026.csv) | Required supply input, with announcement dates, production years, forecast ranges, units and source URLs. |
| [Uranium_Price_Analysis_Step_by_Step_Report.docx](Uranium_Price_Analysis_Step_by_Step_Report.docx) | Step-by-step explanation of the simplified code, analytical choices, tables, plots and conclusions. |

Keep the spreadsheet and supply CSV in the notebook's working directory, or adjust their paths in the code. Keep these filenames if using the relative links above.

## Setup and running

Use Python 3 with Jupyter Notebook or JupyterLab. Install the packages used by the simplified workflow:

```bash
python -m pip install pandas numpy matplotlib openpyxl jupyterlab
```

Then start JupyterLab:

```bash
jupyter lab
```

Open the project notebook and run its cells from top to bottom. Internet access is needed for the Sprott extraction and Ember download unless local snapshots are used.

The simplified analysis starts after the loading cells have created:

- `uranium`: monthly IMF prices, indexed by date, with a `price` column.
- `sprott_monthly`: monthly physical purchases, indexed by date, with a `sprott_purchases_mlb` column.

Run the supply-preparation cell before the model comparisons: it adds the two supply columns to `analysis`. Restarting the kernel requires rerunning the preparation cells in order.

## Data sources

| Source | Measurement | Use in this project |
|---|---|---|
| [IMF Primary Commodity Prices](https://data.imf.org/en/Datasets/PCPS) | Monthly uranium spot price in US dollars per pound | Calculate monthly percentage price changes. |
| [Sprott Physical Uranium Trust](https://sprott.com/investment-strategies/exchange-listed-products/physical-commodity-funds/uranium/) | Reported physical acquisitions in pounds of U₃O₈ | Sum daily purchases into monthly totals. |
| [Cameco](https://www.cameco.com/media/news) and [Kazatomprom](https://www.kazatomprom.kz/en/media/news) | Production-guidance announcements | Measure changes in expected annual production; individual sources are retained in the CSV. |
| [Ember Monthly Electricity Data](https://ember-energy.org/data/monthly-electricity-data/) | Monthly nuclear generation in TWh | Calculate year-on-year growth for a fixed group with complete coverage. |

Sprott extraction relies on its website's acquisitions chart. The website layout and live datasets can change. Retain the input snapshots and retrieval dates to reproduce a particular run. Source data should be credited to the organisations listed above.

## Workflow

1. **Prepare uranium price changes.** Convert the IMF monthly labels to dates and calculate ordinary percentage changes. Changes allow comparisons across different price levels and reduce misleading relationships caused by shared trends.
2. **Add Sprott purchases.** Aggregate physical acquisitions by month and express them in million pounds. The factor measures uranium bought, not Sprott's share price or dollars spent.
3. **Add supply revisions.** Compare old and new production-forecast midpoints, convert quantities to million pounds of U₃O₈, and aggregate by announcement month. Keep current-year and future-year revisions separate. Unquantified warnings and licensed-capacity changes are excluded from the numerical revision factor.
4. **Add nuclear-generation growth.** Select individual countries or economies, exclude regional totals, and retain a fixed set of complete series. Compare generation with the same month a year earlier to reduce seasonal effects.
5. **Combine the factors.** Align all inputs to the same 59 months and check for missing values.
6. **Compare models.** Fit ordinary least squares using earlier months, estimate the next month, expand the training period and repeat. Compare factor models with zero change and the historical training average.
7. **Summarise performance.** Display MAE and RMSE, plot the comparisons, and calculate error reductions against the benchmarks.

## Period and coverage

| Item | Coverage |
|---|---|
| Main analysis | August 2021–June 2026: 59 months |
| Initial training sample | August 2021–July 2024: 36 months |
| Evaluation sample | August 2024–June 2026: 23 months |
| Nuclear-generation preparation | August 2020–June 2026, including an extra year for growth calculations |
| Nuclear-generation group | 28 countries or economies with complete observations in the original run |

The nuclear measure is **generation in covered countries**, not a complete world total. Ukraine, Switzerland and Pakistan were excluded because of missing months. Recorded zeros remain valid observations; missing generation is not filled with zero.

For supply, zero means **no net numerical revision recorded in the collected announcements**. It does not mean no relevant supply news occurred.

## Main variables and units

| Variable | Meaning |
|---|---|
| `analysis` | Main table of prices, price changes and monthly factors. |
| `price_change_pct` | Percentage change in the monthly uranium price. |
| `sprott_monthly` | Table of monthly physical purchases. |
| `sprott_purchases_mlb` | Monthly Sprott purchases in million pounds of U₃O₈. |
| `supply_current_year_mlb` | Change in expected production for the announcement year, in million pounds. |
| `supply_future_year_mlb` | Change in expected production for a later year, in million pounds. |
| `nuclear_generation_yoy_pct` | Year-on-year growth in covered-country nuclear generation. |
| `backtest` | Actual price changes and model estimates for the evaluation months. |
| `scores` | Model MAE and RMSE results. |

`lb` means pounds of mass; `mlb` in these column names means million pounds. U₃O₈ is uranium oxide. `tU` means tonnes of contained uranium: 1 tU converts to approximately 2,599.8 lb U₃O₈. TWh means terawatt-hours of electricity.

## Why use same-month purchases

The exploratory timing comparison used the same 58 matched months:

| Comparison | Correlation |
|---|---:|
| Same-month purchases and price change | 0.323 |
| Previous-month purchases and current price change | −0.042 |

The clearer relationship was within the same month, so this timing was retained. It does not establish which came first or show that purchases predict next month's prices. Timing was selected after inspecting the data, making this an exploratory choice.

## Results

The following values were obtained in the notebook and are documented in the report. Live data revisions may change subsequent runs.

| Model | MAE | RMSE |
|---|---:|---:|
| **Sprott only** | **2.920** | **3.785** |
| Sprott + supply | 2.950 | 3.826 |
| Sprott + nuclear | 3.135 | 4.243 |
| Zero change | 3.393 | 4.330 |
| Historical average | 3.823 | 4.803 |
| Supply only | 3.864 | 4.868 |
| Nuclear only | 4.268 | 5.375 |

- **MAE — mean absolute error:** average absolute error in estimated monthly price changes.
- **RMSE — root mean squared error:** gives greater weight to large errors.

Both are expressed in **percentage points**, and lower values are better. An actual increase of 5% versus an estimate of 2% is an error of 3 percentage points.

Sprott alone reduced MAE by approximately **14%** relative to predicting zero change. Adding supply revisions increased MAE by approximately **1%**; adding nuclear-generation growth increased it by approximately **7.4%**, relative to Sprott alone. These comparisons do not establish statistical significance.

## Interpretation and limitations

Sprott purchases were the most useful measured factor within the chosen model and period. Supply revisions and nuclear-generation growth added no improvement to the Sprott-only model. This does not show that supply and reactor demand are economically unimportant.

The analysis has several practical limits:

- Same-month factor values may only become available during or after the month. This evaluates retrospective relationships in later data, rather than a forecast available before each month begins.
- Purchases may respond to prices or shared market conditions; correlation and regression do not establish causation.
- Numerical supply revisions occur in only eight collected months, including one month with offsetting changes. Some revisions were preceded by warnings, and forecast midpoints or upper limits are approximate measures of news.
- Electricity generation is an indirect proxy for uranium purchasing, which can occur well ahead of reactor use.
- The 23-month evaluation period was reused during model comparison. Results remain exploratory, with no separate untouched final test.
- Website changes, revised datasets and publication delays affect reproducibility and real-time interpretation.

The **14% improvement is a reduction in model error**, not the proportion of uranium price changes caused by Sprott. The project reports relative model usefulness rather than forcing causal influence weights to sum to 100%.
