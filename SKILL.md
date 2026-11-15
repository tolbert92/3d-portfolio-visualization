
---
name: 3d-portfolio-visualization
description: >
  Validate portfolio optimization results and generate an interactive,
  responsive Plotly.js 3D portfolio explorer with Sharpe-ratio coloring,
  optimized-portfolio highlighting, efficient-frontier overlays,
  comparisons, accessible themes, and verified exports. Use when a user
  requests a 3D portfolio dashboard, visualization of portfolio candidates,
  or exploration of results from the Portfolio Optimization skill.
---

# Skill #6 — 3D Portfolio Visualization

**Project:** IE 591 AI Portfolio Optimization Agent  
**Version:** 1.0.0 (design specification)  
**Status:** Ready for implementation and standalone testing; Skill #5 integration pending  
**Upstream:** Skill #5 — Portfolio Optimization  
**Primary technology:** Plotly.js with HTML, CSS, and JavaScript

## 1. Purpose and operating boundaries

Transform *validated* portfolio optimization outputs into an interactive 3D analytical dashboard. This skill is a **visualization and reporting** layer, not an optimizer: do not generate investment recommendations, recalculate optimal weights, or silently alter optimization outputs. It may calculate a missing Sharpe ratio only from supplied, compatible return, volatility, and risk-free-rate inputs, and must disclose that calculation.

The initial portfolio universe is **VTI (equity), BND (fixed income), and SGOV (cash equivalent)**, with long-only weights totaling 100%. The input schema is designed to be extendable, but the Y-axis is specifically **VTI allocation (%)** in version 1. If VTI is absent, reject the dataset rather than relabeling the axis without authorization.

**Axes:** X = annualized portfolio volatility (%); Y = VTI allocation (%); Z = expected annualized return (%).  
**Color:** portfolio Sharpe ratio, with a labeled continuous color scale.  
**Modes:** Scatter, Surface, Both.  
**Default theme:** Dark Analytics; allow switching to Light Professional.  
**Layout:** responsive large visualization left, details/comparison inspector right; stack vertically on small screens.

All illustrative investment values must be clearly marked **SYNTHETIC / DEMONSTRATION DATA — NOT ACTUAL MARKET RESULTS**. Actual-data labels are allowed only when provenance supports them. Never imply that expected returns are guaranteed.

## 2. Activation and prerequisites

Use this skill when asked to visualize portfolio candidates, compare a chosen portfolio with the optimized portfolio, or produce a shareable portfolio exploration dashboard. Accept either:

1. A JSON file conforming to the versioned contract below; or
2. A directly passed, structured JSON-compatible object from Skill #5 using the **same contract**.

Do not accept an unstructured narrative as authoritative portfolio data. Ask for a structured result or explicitly offer a labeled synthetic demonstration. Do not require a dedicated user computer or a local installation merely to review generated outputs; where the execution environment supports file creation, provide artifacts directly.

**Runtime dependencies:** Plotly.js for interactive 3D charts; a PDF generation facility (such as a controlled browser print-to-PDF workflow or an appropriate PDF library) for PDF export. The offline HTML deliverable must bundle required scripts and styles locally/in-line, without CDN calls, remote fonts, telemetry, or remote market-data fetches. Do not claim offline capability until verified with networking disabled.

## 3. Canonical input contract (v1.0)

The following is a *schema example*, not an actual investment result. Percentages use **numeric percentage-point values** (e.g., `12.5` means 12.5%, not 0.125). Sharpe ratios are dimensionless.

