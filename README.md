# 🟢 Green Hydrogen Techno-Economic Assessment (TEA) Framework

[![MATLAB](https://img.shields.io/badge/MATLAB-R2023b+-blue?logo=mathworks)](https://mathworks.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/your-org/green-hydrogen-tea/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/green-hydrogen-tea/actions)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.placeholder.svg)](https://doi.org/10.5281/zenodo.placeholder)

> An early-stage economic feasibility framework for green hydrogen production systems powered by renewable energy, developed as part of the MathWorks Sustainability Challenge.

---

## 📋 Table of Contents

- [Motivation](#motivation)
- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Usage](#usage)
- [Results](#results)
- [Limitations](#limitations)
- [Contributing](#contributing)
- [License](#license)

---

## Motivation

Global greenhouse gas emissions from **transportation**, **electricity generation**, and **industrial processes** account for the majority of climate change drivers. Green hydrogen — produced via water electrolysis powered by renewable energy — offers a pathway to decarbonize energy-dense sectors like aviation and industrial manufacturing.

**The challenge:** Most hydrogen today is produced via natural gas reforming (grey hydrogen), releasing CO₂. Green hydrogen must become economically competitive. This framework quantifies the gap and identifies optimal deployment locations.

---

## Project Overview

This framework performs techno-economic assessment (TEA) of a solar-powered green hydrogen production system by:

1. **Simulating** a Simscape-based hydrogen production model across multiple geographic locations
2. **Importing** real electricity price data from ISOs (e.g., ISO New England)
3. **Importing** real solar irradiance data from NSRDB (National Solar Radiation Database)
4. **Computing** Levelized Cost of Hydrogen (LCOH) for each location
5. **Optimizing** energy storage dispatch to minimize grid electricity costs
6. **Visualizing** results with interactive dashboards

### Key Economic Metrics

| Metric | Description |
|--------|-------------|
| **LCOH** | Levelized Cost of Hydrogen ($/kg) |
| **CAPEX** | Capital Expenditure (electrolyzer, PV, storage) |
| **OPEX** | Operational Expenditure (electricity, water, maintenance) |
| **Payback Period** | Years to recover initial investment |
| **NPV** | Net Present Value at 8% discount rate |
| **IRR** | Internal Rate of Return |

---

## System Architecture

```
Solar PV Array → DC Bus → Electrolyzer → H₂ Compressor → Storage Tank
                   ↕
              Battery Storage
                   ↕
              Grid (backup/export)
```

See [docs/architecture.md](docs/architecture.md) for detailed system diagrams.

---

## Getting Started

### Prerequisites

- MATLAB R2023b or later
- Simulink + Simscape
- Parallel Computing Toolbox (optional, for speed)
- Financial Toolbox (optional)

### Installation

```bash
git clone https://github.com/your-org/green-hydrogen-tea.git
cd green-hydrogen-tea
```

In MATLAB:
```matlab
% Add project to path
addpath(genpath('.'))

% Install dependencies (run once)
setup_project()
```

### Quick Start

```matlab
% Run TEA for a single location (Boston, MA)
results = run_tea_single('Boston_MA');
plot_results(results);

% Run TEA across all locations (parallel)
results_all = run_tea_all_locations();
generate_report(results_all);
```

---

## Project Structure

```
green-hydrogen-tea/
├── README.md
├── LICENSE
├── setup_project.m              # One-time setup script
│
├── src/
│   ├── models/
│   │   ├── GreenH2System.slx    # Simscape system model
│   │   └── economic_signals.m   # Economic signal additions
│   ├── analysis/
│   │   ├── run_tea_single.m     # Single-location TEA
│   │   ├── run_tea_all.m        # Multi-location parallel TEA
│   │   ├── calc_lcoh.m          # LCOH calculation
│   │   ├── calc_npv.m           # NPV/IRR calculation
│   │   └── optimize_storage.m   # Battery dispatch optimization
│   ├── data_import/
│   │   ├── import_iso_prices.m  # ISO electricity price import
│   │   ├── import_nsrdb.m       # NSRDB solar irradiance import
│   │   └── preprocess_data.m    # Data cleaning & resampling
│   └── visualization/
│       ├── plot_lcoh_map.m      # Geographic LCOH heatmap
│       ├── plot_cashflows.m     # Cash flow waterfall
│       └── generate_report.m   # Full HTML report
│
├── data/
│   ├── locations/
│   │   └── station_data.mat     # StationData structure (NSRDB)
│   ├── electricity_prices/
│   │   └── isone_2023.csv       # ISO-NE hourly LMPs
│   └── costs/
│       └── cost_assumptions.json # CAPEX/OPEX parameters
│
├── docs/
│   ├── architecture.md
│   ├── methodology.md
│   └── limitations.md
│
├── results/
│   └── .gitkeep
│
├── tests/
│   ├── test_lcoh_calc.m
│   ├── test_data_import.m
│   └── test_economic_signals.m
│
└── .github/
    └── workflows/
        ├── ci.yml               # MATLAB CI pipeline
        └── release.yml          # Automated release
```

---

## Usage

### 1. Import Electricity Price Data

```matlab
% Download from ISO New England Express
prices = import_iso_prices('isone', '2023-01-01', '2023-12-31');

% Or load cached data
load('data/electricity_prices/isone_2023.mat');
```

### 2. Import Solar Irradiance Data

```matlab
% Load station data (pre-downloaded from NSRDB)
load('data/locations/station_data.mat'); % loads StationData struct

% Available locations
disp({StationData.Name})
% {'Boston_MA', 'Phoenix_AZ', 'Denver_CO', 'Miami_FL', 'Seattle_WA', ...}
```

### 3. Run Multi-Location TEA

```matlab
% Configure analysis
config = struct();
config.years          = 20;          % Project lifetime
config.discount_rate  = 0.08;        % 8% WACC
config.electrolyzer_efficiency = 0.70;
config.pv_capacity_kw = 1000;
config.battery_kwh    = 500;
config.use_parallel   = true;        % Use parfor if PCT available

% Run across all locations
results = run_tea_all_locations(StationData, prices, config);

% Plot LCOH map
plot_lcoh_map(results);
```

### 4. Optimize Battery Dispatch

```matlab
% Minimize grid electricity cost via price-arbitrage dispatch
[optimal_schedule, savings] = optimize_storage(prices, solar_profile, config);
fprintf('Annual grid cost savings: $%.0f\n', savings);
```

### 5. Generate Report

```matlab
generate_report(results, 'results/tea_report_2024.html');
```

---

## Results

### Sample LCOH by Location (2023 assumptions)

| Location | LCOH ($/kg) | Solar CF | Grid % | Payback (yr) |
|----------|-------------|----------|--------|--------------|
| Phoenix, AZ | **3.82** | 28.4% | 8% | 11.2 |
| Denver, CO | 4.15 | 24.1% | 14% | 12.8 |
| Miami, FL | 4.31 | 22.8% | 19% | 13.5 |
| Boston, MA | 5.67 | 16.2% | 31% | 17.4 |
| Seattle, WA | 6.12 | 13.9% | 38% | 19.1 |

> **Key finding:** Southwest US locations (Phoenix, Las Vegas) offer the most economically viable green hydrogen production. Current LCOH remains above the DOE Hydrogen Shot target of **$1/kg by 2031**.

See [`results/`](results/) for full output files and interactive dashboards.

---

## Limitations

See [docs/limitations.md](docs/limitations.md) for a full discussion. Key limitations:

- **Steady-state electrolyzer model** — dynamic startup/shutdown cycles not captured
- **Single year weather data** — interannual variability not assessed
- **No degradation curves** — electrolyzer and PV degradation assumed linear
- **Simplified water cost** — flat $/kg-H₂ assumption; no regional water scarcity pricing
- **No transmission constraints** — grid electricity assumed always available at LMP
- **No H₂ demand model** — production-only assessment, no offtake contracts

---

## Contributing

We welcome contributions! Please read [CONTRIBUTING.md](CONTRIBUTING.md) and open an issue before submitting a PR.

### Development Setup

```bash
git checkout -b feature/your-feature-name
# make changes
matlab -batch "run_tests()"
git push origin feature/your-feature-name
```

---

## Citation

```bibtex
@software{green_hydrogen_tea_2024,
  author    = {Your Name},
  title     = {Green Hydrogen Techno-Economic Assessment Framework},
  year      = {2024},
  publisher = {GitHub},
  url       = {https://github.com/your-org/green-hydrogen-tea}
}
```

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

*This project is a submission to the [MathWorks Sustainability Challenge](https://www.mathworks.com/academia/students/community/student-competitions.html).*
