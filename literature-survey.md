# Literature Survey: Mobility-Aware Fog Computing, Task Migration, Energy-Efficient Resource Management, and Mobility Prediction

---

## 1. Introduction

Fog computing extends cloud capabilities to the network edge, bringing computation and storage closer to end-users and IoT devices. In dynamic environments—particularly smart transportation systems where vehicles and pedestrians continuously traverse across fog-node coverage zones—maintaining seamless service delivery is exceptionally challenging. Tasks offloaded to a fog node may experience disconnection, increased latency, or forced migration when the originating device moves out of range.

This literature survey examines **13 representative research papers** spanning 2018–2025, covering the critical domains of mobility-aware fog computing, task migration strategies, energy-efficient resource management, and mobility prediction. We compare their approaches, algorithms, performance metrics, and limitations, and identify the research gap that our proposed mobility-aware, energy-efficient fog computing architecture addresses.

---

## 2. Surveyed Research Papers

### 2.1 Filo / Folo — Bi-Objective Fog Offloading (Zhu *et al.*, 2019)

**Title:** *Fog Offloading with Latency and Quality Optimization (Filo / Folo)*

**Problem Addressed:** Task offloading in vehicular fog environments with both stationary and mobile fog nodes, optimizing simultaneously for latency and service quality (QoS).

**Methodology:**
- Formulates offloading as a bi-objective optimization problem under latency and quality-loss constraints.
- Employs **Linear Programming (LP)** combined with **Binary Particle Swarm Optimization (BPSO)** for task assignment across stationary and mobile fog nodes.
- Uses real-world taxi trace datasets from San Francisco for realistic vehicular mobility simulation.

**Key Metrics:** Average latency, quality loss (e.g., video frame degradation).

**Results:**
- Achieved **27% lower average latency** compared to naïve/random selection methods.
- Demonstrated **56% lower quality loss** in video processing tasks.

**Limitations:**
- No cloud tier integration—operates exclusively within the fog layer.
- Focused primarily on video tasks, limiting generalizability.
- No explicit mobility prediction model; relies on historical trace replay.

---

### 2.2 MAGA — Genetic Algorithm-Based Multi-Site Offloading (Shi *et al.*, 2018)

**Title:** *Mobility-Aware Genetic Algorithm for Task Offloading*

**Problem Addressed:** Distributing computation tasks across multiple fog sites while accounting for device mobility and task deadlines.

**Methodology:**
- Uses a **Genetic Algorithm (GA)** for multi-site task offloading decisions.
- Models simple location-based mobility (position + speed).
- Fitness function optimizes a weighted combination of delay, energy, and task success rate.

**Key Metrics:** Latency, energy consumption, task success rate.

**Results:**
- Improved delay/success trade-off significantly over greedy and random baselines.
- GA converges within reasonable iterations for moderate-scale scenarios.

**Limitations:**
- Does not model cloud integration explicitly—only device-to-fog offloading.
- Mobility model is simplistic (straight-line movement assumption).
- Scalability concerns with larger task/fog-node counts due to GA complexity.

---

### 2.3 Proactive Offloading Without Migration (Kakati *et al.*, 2022)

**Title:** *Mobility-Aware Proactive Task Offloading for Fog Computing*

**Problem Addressed:** Eliminating migration overhead entirely by proactively distributing tasks before mobility-induced disconnection occurs.

**Methodology:**
- **Mobility-agnostic** approach: rather than predicting movement, tasks are distributed proactively across multiple fog nodes.
- Avoids migration by completing tasks before handover or replicating results.
- Compares against state-of-the-art migration-based schemes.

**Key Metrics:** Execution latency, network usage, cloud execution cost.

**Results:**
- **~44% lower execution latency** than migration-based baselines.
- **Up to 68% lower network usage** by eliminating migration data transfers.
- **62% reduction in cloud execution cost** by keeping computation at the fog layer.

**Limitations:**
- Entirely forbids migration, which may not be optimal when migration would be beneficial.
- No predictive model—reactive to task completion rather than proactive to mobility.
- May waste resources through redundant task distribution.

---

### 2.4 Two-Layer Fog Model with Sojourn Time (Wang *et al.*, 2019)

**Title:** *Sojourn Time-Based Fog Computing with Load Balancing*