```json
{
  "schema_version": "1.0",
  "dataset_id": "ie591-synthetic-demo-001",
  "data_provenance": {
    "classification": "synthetic",
    "source": "Skill #5 demonstration fixture",
    "as_of_date": "2026-11-15",
    "methodology": "Illustrative portfolio candidates; no live market data"
  },
  "units": {
    "return": "annualized_percent",
    "volatility": "annualized_percent",
    "allocation": "percent",
    "risk_free_rate": "annualized_percent"
  },
  "assets": ["VTI", "BND", "SGOV"],
  "constraints": {
    "long_only": true,
    "leverage_allowed": false,
    "allocation_sum_percent": 100
  },
  "risk_free_rate_percent": 3.0,
  "optimized_portfolio_id": "p002",
  "portfolios": [
    {
      "id": "p001",
      "allocations_percent": {"VTI": 40, "BND": 40, "SGOV": 20},
      "expected_return_percent": 6.1,
      "volatility_percent": 8.0,
      "sharpe_ratio": 0.3875
    },
    {
      "id": "p002",
      "allocations_percent": {"VTI": 60, "BND": 30, "SGOV": 10},
      "expected_return_percent": 7.2,
      "volatility_percent": 10.0,
      "sharpe_ratio": 0.42
    },
    {
      "id": "p003",
      "allocations_percent": {"VTI": 80, "BND": 15, "SGOV": 5},
      "expected_return_percent": 8.0,
      "volatility_percent": 13.0,
      "sharpe_ratio": 0.384615
    }
  ],
  "efficient_frontier": {
    "source": "skill_5",
    "portfolio_ids": ["p001", "p002", "p003"]
  }
}
```

### Field requirements

| Field | Requirement |
|---|---|
| `schema_version` | Required; accept supported major version 1 only; reject incompatible versions |
| `dataset_id` | Required, nonempty identifier |
| `data_provenance.classification` | Required: `synthetic` or `actual`; `actual` requires credible source/methodology metadata |
| `data_provenance.source`, `as_of_date`, `methodology` | Required nonempty provenance strings; date must be valid ISO `YYYY-MM-DD` |
| `units` | Required; must match v1 percentage-point conventions |
| `assets` | Required unique symbols; must contain VTI, BND, SGOV for initial profile |
| `constraints` | Required; enforce long-only, no leverage, total 100% for this profile |
| `risk_free_rate_percent` | Optional, finite numeric value; required to calculate missing Sharpe |
| `optimized_portfolio_id` | Required, must resolve to one valid portfolio |
| `portfolios` | Required nonempty array with unique IDs and required numeric fields |
| `portfolios[].allocations_percent` | Required finite values for every declared asset; sum 100% within tolerance |
| `portfolios[].expected_return_percent` | Required finite annualized percentage-point value |
| `portfolios[].volatility_percent` | Required finite value greater than zero for Sharpe calculation; zero volatility may be displayed only if otherwise valid and Sharpe marked unavailable |
| `portfolios[].sharpe_ratio` | Optional finite numeric value; derive only with compatible inputs |
| `efficient_frontier` | Optional; references upstream-computed frontier portfolios or explicit upstream points |

**Numeric precision and tolerances:** Do not silently normalize allocations. For validation, permit an absolute sum deviation of at most **0.01 percentage points** due to rounding, and disclose the tolerance; reject deviations larger than this. Individual long-only weights must be between 0 and 100 inclusive (within no tolerance for negative values). Preserve source precision in stored/exported data. For display, round percentages to two decimals and Sharpe to three decimals by default. Never replace original numbers with rounded display values in calculations.

**Sharpe convention:** `(expected_return_percent - risk_free_rate_percent) / volatility_percent`, provided the values use the same annualization and nominal/real convention. If supplied Sharpe conflicts with a recomputed value beyond a documented tolerance (default absolute 0.01), warn and preserve the supplied value pending review; do not silently overwrite it. If no valid Sharpe values exist, disable Sharpe coloring and show a clear warning rather than inventing a substitute metric.

**Frontier contract:** The preferred representation is `efficient_frontier.portfolio_ids` referencing existing portfolio IDs, ordered by increasing volatility. An alternate explicit `points` array may be supported only when each point carries its own volatility, return, VTI allocation, provenance, and upstream identifier. Do not compute or claim a new efficient frontier inside Skill #6. A conventional efficient frontier is defined in **risk–return space**; a line drawn through 3D points is a visualization of upstream frontier portfolios, not a mathematically defined 3D efficient-frontier surface.

