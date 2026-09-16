# Sales Power BI

![Power BI](https://img.shields.io/badge/Microsoft%20Power%20BI-Project-F2C811?logo=powerbi&logoColor=black)
![Format](https://img.shields.io/badge/format-PBIP-blue)
![Repository](https://img.shields.io/badge/repository-GitHub-181717?logo=github)

A source-controlled Power BI project for exploring sales performance, profitability, products, markets, discounts, and time-based trends. The project uses Power BI Project (`.pbip`) format so report definitions stay in text and Git can review changes.

> **Current data-source note:** The semantic model currently references Microsoft's local Power BI Desktop sample workbook, `Financial Sample.xlsx`. The workbook itself is not included in this repository.

## Contents

- [Project overview](#project-overview)
- [Screenshots](#screenshots)
- [Architecture](#architecture)
- [Report pages](#report-pages)
- [Data model](#data-model)
- [Data source and refresh](#data-source-and-refresh)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [Open the project](#open-the-project)
- [Contributing and change review](#contributing-and-change-review)
- [Git ignore rules](#git-ignore-rules)
- [Limitations and next steps](#limitations-and-next-steps)
- [License](#license)

## Project overview

Sales Power BI is an interactive sales-analysis report built around the Microsoft Power BI financial sample data. It is intended for analysts, business users, and Power BI developers who need to explore financial performance and product-market trends.

- Sales and profit performance over time
- Results by country, segment, and product
- Units sold, sale price, manufacturing price, gross sales, discounts, COGS, and profit
- Discount-band performance
- Daily, monthly, and yearly trends
- Relationships between the `financials` and `Sheet1` source tables

The repository stores the editable report and model definitions rather than a published Power BI Service dashboard or a binary `.pbix` file.

## Screenshots

The report definition currently contains three report pages: **Dashboard**, **Sales Breakdown**, and **Profits Breakdowns**. The project includes the report screenshots below for quick review in the repository.

Add screenshots to `docs/images/` using these suggested names:

```text
docs/
└── images/
    ├── dashboard.png
    ├── sales-breakdown.png
    └── profits-breakdowns.png
```

Then replace or enable the image references below:

### Dashboard

<img src="https://github.com/user-attachments/assets/42f9f0ca-71d5-49cd-9c64-41ad3031862a" alt="Dashboard" width="1347" />

The dashboard is the report landing page and includes time-based analysis such as the **Profits Per Month** visual.

### Sales Breakdown

<img src="https://github.com/user-attachments/assets/614cbfe3-6ea5-4927-9834-df407f94c2b8" alt="Sales Breakdown" width="1342" />

The Sales Breakdown page provides detailed filtering and comparison across sales dimensions, including date, country, segment, product, and discount band.

### Profits Breakdowns

<img src="https://github.com/user-attachments/assets/23e67c87-822c-4b3e-b290-c8d2a9570c13" alt="Profits Breakdowns" width="1347" />

The Profits Breakdowns page includes the **Profits VS Units Sold Per Segment** comparison and is intended for segment-level profitability analysis.

> **Adding screenshots:** Export each page from Power BI Desktop as an image, save the files under `docs/images/`, and commit them with the README update. Avoid publishing screenshots that expose confidential data or proprietary business information.

## Architecture

The project follows the Power BI Project architecture: a project entry point connects a report definition to a semantic model definition.

```text
┌─────────────────────────────┐
│        Sales BQ.pbip        │
│       Project entry point   │
└──────────────┬──────────────┘
               │ points to
               ▼
┌─────────────────────────────┐
│      Sales BQ.Report/       │
│  Pages, visuals, queries,   │
│  report settings, themes   │
└──────────────┬──────────────┘
               │ definition.pbir
               │ references
               ▼
┌─────────────────────────────┐
│   Sales BQ.SemanticModel/   │
│ Tables, columns, partitions,│
│ relationships, model options│
└──────────────┬──────────────┘
               │ Power Query M
               ▼
┌─────────────────────────────┐
│ Financial Sample.xlsx       │
│ financials table + Sheet1  │
│ Local Power BI Desktop path │
└─────────────────────────────┘
```

### Runtime and data flow

1. Power BI Desktop opens `Sales BQ.pbip`.
2. The project loads `Sales BQ.Report` as the report artifact.
3. `Sales BQ.Report/definition.pbir` resolves the semantic model at `../Sales BQ.SemanticModel`.
4. The semantic model loads the `financials` table and `Sheet1` from the Excel sample workbook through Power Query M partitions.
5. Report visuals query model columns and aggregates such as `Profit`, `Sales`, and `Units Sold`.
6. Date hierarchies support day, month, and year analysis, while report interactions apply filters and drill behavior across visuals.

### Architecture responsibilities

| Area | Location | Responsibility |
| --- | --- | --- |
| Project entry point | `Sales BQ.pbip` | Opens and identifies the Power BI report artifact |
| Report-to-model binding | `Sales BQ.Report/definition.pbir` | Points the report to the semantic model |
| Report pages | `Sales BQ.Report/definition/pages/` | Stores page metadata and visual containers |
| Visual queries | Individual `visual.json` files | Defines visual type, fields, aggregations, sorting, and formatting |
| Report settings | `Sales BQ.Report/definition/report.json` | Stores report-level behavior and theme references |
| Theme resources | `Sales BQ.Report/StaticResources/` | Stores shared and custom report styling |
| Model metadata | `Sales BQ.SemanticModel/definition/model.tmdl` | Defines culture, model options, and table references |
| Tables and partitions | `Sales BQ.SemanticModel/definition/tables/` | Defines fields, types, aggregations, and Power Query sources |
| Relationships | `Sales BQ.SemanticModel/definition/relationships.tmdl` | Defines date and country relationships |
| DAX exploration | `Sales BQ.SemanticModel/DAXQueries/` | Stores reusable DAX query examples |
| Model layout | `Sales BQ.SemanticModel/diagramLayout.json` | Stores the model diagram arrangement |

## Report pages

### Dashboard

The landing page uses a `1920 x 1080` canvas and provides an overview of the model. One inspected visual is a line/stacked-column combination chart titled **Profits Per Month**. It uses the `Sheet1` date hierarchy and tracks month-over-month performance.

### Sales Breakdown

This page uses a `1920 x 1080` canvas and includes an advanced date slicer configured against the `Sheet1` date hierarchy at the day level. The slicer is sorted ascending and belongs to the synchronized page filter state.

The page is intended for examining the effects of date selection across business dimensions such as country, segment, product, and discount band.

### Profits Breakdowns

This page uses a `1920 x 1080` canvas with a darker themed background. Its inspected donut chart is titled **Profits VS Units Sold Per Segment**. The chart:

- Uses `financials.Segment` as its category
- Aggregates `Sheet1.Profit`
- Aggregates `Sheet1.Units Sold`
- Sorts by profit in descending order
- Displays category, value, and percentage-of-total labels

## Data model

The semantic model uses the `en-US` culture, Power BI V3 data-source metadata, and Power BI time-intelligence support. It contains two primary imported tables with overlapping business columns.

### `financials`

Loaded from the workbook table named `financials`. Fields include:

- `Segment`
- `Country`
- `Product`
- `Discount Band`
- `Units Sold`
- `Manufacturing Price`
- `Sale Price`
- `Gross Sales`
- `Discounts`
- `Sales`
- `COGS`
- `Profit`
- `Date`
- `Month Number`
- `Month Name`
- `Year`

### `Sheet1`

Loaded from the workbook worksheet named `Sheet1`. It contains the same broad set of dimensions and financial fields and is used by several report visuals.

### Relationships

The model defines:

1. `financials.Date` to its generated local date table.
2. `Sheet1.Date` to its generated local date table.
3. `financials.Country` to `Sheet1.Country`, configured with bidirectional cross-filtering and many-side cardinality on the `Sheet1` side.

Because `financials` and `Sheet1` contain similar fields, contributors should confirm which table a new visual should use. Changes to the country relationship should be validated carefully for duplicate filtering effects and business logic.

### Example DAX query

The repository includes a starter query at `Sales BQ.SemanticModel/DAXQueries/Query 1.dax`:

```DAX
EVALUATE
    TOPN(20, 'financials')
```

This is useful for checking that the model loads and that the `financials` table can be queried.

## Data source and refresh

The current Power Query partitions reference this local Windows path:

```text
C:\Program Files\Microsoft Power BI Desktop\bin\SampleData\Financial Sample.xlsx
```

The model reads:

- The `financials` Excel table for the `financials` model table
- The `Sheet1` worksheet for the `Sheet1` model table

The workbook is not committed to this repository. On another machine, refresh may fail if the workbook is unavailable at that exact path.

### Making the project portable

For a team or production workflow, consider replacing the hard-coded path with one of the following:

- A Power Query parameter for the workbook location
- A shared SharePoint or OneDrive location
- A governed database or warehouse connection
- A controlled data lake or Fabric source
- A deployment-specific data source configured outside the repository

Do not commit confidential, regulated, or production data to the repository merely to make refresh work.

## Repository structure

```text
.
├── .gitignore
├── README.md
├── Sales BQ.pbip                         Power BI Project entry point
├── Sales BQ.Report/
│   ├── definition.pbir                   Report-to-model reference
│   ├── definition/
│   │   ├── pages/                        Page and visual definitions
│   │   ├── report.json                   Report settings and resources
│   │   └── version.json                  Definition version metadata
│   └── StaticResources/                  Shared and custom themes
└── Sales BQ.SemanticModel/
    ├── definition.pbism                  Semantic-model project metadata
    ├── definition/
    │   ├── cultures/                     en-US culture metadata
    │   ├── model.tmdl                    Model options and table references
    │   ├── relationships.tmdl            Table and date relationships
    │   └── tables/                       Columns and Power Query partitions
    ├── DAXQueries/                       DAX query examples
    ├── TMDLScripts/                      TMDL editing scripts
    ├── diagramLayout.json                Model diagram layout
    └── ...
```

## Requirements

- Windows
- Microsoft Power BI Desktop with Power BI Project support
- Access to the source workbook, or permission to update the data-source expression
- Git, if cloning or contributing through version control

There is no application runtime, package manager, build script, automated test runner, or external code dependency declared in this repository. Power BI Desktop is the development and execution environment for this project.

## Open the project

```powershell
git clone https://github.com/Mugweru01/Sales_PowerBi.git
cd Sales_PowerBi
Start-Process '.\Sales BQ.pbip'
```

Alternatively, open `Sales BQ.pbip` from Power BI Desktop using **File → Open**.

After opening:

1. Confirm that the report and semantic model load without definition errors.
2. Check the Excel source path in the model or Power Query editor.
3. Update the source path if `Financial Sample.xlsx` is not installed in the expected location.
4. Refresh the model.
5. Visit **Dashboard**, **Sales Breakdown**, and **Profits Breakdowns**.
6. Test slicers, cross-filtering, drill interactions, and totals.
7. Save the project and review generated text-file changes before committing them.

## Contributing and change review

Use Power BI Desktop for normal report development. Avoid manually editing generated JSON or TMDL unless you understand the Power BI Project schema and have a specific reason to do so.

When reviewing a change, use this guide:

| Change type | Review these files |
| --- | --- |
| Page or visual change | `Sales BQ.Report/definition/pages/` |
| Report settings | `Sales BQ.Report/definition/report.json` |
| Theme change | `Sales BQ.Report/StaticResources/` |
| Table or column change | `Sales BQ.SemanticModel/definition/tables/` |
| Relationship change | `Sales BQ.SemanticModel/definition/relationships.tmdl` |
| DAX query change | `Sales BQ.SemanticModel/DAXQueries/` |
| Model diagram change | `Sales BQ.SemanticModel/diagramLayout.json` |

Before merging a change:

- Open the project in Power BI Desktop.
- Refresh the model where possible.
- Confirm that all report pages load.
- Check KPI totals against an expected result.
- Test date filters and country filters.
- Check visuals using both `financials` and `Sheet1`.
- Ensure local cache and settings files are not staged.

## Git ignore rules

The repository ignores Power BI Desktop local working files:

```gitignore
**/.pbi/localSettings.json
**/.pbi/cache.abf
```

These files are machine-specific settings and cache artifacts. They should remain untracked and are not part of the shareable report definition.

## Limitations and next steps

### Current limitations

- The source workbook is not included.
- The Excel source path is hard-coded to a local Power BI Desktop installation path.
- There are no automated tests or CI workflows for opening, refreshing, or validating the report.
- The repository does not publish a Power BI Service workspace or scheduled refresh.
- The two overlapping source tables require care when adding new visuals or measures.
- No screenshot files are currently committed; the README includes placeholders for future exports.

### Recommended improvements

1. Add the three report screenshots under `docs/images/`.
2. Parameterize the Excel source path.
3. Decide whether `financials`, `Sheet1`, or a consolidated table should be the canonical reporting table.
4. Add explicit model measures for total sales, total profit, profit margin, discounts, COGS, and units sold.
5. Add a dedicated shared date table if time intelligence needs to work consistently across both source tables.
6. Add a data dictionary describing field definitions and business meaning.
7. Add a refresh and validation checklist for contributors.
8. Add a suitable open-source license if reuse is intended.

## License

No license file is currently included. Until a license is added, the repository should be treated as all-rights-reserved by default.