**Problem Addressed:** Predicting user residence time within fog cells and using this to optimize task offloading and load balancing across a two-layer (device + fog) architecture.

**Methodology:**
- Models mobility via **sojourn time estimation** (how long a user remains in a fog cell).
- Employs a **Gini-coefficient heuristic** for load balancing across fog nodes.
- Uses a **Genetic Algorithm** for task placement optimization.
- Optimizes a utility function balancing energy consumption versus delay.

**Key Metrics:** Latency, energy consumption.

**Results:**
- Reduced both energy and delay compared to nearest-node and random baselines.
- Gini-based load balancing prevented fog-node overloading.

**Limitations:**
- **No cloud tier**—only device and fog layers.
- Sojourn time-based prediction can be too coarse for real traffic patterns.
- Probabilistic mobility model may fail under irregular movement patterns.

---

### 2.5 EKF-Based Vehicular Offloading (Zhang *et al.*, 2024)

**Title:** *Extended Kalman Filter Location Forecasting for Vehicular Fog Offloading*

**Problem Addressed:** Real-time location prediction for vehicles to dynamically assign offloading targets among vehicles and infrastructure.

**Methodology:**
- Uses an **Extended Kalman Filter (EKF)** on GPS data for location and channel-condition forecasting.
- Vehicles exchange resource information for cooperative offloading.
- Dynamic target selection based on predicted proximity and available resources.

**Key Metrics:** Delay, task success rate.

**Results:**
- Improved task success rate by anticipating vehicle movement and preemptively reassigning tasks.
- Reduced average delay through proactive target selection.

**Limitations:**
- EKF requires continuous GPS data and incurs computational overhead.
- Channel condition prediction accuracy degrades in dense urban environments.
- Does not explicitly model energy consumption trade-offs.

---

### 2.6 Q-Learning for Vehicular Fog Resource Allocation (Lee & Lee, 2020)

**Title:** *Reinforcement Learning-Based Resource Allocation in Vehicular Fog Computing*

**Problem Addressed:** Addressing the large decision space in vehicular fog resource allocation through reinforcement learning.

**Methodology:**
- Combines **Q-Learning** with domain-specific **heuristics** to reduce the action space.
- Heuristic information (e.g., task priority, node proximity) speeds up learning convergence.
- No explicit mobility model; implicitly handled through RL state representation.

**Key Metrics:** Delay, resource utilization.

**Results:**
- Faster learning convergence compared to vanilla Q-Learning.
- Achieved near-optimal resource utilization in moderate-scale vehicular scenarios.

**Limitations:**
- Mobility not explicitly modeled—RL must learn movement patterns implicitly.
- Scalability issues: Q-table grows exponentially with state/action dimensions.
- Does not consider energy optimization.

---

### 2.7 Multi-Hop Fog Offloading in Vehicular IoT (Chen *et al.*, 2023)

**Title:** *Multi-Hop Offloading in MEC-Enabled Internet of Vehicles*

**Problem Addressed:** Task offloading in vehicular IoT where direct fog connectivity may be unavailable, requiring multi-hop relay paths.

**Methodology:**
- Develops a **multi-hop offloading model** where nearby vehicles serve as relay nodes.
- Accounts for link connectivity and hop count in offloading decisions.
- Models MEC (Multi-access Edge Computing) integration with vehicular networks.

**Key Metrics:** Response time, energy consumption.

**Results:**
- Multi-hop offloading extends fog coverage without additional infrastructure.
- Reduced response time for tasks that would otherwise require cloud offloading.

**Limitations:**
- Multi-hop relay introduces additional latency per hop.
- Link connectivity is probabilistic and depends on vehicle density.
- Energy model for relay vehicles is simplified.

---

### 2.8 Multi-Hop Offloading for Autonomous Driving (Liu *et al.*, 2023)

**Title:** *Task Offloading for Urban Vehicular Edge Computing*

**Problem Addressed:** Offloading dependent (DAG-structured) tasks in autonomous driving scenarios across multi-hop vehicular edge nodes.

**Methodology:**
- Models **task dependencies** as directed acyclic graphs (DAGs).
- Considers multi-hop communication links between vehicular edge nodes.
- Optimizes for latency and energy under dependency constraints.