## 4. Six-phase execution workflow

### Phase 1 — RECEIVE

- Identify input method: JSON file or direct Skill #5 structured transfer.
- Capture dataset ID, schema version, provenance, and requested outputs.
- Parse JSON safely. Never execute embedded code, HTML, or script fields.
- If input is missing, ask for it or offer an explicitly synthetic sample.

**Exit condition:** Parsed structured object or an actionable input error.

### Phase 2 — VALIDATE

- Verify schema version, units, unique IDs, required fields, finite numeric values, asset membership, long-only constraints, and allocation sums.
- Validate the optimized portfolio ID; do not mark an invalid portfolio as optimized.
- Validate Sharpe values or derive from compatible inputs when possible.
- Validate optional frontier references and source ordering.
- Classify issues into **critical**, **record-level**, and **noncritical**; produce a machine-readable validation report.

**Critical / block:** malformed or incompatible schema; missing provenance classification; missing VTI axis; no valid portfolios; missing/invalid optimized portfolio; invalid global units/constraints.  
**Record-level / exclude affected record:** invalid allocation totals, prohibited negative weights, missing/nonfinite risk or return. Exclusions must be counted, identified, and disclosed; never plot or export an excluded record as valid.  
**Noncritical / continue with warning:** missing optional frontier, descriptions, or Sharpe values that cannot be derived. Disable the affected feature if needed.

**Exit condition:** At least one valid portfolio including a valid optimized portfolio, or a blocking report.

### Phase 3 — PREPARE

- Preserve a complete, immutable copy of all **validated** records for comparisons and exports.
- Prepare 3D coordinates using the fixed axes and Sharpe color scale.
- Identify the optimized portfolio as a separate, always-visible trace.
- Prepare frontier overlay only from validated upstream frontier data.
- Implement **adaptive rendering** when chart density or device performance warrants it; choose a documented sampling strategy that retains the optimized portfolio, selected portfolio, and every valid upstream frontier point. Do not change any source metrics or optimization ranking.
- Report total valid records, plotted records, excluded records, and sampling method.
- Surface mode is enabled only if there are enough well-distributed valid points for a meaningful surface. A gridded or interpolated surface must be labeled **INTERPOLATED VISUALIZATION — NOT CALCULATED PORTFOLIOS**, must not invent feasible allocations, and must not be used for comparisons or exports as candidate records. If insufficient data exists, disable Surface and Both with an explanation or render the scatter-only component of Both with a warning.

**Exit condition:** Valid plot model, trace data, and inspector dataset.

### Phase 4 — VISUALIZE

Build a responsive HTML/CSS/JS dashboard using Plotly.js:

1. **3D explorer:** Scatter / Surface / Both mode selector; drag rotation, zoom, hover details, and point selection.
2. **Axes:** X annualized volatility (%), Y VTI allocation (%), Z expected annualized return (%).
3. **Sharpe color scale:** labeled legend and sensible treatment of missing values; do not encode Sharpe as a false zero.
4. **Optimized portfolio:** a prominent gold star marker, subtle glow via layered traces if feasible, and an inspector callout when selected. Keep it visible during adaptive sampling.
5. **Efficient frontier:** Show/Hide toggle; use upstream-computed data only. Label the 3D projection accurately and optionally provide a 2D risk–return inset for mathematical clarity.
6. **Inspector:** side-by-side selected vs. optimized metrics, allocation percentages, and signed differences for expected return, volatility, Sharpe, VTI, BND, SGOV. Highlight the optimized portfolio if selected.
7. **Theme switch:** Dark Analytics default; Light Professional alternate; preserve readable contrast, labels, and chart controls.
8. **Responsive behavior:** chart and inspector side-by-side on larger screens, stacked on smaller screens; keyboard-operable controls and visible focus states.
9. **Disclosure:** visible provenance classification, data as-of date, sampling indicator, validation warnings, and expected-return caveat.

