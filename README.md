# 4R Optimization Engine

> Recommends fertilizer management plans (Right Rate, Right Time, Right Source, Right Place) for any crop in any region using crop simulation and Pareto analysis.

## What it does

The 4R Optimization Engine samples representative geographic points from satellite data, generates a combinatorial search space of nitrogen and phosphorus fertilizer scenarios, runs multi-year DSSAT crop simulations, and identifies Pareto-optimal plans along a yield vs. N₂O trade-off curve. Results are surfaced through a FastAPI backend and an interactive React dashboard.

## Architecture

```
Spatial sampling  →  Scenario generation  →  Crop simulation  →  Pareto analysis  →  Dashboard
GADM / GAEZ           N×P×K combos            agrosim / DSSAT      5-tier plans       React + FastAPI
AEZ stratification    search_space config      multi-year runs      response envelope   maps & charts
representative pts    scenarios.parquet        annual_results       scorecard           RAG chat
```

## Quickstart

### Prerequisites

- Python 3.11+
- [agrosim](https://github.com/Said-taoussi/AgroSim) with a working DSSAT installation
- Node.js 18+ (for the frontend only)

### Install

```bash
git clone https://github.com/Ettousy-Said_bcgprod/4r-optimization-engine.git
cd 4r-optimization-engine
pip install -e ".[dev,api,bayesian]"
```

Verify:

```bash
fourr --version
```

### Run the Brazil maize example

```bash
# Step 1: select representative simulation points for Mato Grosso maize
fourr spatial run \
  --country BRA --admin-level 1 --region "Mato Grosso" \
  --crop maize --run-dir data/runs/quickstart

# Step 2: generate scenarios, simulate, and analyze
# (uses bayesian_pareto_pathc strategy by default)
fourr run \
  --config configs/presets/brazil_maize.yaml \
  --run-dir data/runs/quickstart \
  --simulate --workers 4
```

Or run everything in a single command:

```bash
fourr pipeline \
  --config configs/presets/brazil_maize.yaml \
  --run-dir data/runs/quickstart \
  --workers 4
```

### Launch the dashboard

```bash
cp .env.example .env          # fill in Azure credentials if you want the chat feature
cd frontend && npm install && npm run dev &
uvicorn api.app:app --reload
```

Open http://localhost:5173.

## What you'll get

After a successful run, three key outputs are written to `data/runs/quickstart/`:

- **`simulation_points.parquet`** — the representative geographic points used for simulation, with AEZ class labels and GAEZ crop statistics.
- **`plans/recommended_plans.parquet`** — five fertilizer plans per point (Baseline, Conservative, Balanced, Intensive, Max_Yield), each with full 4R prescription, yield, N₂O, NUE/PUE metrics, and environmental flags.
- **Dashboard** — interactive map, Pareto chart, plan comparison table, surface heatmap, climate panel, and RAG-backed chat over DSSAT documentation.

## Documentation

| Guide                                                  | Description                                       |
| ------------------------------------------------------ | ------------------------------------------------- |
| [docs/getting-started.md](docs/getting-started.md)     | Full onboarding walkthrough                       |
| [docs/architecture.md](docs/architecture.md)           | Pipeline design, module maps, data schema         |
| [docs/configuration.md](docs/configuration.md)         | Complete YAML configuration reference             |
| [docs/cli-reference.md](docs/cli-reference.md)         | All CLI commands and options                      |
| [docs/api-reference.md](docs/api-reference.md)         | REST API endpoints                                |
| [docs/search-strategies.md](docs/search-strategies.md) | full_grid, multi_pass, bayesian_pareto strategies |
| [docs/frontend.md](docs/frontend.md)                   | React frontend guide                              |
| [docs/glossary.md](docs/glossary.md)                   | Domain terminology                                |
| [docs/troubleshooting.md](docs/troubleshooting.md)     | Common problems and fixes                         |

## Project status

V1 is feature-complete. The core pipeline (spatial sampling, scenario generation, DSSAT simulation, Pareto analysis), Bayesian search (BoTorch qEHVI), the React dashboard, and RAG chat are all implemented.

## License

MIT — see [LICENSE](LICENSE).

## Citation

If you use this engine in research, please cite:

```bibtex
@software{4r_optimization_engine,
  title  = {4R Optimization Engine},
  year   = {2025},
  url    = {https://github.com/Said-taoussi/4r-optimization-engine}
}
```