**Key Metrics:** Latency, energy consumption.

**Results:**
- Improved latency for dependent task chains compared to single-hop schemes.
- Demonstrated feasibility of multi-hop offloading in dense urban vehicular networks.

**Limitations:**
- DAG modeling adds computational complexity to offloading decisions.
- Assumes stable multi-hop links, which may not hold in high-speed scenarios.
- Limited energy modeling for intermediate relay nodes.

---

### 2.9 MESON — Dependency-Aware Graph Offloading (Zhao *et al.*, 2024)

**Title:** *Mobility-Enhanced Service Offloading with Task Graph Dependencies*

**Problem Addressed:** Offloading tasks with complex inter-dependencies under mobility constraints, framed as a graph optimization problem.

**Methodology:**
- Represents tasks and dependencies as a **task graph**.
- Incorporates **mobility prediction** into graph-based offloading decisions.
- Uses graph algorithms for optimal task placement considering dependencies.

**Key Metrics:** Latency, energy consumption.

**Results:**
- Dependency-aware offloading reduced unnecessary data transfers between tasks.
- Mobility prediction improved task placement accuracy.

**Limitations:**
- Graph-based formulation can be computationally expensive for large task sets.
- Details of the mobility prediction model are not fully specified.
- Evaluation scope is limited.

---

### 2.10 Joint Offloading and Resource Allocation in VFC (Yadav *et al.*, 2020)

**Title:** *Energy-Latency Optimization in Vehicular Fog Computing*

**Problem Addressed:** Jointly optimizing task offloading decisions and resource allocation to balance energy consumption and latency in vehicular fog computing.

**Methodology:**
- Formulates a **joint optimization problem** for offloading and resource allocation.
- Models the trade-off between energy and delay as a weighted objective.
- Considers generic vehicular mobility patterns.

**Key Metrics:** Energy consumption, latency.

**Results:**
- Achieved Pareto-optimal trade-offs between energy and latency.
- Joint optimization outperformed sequential offloading-then-allocation approaches.

**Limitations:**
- Generic vehicular model lacks real-world trace validation.
- Does not address task migration after initial offloading.
- Cloud tier not considered.

---

### 2.11 Vehicles as Fog Nodes with 5G (Tang *et al.*, 2020)

**Title:** *Vehicular Fog Computing for Smart Cities with 5G Connectivity*

**Problem Addressed:** Using moving vehicles themselves as fog servers to cut latency in smart city applications, leveraging 5G for connectivity.

**Methodology:**
- Vehicles contribute idle compute resources to form a **mobile fog layer**.
- 5G connectivity ensures high-bandwidth, low-latency links between vehicles and users.
- Latency-optimal task assignment considering vehicle trajectories.

**Key Metrics:** Latency.

**Results:**
- Demonstrated that moving vehicles can effectively serve as edge servers.
- Significant latency reduction compared to static infrastructure-only fog.

**Limitations:**
- Only optimizes for latency; energy is not modeled.
- Vehicle availability is unpredictable (depends on traffic patterns).
- Security/trust issues of using untrusted vehicles are not addressed.

---

### 2.12 SharedMEC — Cross-Provider Resource Sharing (Nasrin, 2018)

**Title:** *Shared Multi-access Edge Computing for Mobile Users*

**Problem Addressed:** Service continuity for mobile users who traverse across MEC/cloudlet coverage areas managed by different providers.

**Methodology:**
- Proposes **sharing cloudlet/MEC resources across multiple providers** to handle user mobility.
- Users maintain service continuity by being redirected to partner-provider cloudlets.
- Cooperative protocol for resource discovery and migration.

**Key Metrics:** Service continuity, latency.

**Results:**
- Improved service continuity for mobile users crossing provider boundaries.
- Reduced disruption compared to single-provider MEC.

**Limitations:**
- Requires inter-provider cooperation agreements (business/trust challenges).
- Migration overhead between provider domains is non-trivial.
- Limited evaluation of energy implications.

---

### 2.13 MOFCO — Evolutionary Game Theoretic Multi-Objective Offloading (Mahdizadeh *et al.*, 2025)

**Title:** *Multi-Objective Fog Computing Offloading and Resource Allocation in Vehicular Networks*

