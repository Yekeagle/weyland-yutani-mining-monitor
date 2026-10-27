# Weyland-Yutani Mining Monitor

A synthetic data source for daily resource extraction at Weyland-Yutani mines, and a web dashboard that reads it, finds anomalies and exports a PDF report.

| Part | What it is |
| --- | --- |
| **Data source** | A Google Spreadsheet built only from cell formulas (no script). Publicly editable. [Open the sheet](https://docs.google.com/spreadsheets/d/1Y4gW2oHh10QQmiosIbG1mG_zgL7x1ODybTCPH06pj_o/edit) |
| **Dashboard** | One static page, `index.html`. [Open the dashboard](https://yekeagle.github.io/weyland-yutani-mining-monitor/) |

> This repository is published for viewing. Issues and pull requests are not being accepted.

## 1. Data source (Google Sheets)

All settings live on the **Parameters** sheet. Yellow cells are inputs, gray cells are calculated.

| Setting | Where |
| --- | --- |
| Number of mines (1-20) and their names and base outputs | Parameters, sections 1 and 7 |
| Date range (up to 1100 days) | Parameters, section 1 |
| Distribution: Uniform or Normal, parameters switch on and off with the choice | Parameters, section 2 |
| Smoothing between consecutive days (AR(1) factor, keep or reduce noise amplitude) | Parameters, section 3 |
| Overall trend: linear or exponential, total change over the period | Parameters, section 4 |
| Day-of-week factors | Parameters, section 5 |
| Events: 8 slots with name, start date, duration, magnitude, and either a probability or a number of affected mines | Parameters, section 6 |
| Random seed (a new seed gives a new, repeatable data set) | Parameters, section 1 |

Sheets: `Parameters` (inputs), `Events` (which mines each event hits), `Calendar` (weekday, trend and event effect per day), `Random` (seeded pseudo-random numbers), `Noise` (smoothed noise), `Data` (the output the dashboard reads), `Charts` (line chart of all mines, updates with the parameters).

Output of mine *m* on day *t*:

```
ROUND( base_m x MAX(0, noise_t,m) x weekday_t x trend_t x MAX(0, 1 + sum_k event_k,t x hit_k,m), 1 )
```

See [docs/MODEL.md](docs/MODEL.md) for the details.

## 2. Dashboard

- Reads the `Data` tab of the sheet (live when served over http/https). Falls back to an embedded snapshot, and also accepts a CSV export.
- Per mine and total: mean, standard deviation, median, Q1, Q3, IQR, min, max.
- Anomaly detection with adjustable parameters: IQR rule, z-score, percentage distance from a moving average, Grubbs' test. Optional day-of-week adjustment. Methods can be combined ("any" or "at least N agree").
- Charts: line, bar, stacked. Anomalies are highlighted. Polynomial trendline of degree 1 to 4 with R-squared.
- PDF report: statistics tables, detection settings, charts, an episode table and one section per anomaly with a context chart.

Libraries (loaded from cdnjs): Chart.js 4.4.1, jsPDF 2.5.1, jsPDF-AutoTable 3.5.31.

To use another copy of the sheet, paste its link into the **Data source** panel (the sheet must be shared with link access).