**Plotly.js implementation note:** Confirm supported `scatter3d`, `surface`, `plotly_click`, theme/layout, and export APIs in the chosen bundled version. Do not assume that a built-in 3D gold star glyph or true volumetric glow exists; implement a tested layered-marker approximation or clearly document the fallback. Point selection in 3D may require `plotly_click` and a separate selection state rather than standard 2D lasso selection.

**Exit condition:** Interactive dashboard renders, controls function, and data labels remain accurate.

### Phase 5 — EXPORT

Produce a **complete export package** on request, each item independently validated:

| Artifact | Required contents | Verification |
|---|---|---|
| PNG | Current 3D chart view, theme, selected/highlighted portfolio, axis labels, provenance label | Open image; verify dimensions and nonblank chart; current camera view captured where supported |
| PDF | Portfolio summary: dataset metadata, selected vs. optimized metrics, allocations, chart image, validation notes, methodology/disclaimer | Open PDF; verify readable text, chart, page boundaries, and accurate metrics |
| CSV | **All validated candidate portfolios**, including IDs, VTI/BND/SGOV weights, return, volatility, Sharpe (blank if unavailable), optimized flag, frontier flag, provenance | Reparse CSV; check row count and numeric values against canonical validated data; disclose excluded count separately |
| HTML | Standalone interactive dashboard with all necessary Plotly.js/CSS/JS bundled, dataset embedded locally, no network dependency | Open with network disabled; verify modes, themes, selection, comparison, and frontier toggle |

Use safe filenames such as `portfolio_3d_chart.png`, `portfolio_summary.pdf`, `portfolio_candidates.csv`, and `portfolio_explorer_offline.html`. If a requested export cannot be produced or verified, report the failure accurately and do not claim the package is complete. Avoid embedding secrets, API keys, or personal identifiers. Escape user-supplied labels in HTML and CSV to mitigate injection and spreadsheet-formula execution risks (prefix risky CSV cells or otherwise neutralize formula-leading characters).

**PDF limitation:** A PDF is a static report, not an interactive 3D dashboard. **PNG limitation:** a static image may not capture hover-only content; include essential metrics in the PDF/CSV.

**Exit condition:** All requested deliverables exist and have passed format-specific verification, or failures are disclosed.

### Phase 6 — VERIFY

Run the acceptance suite below; summarize pass/fail/blocked for each test. Distinguish tested behavior from planned behavior. Provide filenames, data provenance, excluded counts, sampling counts, warnings, and outstanding integration requirements.

**Exit condition:** Test report and traceable artifact manifest.

## 5. Acceptance tests

### Standalone functional tests (synthetic fixtures)

- **T01** Valid v1 JSON imports from file and renders all eligible points.
- **T02** The identical object passed directly produces equivalent output.
- **T03** X/Y/Z axes, units, and tooltips match source metrics exactly.
- **T04** Scatter renders; Surface and Both render only when interpolation requirements are met; interpolated geometry is explicitly labeled.
- **T05** Sharpe color mapping and legend are correct; missing Sharpe is not represented as zero.
- **T06** Gold optimized marker is always visible and its callout displays correct metrics.
- **T07** Rotate, zoom, hover, click, and inspector selection work.
- **T08** Selected-vs-optimized comparisons and signed deltas are correct.
- **T09** Frontier Show/Hide works; only Skill #5-provided frontier is plotted; missing frontier disables overlay with warning.
- **T10** Dark default and Light theme both work; switching retains selected portfolio and camera state where practical.
- **T11** Dashboard is usable at desktop and mobile widths, with accessible control labels and focus visibility.

### Validation and performance tests

- **T12** Reject incompatible schema, wrong units, absent VTI, and missing optimized ID.
- **T13** Exclude records with allocation sums outside tolerance, negative weights, nonfinite return, or invalid volatility; report each exclusion.
- **T14** Derive missing Sharpe only from compatible inputs; otherwise warn and disable Sharpe coloring if no valid values remain.
- **T15** Missing optional frontier or descriptions does not block valid scatter visualization.
- **T16** Dataset containing only invalid candidates blocks rendering with actionable errors.
- **T17** Large dataset triggers adaptive sampling when needed; all valid optimized/frontier/selected points are preserved.
- **T18** Sampled chart never changes comparison values or the complete CSV row count.
- **T19** Synthetic labeling is prominent on dashboard and all exported reports; `actual` classification requires provenance.