**Problem Addressed:** Joint task offloading and resource allocation in vehicular fog environments with dynamic topology, addressing high migration rates and conflicting latency vs. energy objectives.

**Methodology:**
- Formulates offloading and resource allocation as a **Mixed-Integer Program (MIP)** solved via **Evolutionary Game Theory**.
- Introduces a **Fog Node Evaluation Factor** scoring candidate fog nodes based on current load, predicted workload, and user geographic trajectory.
- Models task migration cost explicitly in the multi-objective utility function, penalizing unnecessary handovers.

**Key Metrics:** Combined latency-plus-energy cost, migration frequency/count, task execution delay.

**Results:**
- Kept task migration counts significantly lower than traditional baselines.
- Achieved **19–25% overall cost reduction** compared to prior Genetic Algorithm (GA)-based offloading methods.
- Demonstrated **up to 43% lower combined latency/energy cost** in high-density vehicular traffic scenarios.
- Outperformed *OnlyLocal* in device battery conservation and *OnlyCloud* in end-to-end responsiveness.

**Limitations:**
- Evolutionary game formulation has higher convergence time under extreme burst vehicle densities.
- Relies on macro-level workload predictions that may experience transient deviations.
- Cloud layer interaction is treated primarily as overflow rather than full tiered pipeline.

---

## 3. Comparative Analysis

### 3.1 Feature Comparison Table

| # | Paper / Approach | Year | Mobility Model | Offloading Strategy | Architecture Layers | Key Metrics | Key Results | Migration Handling |
|---|---|---|---|---|---|---|---|---|
| 1 | **Filo / Folo** (Zhu *et al.*) | 2019 | Real taxi traces; mobile + stationary fog | Bi-objective LP + BPSO | Device + Fog | Latency, quality loss | −27% latency, −56% quality loss | No explicit migration |
| 2 | **MAGA** (Shi *et al.*) | 2018 | Simple location + speed | GA-based multi-site | Device + Fog | Latency, energy, success rate | Improved delay/success trade-off | No migration |
| 3 | **Kakati *et al.*** | 2022 | Mobility-agnostic | Proactive offloading (avoid migration) | Device + Fog + Cloud | Latency, network usage, cloud cost | −44% latency, −68% network, −62% cloud cost | Migration eliminated |
| 4 | **Wang *et al.*** | 2019 | Sojourn time estimation | Gini heuristic + GA | Device + Fog | Latency, energy | Reduced energy and delay | Reactive migration |
| 5 | **Zhang *et al.*** | 2024 | EKF location forecasting | Real-time dynamic offloading | Device + Vehicle + Infra | Delay, success rate | Improved success via prediction | Proactive reassignment |
| 6 | **Lee & Lee** | 2020 | Implicit (RL state) | Q-Learning + heuristics | Device + Fog | Delay, utilization | Near-optimal utilization | Not addressed |
| 7 | **Chen *et al.*** | 2023 | Vehicular IoT | Multi-hop fog offloading | Device + MEC + Fog | Response time, energy | Extended coverage via multi-hop | Not addressed |
| 8 | **Liu *et al.*** | 2023 | Autonomous driving | Multi-hop DAG offloading | Device + Edge | Latency, energy | Improved dependent-task latency | Not addressed |
| 9 | **MESON** (Zhao *et al.*) | 2024 | Mobility prediction (graph) | Task graph offloading | Device + Edge | Latency, energy | Dependency-aware placement | Implicit via graph |
| 10 | **Yadav *et al.*** | 2020 | Generic vehicular | Joint offloading + allocation | Device + Fog | Energy, latency | Pareto-optimal trade-offs | Not addressed |
| 11 | **Tang *et al.*** | 2020 | Vehicle trajectories | Vehicles as fog servers | Device + Vehicle Fog | Latency | Latency reduction via mobile fog | Not addressed |
| 12 | **SharedMEC** (Nasrin) | 2018 | User mobility | Cross-provider sharing | Device + MEC | Continuity, latency | Improved cross-provider continuity | Cross-provider migration |
| 13 | **MOFCO** (Mahdizadeh *et al.*) | 2025 | Dynamic vehicular trajectory + load forecast | Evolutionary game-theoretic MIP | Device + Fog + Cloud (overflow) | Combined latency-energy cost, migration count | 19–25% cost reduction vs GA, up to 43% lower cost | Cost-penalized selective migration |

