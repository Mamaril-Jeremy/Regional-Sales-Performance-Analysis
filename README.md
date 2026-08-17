# Regional Sales Performance Analysis

An interactive Tableau dashboard analyzing sales, profitability, and fulfillment performance across a multi-region retail operation.

![Tableau](https://img.shields.io/badge/Tableau-dashboard-e97627)
![LOD](https://img.shields.io/badge/LOD%20expressions-FIXED-1f77b4)
![Data](https://img.shields.io/badge/joins-6%20tables-green)
![Views](https://img.shields.io/badge/worksheets-10-lightgrey)

**[View the live dashboard on Tableau Public](https://public.tableau.com/app/profile/jeremy.mamaril/viz/DTSC600_FinalProject_JeremyMamaril/SalesStoryJeremyMamaril#1)**

FILL: add 2-3 screenshots here — the full dashboard, plus the profit-per-capita map and the treemap. Even with a live link, images in the README mean someone browsing GitHub sees the work without leaving the page.

---

## The question

Revenue tells you what sold. It doesn't tell you where you're actually making money, which teams are underperforming, or whether fulfillment is getting slower.

This dashboard separates those out. It answers four things a sales operation actually needs to know:

- **Where is profit concentrated**, adjusted for the size of the market rather than raw totals
- **Which channels and categories** carry the margin, not just the volume
- **Which teams are missing goal**, and by how much
- **Is fulfillment holding up** — how long orders take to ship and then to arrive

---

## Data

A relational retail sales dataset joined across six tables:

| Table | Contributes |
|---|---|
| Sales Orders | order lines, quantity, unit cost, unit price, dates |
| Customers | customer identity and distribution |
| Categories | product categorization |
| Sales Team | rep assignment and yearly sales goals |
| Store Locations | city, state, region, coordinates |
| Regions | regional rollup |

Store Locations also carries demographic attributes — population, median income, household income, land area — which is what makes the per-capita analysis possible.

Orders table consists of 7983 rows and ranged from December 2018 to July 2020.

---

## What I built

### Derived metrics

None of the core business metrics existed in the raw data. Each is a calculated field:

```
Revenue        = [UnitPrice] * [OrderQuantity]
Cost           = [UnitCost]  * [OrderQuantity]
Profit         = Revenue - Cost
Margin %       = (Revenue - Cost) / Revenue
Days to Ship   = DATEDIFF('day', [OrderDate], [ShipDate])
Days to Arrive = DATEDIFF('day', [ShipDate],  [DeliveryDate])
Goal Attainment= SUM(Revenue) / SUM([YearlySalesGoal])
```

### Level of Detail expressions

The more interesting calculations use LOD expressions to compute at a different grain than the view:

```
Profit per Capita  = {FIXED [CityName] : SUM([Profit]) / SUM([Population])}
Top Store by Sales = IF {FIXED [StoreID] : SUM([Revenue])}
                      = {MAX({FIXED [StoreID] : SUM([Revenue])})}
                     THEN [StoreID] END
```

**Profit per capita** Ranking cities by raw profit just returns the biggest cities. Dividing by population inside a `FIXED` expression normalizes for market size, which surfaces the cities that are genuinely efficient rather than just large. Same pattern is reused to identify the top-performing store, category, and region without hardcoding anything.

### Views

Ten worksheets feeding one dashboard:

| View | Shows |
|---|---|
| Total Profit per Month | profit trend over time |
| Total Sales per Category | category volume comparison |
| Total Profit per Sales Channel | channel margin mix |
| Profit per Capita by City | population-adjusted geographic map |
| Treemap by State | household-income-weighted state breakdown |
| Customer Distribution by Region | where the customer base sits |
| Unit Cost vs Unit Price | pricing and margin spread |
| Monthly Shipping and Delivery Time | fulfillment trend |
| Team Members with Least Sales | reps below goal |
| Frequency of Order Quantity | order size distribution |

Plus two KPI helper sheets driving the summary tiles on the dashboard.

### Interactivity

Ten parameters let a viewer re-slice the dashboard — filtering by region, category, channel, and date without needing separate views.

Main parameter in the dashboard is category.

---

## What the data showed

- Which region or city won on profit per capita, and was it different from the raw-profit leader? (If yes, that's your headline — it's the whole point of the LOD work.)
- Did any category sell well but carry thin margin?
- Was one channel meaningfully more profitable than the others?
- Did shipping or delivery time trend up over the period?
- How far below goal were the underperforming reps?

---

## Repo contents

```
├── DTSC600_FinalProject_JeremyMamaril.twbx   # packaged workbook (data included)
├── images/                                    # dashboard screenshots
└── README.md
```

To open the workbook locally you need Tableau Desktop or the free Tableau Public Desktop. The `.twbx` is packaged, so the data extract travels with it — no separate download or connection setup.

---

## Limitations

- Single dataset with no external benchmark, so "good margin" is relative to the rest of the data rather than to industry norms.
- The per-capita metric assumes the store's city population approximates its addressable market, which overstates the market for stores serving a wider area.
- Fully descriptive. It reports what happened; it doesn't forecast or model drivers.

---

## Built with

Tableau Desktop · LOD expressions · calculated fields · parameters · dual-axis and mapped views · packaged extract (`.hyper`)
