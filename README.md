# Mobility-Aware, Energy-Efficient Fog Computing (MAEFC)

**Course:** CS4001 - Cloud and Fog Computing for IoT  
**Assessment:** CIA - 2  
**Author:** Priyajit Biswal  
**Registration No:** 23011102068  
**Repository:** [https://github.com/priyajitbiswal/mobility-aware-fog-simulation](https://github.com/priyajitbiswal/mobility-aware-fog-simulation)

---

## Overview

In intelligent transportation systems, connected vehicles move rapidly (40–100 km/h) through roadside fog node (RSU) coverage zones (150–300 m radius). This rapid movement causes frequent disconnections, high migration overheads, and orphaned tasks.

This repository implements the **MAEFC (Mobility-Aware, Energy-Efficient Fog Computing)** framework, a three-tier architecture (Vehicle $\rightarrow$ Roadside Fog $\rightarrow$ Remote Cloud) that combines:
1. **Kinematic Dwell-Time Estimation:** Predicts vehicle cell residence time based on velocity and distance to the cell boundary ($T_{\text{dwell}} = d_{\text{boundary}} / v$).
2. **Selective Migration:** Implements a cost-benefit decision rule to migrate tasks *if and only if* the latency and failure prevention benefits outweigh memory transfer costs.
3. **Multi-Criteria Fog Selection:** Balances proximity, remaining dwell time, and RSU workload to prevent intersection hotspots.

---

## Project Structure

```text
├── simulation.py              # Main Python simulation & Monte Carlo benchmark engine
├── Cloud-Computing-CIA2.pdf   # Complete CIA-2 Literature Survey & Architecture Report
├── Cloud-Computing-CIA2.docx  # Word document format of report
├── ps.txt                     # Original assignment problem statement
├── README.md                  # Project documentation
└── results/                   # Generated evaluation plots (300 DPI PNGs)
    ├── fig1_latency_vs_speed.png
    ├── fig2_energy_breakdown.png
    ├── fig3_service_continuity.png
    └── fig4_migration_comparison.png
```

---

## Key Results Summary

From 1,000 Monte Carlo simulation runs across varying vehicular speed regimes and road scenarios:

| Metric | Only Local | Standard Fog (No Migr.) | Only Cloud | MAEFC (Proposed) |
| :--- | :---: | :---: | :---: | :---: |
| **Average Latency** | 25.0 ms | 45–140 ms *(Packet drops)* | 180–185 ms | **35–55 ms** |
| **Total Energy / Task** | 284 mJ | 162 mJ | 89 mJ | **125 mJ** |
| **Service Continuity** | 100% | 65% | 95% | **96%** |
| **Device Battery Drain** | Extreme (254 mJ) | Low (22 mJ) | Minimal (6 mJ) | **Balanced (34 mJ)** |

- **Selective Migration:** Cuts unnecessary migrations down to **10 per 100 tasks** (compared to 36/100 for greedy nearest-node).
- **Service Continuity:** Achieves **96.5%** on linear highways and **92.4%** during abrupt turns/lane changes.

---

## How to Run the Simulation

### Prerequisites
Make sure you have Python 3.8+ installed along with `numpy` and `matplotlib`:

```bash
pip install numpy matplotlib
```

### Run
Execute the main simulator:

```bash
python simulation.py
```

The script will:
1. Simulate 1,000 tasks across the 7-RSU highway corridor.
2. Print comparative performance tables for migration strategies, fog selection policies, speed regimes, and road scenarios.
3. Automatically update the publication plots in the `results/` folder.