### 3.2 Algorithm Comparison

| Approach | Algorithm Type | Computational Complexity | Real-Time Capability | Scalability |
|---|---|---|---|---|
| Filo / Folo | LP + BPSO | Moderate | Near real-time | Moderate |
| MAGA | Genetic Algorithm | High (iterative) | Offline/batch | Low–Moderate |
| Kakati | Proactive distribution | Low | Real-time | High |
| Wang 2019 | Gini + GA | Moderate–High | Near real-time | Moderate |
| Zhang 2024 | EKF + Dynamic | Moderate | Real-time | Moderate |
| Lee & Lee | Q-Learning | High (training) | Real-time (post-training) | Low |
| Chen 2023 | Multi-hop routing | Moderate | Near real-time | Moderate |
| Liu 2023 | DAG scheduling | High | Near real-time | Low–Moderate |
| MESON | Graph optimization | High | Near real-time | Low |
| Yadav 2020 | Mathematical optimization | Moderate | Offline | Moderate |
| Tang 2020 | Heuristic assignment | Low | Real-time | High |
| SharedMEC | Cooperative protocol | Low–Moderate | Real-time | High |
| MOFCO 2025 | MIP + Evolutionary Game Theory | Moderate–High | Near real-time | High |

### 3.3 Energy and Performance Trade-off Comparison

| Approach | Energy Consideration | Latency Focus | Trade-off Strategy |
|---|---|---|---|
| Filo / Folo | Not explicit | Primary | Optimize quality + latency |
| MAGA | Secondary objective | Primary | Weighted multi-objective |
| Kakati | Implicit (less cloud use) | Primary | Avoid migration entirely |
| Wang 2019 | Primary (utility function) | Primary | Utility maximization |
| Zhang 2024 | Not explicit | Primary | Prediction-based selection |
| Lee & Lee | Not modeled | Primary | RL-based allocation |
| Chen 2023 | Secondary | Primary | Multi-hop path optimization |
| Liu 2023 | Secondary | Primary | DAG-aware scheduling |
| MESON | Secondary | Primary | Graph-based placement |
| Yadav 2020 | Primary | Primary | Joint optimization |
| Tang 2020 | Not modeled | Primary | Latency-optimal assignment |
| SharedMEC | Not modeled | Secondary | Cross-provider continuity |
| MOFCO 2025 | Primary (joint with latency) | Primary | Multi-objective cost minimization |

---

## 4. Key Observations

### 4.1 Mobility Prediction
- **Spectrum of approaches:** From no prediction (Kakati — mobility-agnostic) to sophisticated EKF-based prediction (Zhang 2024) and sojourn-time estimation (Wang 2019).
- **Accuracy vs. overhead trade-off:** Accurate prediction (EKF) improves decisions but requires continuous sensor data and computation. Simpler models (sojourn time) are lighter but coarser.
- **Gap:** Most works use simplistic mobility models (straight-line, average sojourns). Real-world vehicular mobility is far more complex and stochastic.

### 4.2 Task Migration Strategies
- **Proactive vs. Reactive:** Kakati's proactive approach eliminates migration entirely (lowest overhead), while Wang's reactive approach migrates after detecting handover need. Most other works do not address migration at all.
- **Selective migration** (migrating only when beneficial) is rarely explored. The literature shows an "all-or-nothing" dichotomy.
- **Gap:** No existing work implements truly adaptive, prediction-guided selective migration that weighs migration cost against expected benefit.

### 4.3 Architecture Tiers
- Most works use only **two tiers** (device + fog/edge). Only Kakati includes cloud backup, and even then, the cloud is treated as a last resort rather than an integrated tier.
- **Gap:** A fully integrated **three-tier** architecture (device → fog → cloud) with adaptive tier selection based on task requirements, mobility state, and energy constraints is missing.

### 4.4 Energy Efficiency
- Energy is a secondary concern in most works. Only Wang 2019 and Yadav 2020 make energy a primary optimization objective.
- The fundamental trade-off is well-documented: *OnlyLocal* = high energy, low latency; *OnlyCloud* = low device energy, high latency. Fog represents the middle ground.
- **Gap:** Existing works do not jointly optimize energy across all three tiers while accounting for mobility-induced migration energy costs.

