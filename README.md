# FOOD-DELIVERY-USING-AUTONOMOUS-ROBOTS
This project was developed as my Bachelor's thesis in Economics and Digital Firms at the University of Brescia, defended on 30 October 2024.
The research addresses the risks and inefficiencies associated with human food delivery riders ,  including safety hazards, labour costs, and unreliable delivery times,  by simulating an alternative model in which autonomous delivery robots (inspired by Starship Technologies) use the existing public transit network to travel from restaurants across the city to customers, without a human operator. 
## Selam Mahmud Ali

# 🚇 Brescia Transit Network — Food Delivery Optimization

A mixed-integer programming model for scheduling last-mile food delivery orders across the Brescia metro and tram network. The system routes orders through the public transit graph, respects vehicle capacity constraints, and maximises the number of early-run deliveries.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Model Formulation](#model-formulation)
- [Configuration](#configuration)
- [Dependencies](#dependencies)
- [Notes & Limitations](#notes--limitations)

---

## Overview

This project combines geospatial network modelling with integer optimisation to solve a transit-based delivery scheduling problem in Brescia, Italy. Restaurants and customers are randomly distributed near transit stations; orders are routed through the metro/tram graph via Dijkstra's algorithm; and a Pyomo MIP model assigns orders to departure runs while respecting arc capacity and vehicle limits.

---

## Project Structure

```
brescia_transit_optimization/
│
├── brescia_transit_optimization.ipynb   # Main notebook (data + model combined)
└── README.md                            # This file
```

---

## Installation

### 1. Clone or download the project

```bash
git clone https://github.com/your-username/brescia-transit-optimization.git
cd brescia-transit-optimization
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows
```

### 3. Install Python dependencies

```bash
pip install osmnx geopy pyomo numpy matplotlib
```

### 4. Install the GLPK solver

GLPK must be available on your system PATH. Installation depends on your OS:

| OS | Command |
|----|---------|
| **macOS** | `brew install glpk` |
| **Ubuntu / Debian** | `sudo apt-get install glpk-utils` |
| **Windows** | Download from [winglpk.sourceforge.net](http://winglpk.sourceforge.net/) and add to PATH |

> Alternatively, you can use **CPLEX**, **Gurobi**, or **HiGHS** by changing the `SOLVER_NAME` variable in Section 8 of the notebook.

---

## Usage

Launch Jupyter and open the notebook:

```bash
jupyter notebook brescia_transit_optimization.ipynb
```

Then run all cells from top to bottom (`Kernel → Restart & Run All`).

---

## How It Works

The notebook is structured in 9 sequential sections:

| # | Section | Description |
|---|---------|-------------|
| 1 | Imports & Setup | All library imports |
| 2 | Network Definition | Metro (17 stops) and tram (23 stops) stations and arcs, including transfer arcs |
| 3 | Timetable | Peak (07:00–19:00, every 10 min) and off-peak (every 20 min) schedules |
| 4 | Graph & Dijkstra | Adjacency-list graph with shortest-path routing between stations |
| 5 | Restaurants & Customers | Points sampled from a normal distribution around each station |
| 6 | Orders & Matrix B | Order pairs filtered by distance (> 1.6 km); arc-usage binary matrix built via Dijkstra |
| 7 | Map Visualisation | OSMnx street map overlaid with metro line, tram line, restaurants, and customers |
| 8 | Optimisation Model | Pyomo `ConcreteModel` with binary variables, objective, and constraints |
| 9 | Results | Per-order assignment and per-arc usage printed to output |

---

## Model Formulation

### Decision Variable

$$y_{o,t} \in \{0, 1\} \quad \forall\, o \in \text{Orders},\; t \in T$$

$y_{o,t} = 1$ if order $o$ is assigned to departure run $t$.

### Objective — Maximise early delivery

$$\max \sum_{o \in \text{Orders}} \sum_{t \in T} \frac{19 \times 60 - t}{19 \times 60} \cdot y_{o,t}$$

Earlier runs receive a higher weight, incentivising the model to schedule deliveries as soon as possible.

### Constraints

**Arc capacity** — total demand on each arc at each time slot must not exceed 20 units:

$$\sum_{o \in \text{Orders}} B_{a,o} \cdot y_{o,t} \cdot d_o \leq 20 \quad \forall\, a \in \text{Arcs},\; t \in T$$

**Single assignment** — each order can be served in at most one run:

$$\sum_{t \in T} y_{o,t} \leq 1 \quad \forall\, o \in \text{Orders}$$

### Parameters

| Symbol | Description |
|--------|-------------|
| $B_{a,o}$ | 1 if arc $a$ is used by order $o$'s shortest path, 0 otherwise |
| $d_o$ | Demand (number of packages) for order $o$, drawn from $U[1, 4]$ |
| $T$ | Set of departure times (minutes from midnight), determined by peak/off-peak schedule |

---

## Configuration

The following parameters can be adjusted directly in the notebook:

| Parameter | Location | Default | Description |
|-----------|----------|---------|-------------|
| `peak_frequency` | Section 3 | `10` min | Run frequency during peak hours |
| `off_peak_frequency` | Section 3 | `20` min | Run frequency during off-peak hours |
| `sigma` | Section 5 | `0.01` | Spread of restaurants/customers around stations |
| `restaurants` cap | Section 5 | `10` | Max number of restaurants sampled |
| `customers` cap | Section 5 | `10` | Max number of customers sampled |
| Distance threshold | Section 6 | `1.6` km | Minimum order distance to be included |
| Arc capacity | Section 8 | `20` | Max demand units per arc per run |
| `SOLVER_NAME` | Section 8 | `'glpk'` | Solver to use (`glpk`, `cplex`, `gurobi`, `highs`) |
| `TIME_LIMIT` | Section 8 | `60` s | Solver time limit |

---

## Dependencies

| Library | Purpose |
|---------|---------|
| `osmnx` | Download and plot OpenStreetMap street networks |
| `geopy` | Geodesic (great-circle) distance between coordinates |
| `pyomo` | Algebraic modelling language for the MIP |
| `numpy` | Numerical arrays and random sampling |
| `matplotlib` | Plotting |
| `heapq` | Priority queue for Dijkstra's algorithm |
| `datetime` | Peak/off-peak time detection |
| GLPK / other solver | Solving the integer programme |

---

## Notes & Limitations

- **Randomness** — restaurants and customers are sampled stochastically; results will differ between runs. Fix `np.random.seed(...)` at the top of Section 5 for reproducibility.
- **Scale** — the model is limited to 10 restaurants and 10 customers by default to keep solve times short. Increasing these values will grow the number of orders and constraints significantly.
- **Arc capacity unit** — the capacity limit of 20 is set in abstract "demand units", not passengers. Adjust based on the real vehicle capacity of the transit line.
- **Transfer arcs** — two metro-to-tram interchange points are hardcoded: *San Faustino* ↔ *Tram 10* and *Stazione FS* ↔ *Tram 15*. Additional interchanges can be added in Section 2.
- **Network type** — the OSMnx download uses `network_type='all'`, which includes footpaths. Change to `'drive'` or `'bike'` if needed.

---