### Export tests

- **T20** PNG opens and accurately reflects current chart state.
- **T21** PDF opens and includes chart, metadata, comparisons, and disclosures.
- **T22** CSV parses correctly and contains every validated portfolio, with no interpolated pseudo-portfolios.
- **T23** Offline HTML works with networking disabled, without external script or font requests.
- **T24** Export failures are surfaced rather than reported as success.

### Future integration tests (blocked until Skill #5 exists)

- **I01** Skill #5 output conforms to schema v1 or a documented compatible adapter.
- **I02** Direct transfer and saved JSON import produce identical metrics, provenance, and optimized ID.
- **I03** Efficient-frontier references and ordering match Skill #5 calculations.
- **I04** Invalid upstream results are rejected or excluded consistently without changing optimization outputs.
- **I05** End-to-end exports reproduce validated upstream metrics and preserve data classification.

**Acceptance gate:** All applicable standalone tests T01–T24 must pass; document any inapplicable surface tests with the fixture rationale. Integration tests I01–I05 remain **PENDING**, not PASS, until Skill #5 is implemented and tested.

## 6. Output manifest and agent response format

At completion, provide:

```text
Skill #6 — 3D Portfolio Visualization
Dataset: <dataset_id> | Classification: SYNTHETIC or ACTUAL | As of: <date>
Input: JSON file or direct Skill #5 transfer
Validation: <valid_count> valid; <excluded_count> excluded; <warning_count> warnings
Rendering: <plotted_count>/<valid_count> portfolios displayed; <sampling method>
Optimized portfolio: <id> (validated)
Modes: Scatter [status], Surface [status], Both [status]
Frontier: [available / unavailable; source]
Exports: PNG [pass/fail], PDF [pass/fail], CSV [pass/fail], HTML [pass/fail]
Standalone tests: <passed>/<applicable> passed
Skill #5 integration: PENDING / PASS / FAIL
Limitations: <explicit issues or none>
```

Never say a file was created, an export works, or a test passed without an actual artifact or observed test result. If the environment lacks file or browser execution capability, deliver implementation-ready source/instructions and clearly mark runtime tests as not executed.

## 7. Development guidance and non-goals

- Prefer maintainable modular functions: `parseInput`, `validateDataset`, `preparePlotData`, `renderPlot`, `renderInspector`, `exportPng`, `exportPdf`, `exportCsv`, `buildOfflineHtml`, and `runAcceptanceTests`.
- Keep the canonical validated dataset separate from sampled plot data.
- Use one versioned JSON contract for file import and direct transfer.
- Provide actionable validation messages with portfolio IDs and offending fields; avoid logging sensitive data.
- Use explicit chart labels and accessible colors; do not rely on color alone to identify the optimized portfolio.
- Do not send investor data to external analytics, APIs, or remote asset hosts.
- Do not treat interpolated surfaces as investable portfolios or as newly optimized candidates.
- Do not infer a Sharpe ratio when volatility is zero or required conventions are inconsistent.
- Do not promise an offline export if Plotly.js or other assets are loaded from a CDN.
- Do not provide investment advice, claim a guaranteed return, or misrepresent synthetic examples as market observations.

## 8. Definition of done

This skill's **specification** is complete when the above workflow, contract, validation policy, and tests are documented. The **implementation** is complete only after a working dashboard and the four export artifacts have been generated and independently verified against synthetic fixtures. The **integration** is complete only after the five Skill #5 integration tests pass using genuine Skill #5 output.

**Academic demonstration note:** Show the six-phase workflow, explain the distinction between calculated portfolio candidates and interpolated visual geometry, demonstrate the two themes and portfolio comparison, and disclose that standalone tests use synthetic data until upstream integration is complete.