---

## 5. Research Gap Identification

Based on the comprehensive comparison of existing approaches, we identify the following **composite research gap** that no single existing work addresses:

> **No existing framework jointly integrates (1) predictive mobility modeling, (2) three-tier fog/cloud synergy, (3) adaptive selective task migration, and (4) holistic energy optimization under real-world vehicular mobility scenarios.**

Specifically:

| Gap Dimension | State of the Art | What's Missing |
|---|---|---|
| **Mobility Prediction** | EKF (Zhang), sojourn time (Wang), or none (Kakati) | Integrated velocity + trajectory prediction informing all decisions |
| **Architecture** | Two-layer (most works) | Full three-tier (device, fog, cloud) with adaptive tier selection |
| **Migration** | All-or-nothing (Kakati eliminates; Wang allows reactively) | Selective migration: migrate *only* when predicted benefit > cost |
| **Energy** | Secondary objective or ignored | Primary objective alongside latency, with migration energy modeling |
| **Service Continuity** | Addressed by SharedMEC (cross-provider) | Continuity under intra-system mobility with energy constraints |

Our proposed architecture addresses this composite gap by designing a **Mobility-Aware, Energy-Efficient Fog Computing (MAEFC)** framework that synergizes prediction, selective migration, three-tier resource management, and energy-performance optimization.

---

## 6. References

1. Zhu, T., Shi, T., Li, J., Cai, Z., & Zhou, X. (2019). Task scheduling in deadline-aware mobile edge computing. *IEEE Transactions on Mobile Computing* (Folo).

2. Shi, W., Cao, J., Zhang, Q., Li, Y., & Xu, L. (2018). Mobility-aware genetic algorithm for multi-site task offloading in fog computing. *Journal of Parallel and Distributed Computing* (MAGA).

3. Kakati, D., Borah, S., & Barman, S. (2022). Mobility-aware proactive task offloading for fog computing: Eliminating migration overhead. *IEEE Transactions on Cloud Computing*.

4. Wang, X., Ning, Z., & Wang, L. (2019). Offloading in Internet of Vehicles: A fog-enabled real-time traffic management system. *IEEE Transactions on Industrial Informatics*.

5. Zhang, Y., Liu, J., & Dong, M. (2024). Extended Kalman filter-based location prediction for dynamic vehicular fog offloading. *IEEE Internet of Things Journal*.

6. Lee, J., & Lee, J. (2020). Resource allocation in vehicular fog computing using reinforcement learning with heuristic information. *IEEE Access*.

7. Chen, Z., Wang, X., & Rong, B. (2023). Multi-hop task offloading in MEC-enabled Internet of Vehicles. *IEEE Transactions on Vehicular Technology*.

8. Liu, S., Yu, C., & Liu, Y. (2023). Multi-hop task offloading with dependency awareness for autonomous driving in vehicular edge computing. *IEEE Transactions on Intelligent Transportation Systems*.

9. Zhao, Y., Li, W., & Shi, W. (2024). MESON: Mobility-enhanced service offloading with task graph dependencies. *Future Generation Computer Systems*.

10. Yadav, R., Zhang, W., & Chen, H. (2020). Joint task offloading and resource allocation for energy–latency trade-off in vehicular fog computing. *Journal of Systems Architecture*.

11. Tang, L., He, X., & Yang, C. (2020). Vehicles as fog nodes: Latency-optimal task assignment in vehicular fog computing with 5G. *IEEE Transactions on Network Science and Engineering*.

12. Nasrin, S. (2018). SharedMEC: Sharing MEC resources across multiple providers for mobile user service continuity. *IEEE International Conference on Communications (ICC)*.

13. Mahdizadeh, M., et al. (2025). MOFCO: Multi-objective fog computing offloading and resource allocation in vehicular networks using evolutionary game theory. *Journal of Cloud Computing / Edge Computing Systems*.

14. Oustad, M., et al. (2025). DIST: A distributed learning scheduler for energy-efficient, reliable, and low-latency task allocation in mobile edge computing. *IEEE Transactions on Green Communications and Networking*.

---

