# Football Player Market Value Prediction

An R project for estimating football player market values from season-level performance data and exploring how observed Transfermarkt valuations compare with model estimates. The analysis adds **prior-season club performance** as team context while keeping the evaluation time-ordered.

[![R analysis CI](https://github.com/AhadHashmani/football-market-value-team-context/actions/workflows/ci.yml/badge.svg)](https://github.com/AhadHashmani/football-market-value-team-context/actions/workflows/ci.yml)

> **Interpretation:** A model gap is a descriptive difference from an estimated value—not proof of mispricing. Team results are contextual features, not a causal estimate of an individual player's impact.

## Latest time-based evaluation

The current analysis trains on seasons **2014–2021**, selects and calibrates using **2022**, and evaluates on the held-out **2023** season.

| Test result | Value |
| --- | ---: |
| Selected model | XGBoost |
| Test players | 1,538 |
| RMSE | €10.70M |
| MAE | €6.25M |
| R-squared | 0.729 |
| Observed coverage of nominal 90% prediction interval | 88.2% |
| Test rows with prior-season team context | 91.4% |

These are results from the included data snapshots and current workflow, not a guarantee of future performance. Values are in **millions of euros**. The input snapshots do not include valuation timestamps, so the holdout is not a fully prospective forecast.

## What the analysis does

- Matches player-season records from FBref and Transfermarkt using normalized names, birth years, league names, and ambiguity checks.
- Engineers player performance, demographic, position, and team-context features.
- Adds the player's club results from the **previous** season: points per game, goal difference per game, final table position, and matches played.
- Compares linear regression, Elastic Net, random forest, and XGBoost. Model selection and prediction-interval calibration use the calibration season; the latest season is held out for evaluation.
- Produces player-level predictions, intervals, model gaps, evaluation metrics, and league/position summaries.
- Provides a Shiny app for exploring latest-season player estimates and their prior-season club context.

Prior-season standings are used to avoid using a season's final results to explain valuations from that same season. Promoted clubs and the first data season may have no prior-season record; missing features are imputed using training-season medians and accompanied by missingness indicators. Final table positions are derived by points, goal difference, and goals scored; tie-breakers can differ from official source standings.

## Repository map

| Path | Description |
| --- | --- |
| [`player_market_value_analysis.Rmd`](player_market_value_analysis.Rmd) | Main feature engineering, model training, evaluation, and report |
| [`app.R`](app.R) | Shiny player valuation and team-context explorer |
| [`R/data_pipeline.R`](R/data_pipeline.R) | Player/team name normalization, matching, and data preparation |
| [`R/model_metrics.R`](R/model_metrics.R) | Regression metrics, value conversion, and prediction intervals |
| [`scripts/build_team_performance.R`](scripts/build_team_performance.R) | Rebuilds the derived team-season dataset from pinned match-result sources |
| [`tests/run_checks.R`](tests/run_checks.R) | Data, matching, and metric checks |
| [`tests/run_dashboard_check.R`](tests/run_dashboard_check.R) | Dashboard smoke test |
| [`data/team_season_performance.csv`](data/team_season_performance.csv) | Derived team standings and performance, 2014–2023 |
| [`ststistics from all leagues.csv`](<ststistics from all leagues.csv>) | Included FBref player-season snapshot |
| [`market_values.csv`](market_values.csv) | Included Transfermarkt valuation snapshot |

The analysis also writes `player_market_value_analysis.html`, `evaluation_summary.csv`, `model_comparison.csv`, `player_valuations.csv`, `data_quality_summary.csv`, and league/position summaries when rendered.

## Data sources and reproducibility

Player performance and valuation snapshots are included in the repository. Their original sources are [FBref](https://fbref.com/en/) and [Transfermarkt](https://www.transfermarkt.com/); review each source's terms before reuse or redistribution.

Team match results come from [OpenFootball England](https://github.com/openfootball/england), [Germany](https://github.com/openfootball/deutschland), [Italy](https://github.com/openfootball/italy), [Spain](https://github.com/openfootball/espana), and France's files in [OpenFootball Europe](https://github.com/openfootball/europe). These projects dedicate their football data to the public domain (CC0). The script pins source commit IDs and writes the derived CSV:

```bash
Rscript scripts/build_team_performance.R
```

Rebuilding the team dataset requires internet access. Match data is provided as-is; the derived standings are not an official league table.

## Run locally

Requirements: **R 4.6.1** and **Pandoc** for rendering the HTML report. Restore the locked package library with `renv`:

```bash
git clone https://github.com/AhadHashmani/football-market-value-team-context.git
cd football-market-value-team-context
Rscript -e 'install.packages("renv", repos = "https://cloud.r-project.org")'
Rscript -e 'renv::restore(prompt = FALSE)'
```

Run checks and render the analysis from the repository root:

```bash
Rscript tests/run_checks.R
Rscript -e 'rmarkdown::render("player_market_value_analysis.Rmd", output_format = "html_document", knit_root_dir = getwd(), envir = new.env())'
```

After rendering, launch the interactive explorer:

```bash
Rscript -e 'shiny::runApp()'
```

The app reads the generated prediction and evaluation CSV files, so render the report before starting it. GitHub Actions runs the data checks, renders the report, and smoke-tests the dashboard on pushes and pull requests.

## Credits and reuse

The original analysis credits Aniketh Kalagara, Palash, Jovan, and Kelly. This repository preserves the existing Git history and project attribution.

No project-wide software/data license is specified here. The included player datasets retain their source-specific terms; the OpenFootball match data is CC0. Check the original sources' terms before copying or reusing any data.
