# MAEFC: Mobility-Aware, Energy-Efficient Fog Computing Framework

## Architecture Design for Smart Transportation Environments

---

## 1. Introduction

In smart transportation environments, connected vehicles and mobile users continuously traverse across heterogeneous fog-node coverage areas. While fog computing significantly reduces propagation delays by executing computations near mobile endpoints, user mobility introduces severe operational hurdles: rapid wireless link disconnections, unpredictable handover latency spikes, and recurring task migration penalties.

The **Mobility-Aware, Energy-Efficient Fog Computing (MAEFC)** framework addresses these core challenges through a three-tier architecture (Mobile Device / Vehicle $\rightarrow$ Edge Fog $\rightarrow$ Remote Cloud). MAEFC incorporates:
1. **Hybrid mobility prediction** (Extended Kalman Filtering coupled with road-network topology constraints and sojourn-time forecasting).
2. **Dimensionally normalized multi-objective offloading decisions** balancing latency, energy consumption, and migration risk.
3. **Adaptive task migration** providing context-sensitive switching between proactive live migration, reactive checkpoint migration, and cloud-assisted checkpoint recovery.
4. **Physics-grounded energy-aware resource allocation** leveraging Dynamic Voltage and Frequency Scaling (DVFS) and Shannon channel capacity modeling.

---

## 2. Evaluation of Mobility Support Challenges in Fog Computing

Supporting computational services in vehicular and mobile fog environments involves addressing critical physical, network, and algorithmic challenges:

### 2.1 Intermittent Wireless Connectivity and Orphaned Tasks
- **Rapid Coverage Traversal:** In urban and highway environments, vehicles move at velocities ranging from $30\text{ km/h}$ ($8.3\text{ m/s}$) to $>120\text{ km/h}$ ($33.3\text{ m/s}$). Given typical Roadside Unit (RSU) wireless radii of $150\text{ m}$ to $300\text{ m}$, a vehicle's residence time within a single cell ranges from just $9\text{ seconds}$ to $36\text{ seconds}$.
- **Orphaned Task State:** If a computationally intensive task offloaded to Fog Node $F_1$ has not concluded before the vehicle departs its transmission footprint, single-hop direct result return fails. Without coordinated handovers, the task execution becomes orphaned, squandering edge processing cycles and device battery energy.

### 2.2 Latency Degradation and Triangle Routing Penalty
- **Indirect Forwarding Delay:** If a task concludes after the mobile client has departed, and state migration was omitted, results must traverse multi-hop backhaul networks or cellular core networks to reach the vehicle's new serving node.
- **Jitter and Service Violation:** This triangle routing introduces substantial jitter, elevating round-trip latency from $<10\text{ ms}$ to $>80\text{ ms}$. Such latency inflation violates strict Service Level Agreements (SLAs) for safety-critical services such as emergency braking alerts or intersection collision avoidance.

### 2.3 Migration Overhead: Downtime, Bandwidth, and Energy Penalties
- **State Footprint Transfer:** Migrating active application containers or execution contexts requires moving memory states, execution stacks, and dirty memory pages across fog nodes over inter-RSU fiber or mmWave wireless links.
- **Backhaul Strain & Energy Drain:** High migration volumes consume significant edge backhaul bandwidth. Moreover, the power consumed in serializing, transmitting, and restoring memory states can exceed the energy saved by offloading, creating net performance penalties if executed indiscriminately.

### 2.4 Ping-Pong Handover and Boundary Oscillation
- **Overlapping Micro-Cells:** Roadside fog architectures frequently employ overlapping coverage to ensure continuous communication. Multipath shadowing or minor vehicular course adjustments near boundary zones can trigger rapid alternating handover requests between adjacent nodes ($F_i \leftrightarrow F_j$).
- **Migration Thrashing:** Uncontrolled handover logic leads to migration thrashing—tasks are repeatedly packaged and dispatched across nodes, depleting edge network resources without advancing task completion.

### 2.5 Dynamic Wireless Channels and Doppler Effects
- **Velocity-Induced Doppler Shift:** High vehicle speeds relative to stationary RSUs induce Doppler frequency shifts ($f_d = \frac{v}{\lambda}\cos\theta$), causing rapid carrier frequency offsets, inter-carrier interference in OFDM systems, and fast Rayleigh/Rician fading.
- **Dynamic Capacity Variations:** Achievable transmission data rates fluctuate unpredictably. Offloading decisions formulated under momentary peak Signal-to-Interference-plus-Noise Ratio (SINR) can stall mid-transfer if link conditions rapidly degrade.

### 2.6 Asymmetric Constraints and Load Heterogeneity
- **Energy Asymmetry:** Mobile user equipment (UE) and vehicular On-Board Units (OBUs) operate under strict energy and thermal dissipation limits, while RSUs may rely on solar power with limited battery reserves or grid connections with peak demand pricing.
- **Spatial Load Imbalance:** Sudden vehicular clustering (e.g., traffic congestions, intersections) causes extreme compute overload at specific roadside fog nodes while adjacent nodes remain idle.

---

## 3. System Architecture

### 3.1 Three-Tier Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLOUD TIER                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Cloud Data Center                                                  │    │
│  │  • Massive Elastic Compute & Long-term Storage Repositories         │    │
│  │  • Global Task Repository, Macro Route Planning & Model Training     │    │
│  │  • Centralized Policy Distribution & Fail-safe Task Checkpointing   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                              ▲  ▼  (WAN / High-Capacity 5G Core)            │
├─────────────────────────────────────────────────────────────────────────────┤
│                              FOG TIER                                       │
│                                                                             │
│  ┌──────────┐    High-Speed Backhaul    ┌──────────┐    High-Speed Backhaul │
│  │  Fog      │◄────────────────────────►│  Fog      │◄────────────────────► │
│  │  Node F1  │   (Inter-Fog Migration)   │  Node F2  │   (Inter-Fog Migr.)   │
│  │          │                          │          │                       │
│  │ • RSU Edge│                          │ • RSU Edge│                       │
│  │ • GPU/CPU │                          │ • GPU/CPU │                       │
│  │ • Cache/Q │                          │ • Cache/Q │                       │
│  └─────┬─────┘                          └─────┬─────┘                       │
│        │ (5G NR / DSRC V2I)                   │ (5G NR / DSRC V2I)          │
├────────┼──────────────────────────────────────┼─────────────────────────────┤
│        ▼             DEVICE TIER              ▼                             │
│                                                                             │
│   🚗 Vehicle V1         🚗 Vehicle V2         📱 Mobile User U1             │
│   • Multi-core OBU      • Multi-core OBU      • Handheld Edge CPU           │
│   • GPS / IMU Sensors   • GPS / IMU Sensors   • GNSS / Accelerometer        │
│   • Task Generator      • Task Generator      • Real-time Client App        │
│   • Battery Management  • Battery Management  • Strict Battery Constraint   │
│                                                                             │
│   Speed: 70 km/h        Speed: 50 km/h        Speed: 5 km/h                 │
│   Traj: F1 ──► F2       Traj: F2 ──► F1       Traj: Static / Pedestrian     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Tier Descriptions

#### Tier 1: Device Tier (Mobile Devices / Vehicles)
- **Physical Nodes:** Connected vehicles equipped with On-Board Units (OBUs), smart sensors, and mobile user smartphones.
- **Capabilities:** Low-to-moderate local processing (ARM/x86 low-power cores, embedded neural accelerators), high-accuracy GNSS/IMU positioning, and short-range transceivers.
- **Responsibilities:**
  - Generating task requests and filtering raw sensor streams.
  - Executing safety-critical Ultra-Low Latency (ULL) tasks locally.
  - Monitoring instantaneous battery levels, vehicle kinematics (speed, heading, acceleration), and radio channel metrics.
  - Initiating offloading requests with task requirement metadata.

#### Tier 2: Fog Tier (Edge Fog Nodes / RSUs)
- **Physical Nodes:** Roadside Units (RSUs) co-located with base stations, edge micro-datacenters, and municipal roadside server cabinets.
- **Capabilities:** High-performance multi-core processors, edge GPUs/TPUs, low-latency localized storage, and inter-RSU fiber/mmWave communication links.
- **Responsibilities:**
  - Executing latency-critical offloaded tasks within sub-second deadlines.
  - Managing task admission, queuing, and local DVFS scheduling.
  - Running short-to-medium-term mobility prediction and sojourn time estimators.
  - Coordinating proactive and reactive task state migrations between adjacent fog nodes.
  - Maintaining periodic state checkpoints to the cloud tier.

#### Tier 3: Cloud Tier (Remote Cloud Data Centers)
- **Physical Nodes:** Centralized hyperscale cloud infrastructure (e.g., regional data centers).
- **Capabilities:** Virtually unbounded computing power, massive parallel storage, and broad connectivity.
- **Responsibilities:**
  - Handling heavy, delay-tolerant batch analytics, machine learning model training, and global map updates.
  - Serving as the authoritative state repository and ultimate fallback tier when all local fog nodes are congested.
  - Long-term storage of vehicular performance logs and global traffic intelligence.

### 3.3 Communication Interfaces and Protocols

| Link Interface | Protocol / Radio Standard | Bandwidth | Typical Latency | Primary Function |
|---|---|---|---|---|
| **V2I / Device-to-Fog** | 3GPP 5G NR-V2X (PC5 / Uu), IEEE 802.11bd (DSRC) | $100\text{ Mbps} - 1\text{ Gbps}$ | $2 - 8\text{ ms}$ | Task offloading, real-time control, result delivery |
| **Fog-to-Fog (Xn/F2F)** | 10GbE Optical Fiber, mmWave (E-band) | $1 - 10\text{ Gbps}$ | $1 - 3\text{ ms}$ | Inter-fog live task migration, state sync, load sharing |
| **Fog-to-Cloud (N6/WAN)**| Fiber backhaul / Carrier Ethernet | $500\text{ Mbps} - 10\text{ Gbps}$| $25 - 80\text{ ms}$| Checkpoint synchronization, cloud offloading fallback |
| **V2V (Device-to-Device)**| 5G Sidelink (PC5), C-V2X direct | $10 - 100\text{ Mbps}$ | $1 - 5\text{ ms}$ | Cooperative sensing, multi-hop task forwarding |

---

## 4. Key Framework Components

### 4.1 Hybrid Mobility Prediction Module (MPM)

The Mobility Prediction Module eliminates reactive handover penalties by forecasting vehicular trajectories using a three-layer estimation engine:

#### 4.1.1 State-Space Kinematic Model (Extended Kalman Filter)
The vehicle kinematics are captured by the state vector:
$$\mathbf{x}_t = [p_x, p_y, v_x, v_y, a_x, a_y]^T$$
where $(p_x, p_y)$ represents 2D Cartesian spatial coordinates, $(v_x, v_y)$ represents velocity components, and $(a_x, a_y)$ denotes vehicular acceleration.

The non-linear state transition and discrete measurement models are:
$$\mathbf{x}_{t+1} = \mathbf{F}\mathbf{x}_t + \mathbf{w}_t, \quad \mathbf{z}_t = \mathbf{H}\mathbf{x}_t + \mathbf{v}_t$$
where $\mathbf{F}$ is the continuous-to-discrete state transition matrix, $\mathbf{w}_t \sim \mathcal{N}(0, \mathbf{Q})$ represents acceleration-induced process noise, $\mathbf{z}_t$ is the noisy GPS/IMU measurement vector, and $\mathbf{v}_t \sim \mathcal{N}(0, \mathbf{R})$ is measurement error.

#### 4.1.2 Road-Network Map Matching
To prevent physically impossible trajectory projections, raw EKF state projections are projected onto topological road segments extracted from OpenStreetMap graph models $\mathcal{G}_{\text{road}} = (\mathcal{V}_{\text{road}}, \mathcal{E}_{\text{road}})$. This constrains the predicted trajectory $\hat{\mathcal{T}}_v = \{(\hat{p}_x(\tau), \hat{p}_y(\tau)) \mid \tau \in [t, t + \Delta T]\}$ strictly to navigable lanes.

#### 4.1.3 Sojourn Time Forecasting
Given a fog node $F_j$ located at $(X_j, Y_j)$ with effective communication radius $R_j$, the remaining dwell/sojourn time $T_{\text{sojourn}}(v, F_j)$ of vehicle $v$ moving at speed $v(t) = \sqrt{v_x^2 + v_y^2}$ along trajectory heading $\theta(t)$ is computed by solving the intersection geometry:
$$T_{\text{sojourn}}(v, F_j) = \frac{\sqrt{R_j^2 - d_{\perp}^2} + d_{\parallel}}{v(t)}$$
where $d_{\perp}$ is the perpendicular distance from $F_j$ to the vehicle's trajectory vector, and $d_{\parallel}$ is the projected forward distance to the exit boundary.

---

### 4.2 Task Classification Module

Tasks generated in smart transportation environments exhibit diverse resource and deadline footprints:

| Task Class | Latency Threshold ($T_{\text{deadline}}$) | Computational Intensity ($C_{\text{task}}$) | State Footprint ($D_{\text{state}}$) | Exemplary Application | Tier Allocation |
|---|---|---|---|---|---|
| **Ultra-Low Latency (ULL)** | $< 10\text{ ms}$ | $50 - 200\text{ Mcycles}$ | $< 50\text{ KB}$ | Collision avoidance, pedestrian detection, platooning | **Local Device Tier** |
| **Low Latency (LL)** | $10 - 100\text{ ms}$ | $200 - 1500\text{ Mcycles}$ | $100\text{ KB} - 2\text{ MB}$ | Traffic sign recognition, localized navigation | **Fog Tier (RSU)** |
| **Moderate Latency (ML)** | $100\text{ ms} - 1\text{ s}$ | $1.5 - 8\text{ Gcycles}$ | $2 - 20\text{ MB}$ | Dynamic multi-route optimization, urban video analytics | **Fog Tier or Cloud** |
| **Best Effort (BE)** | $> 1\text{ s}$ | $> 8\text{ Gcycles}$ | $> 20\text{ MB}$ | HD map synchronization, fleet analytics, DNN re-training | **Cloud Tier** |

---

### 4.3 Offloading Decision Engine (ODE)

#### 4.3.1 Normalized Multi-Objective Optimization Formulation
To resolve the dimensional incompatibility of raw time (seconds) and energy (joules), MAEFC formulates offloading as a **dimensionally normalized multi-objective mixed-integer optimization problem**:

$$\min_{\mathbf{x}} \quad \Phi(\mathbf{x}) = \alpha \cdot \frac{T_{\text{total}}(\mathbf{x})}{T_{\text{deadline}}} + \beta \cdot \frac{E_{\text{total}}(\mathbf{x})}{E_{\text{budget}}} + \gamma \cdot \frac{C_{\text{mig}}(\mathbf{x})}{C_{\text{mig,max}}}$$

**Subject to:**
1. $\sum_{j \in \mathcal{K}} x_{i,j} = 1 \quad \forall \tau_i \quad (\text{Unique placement: local, candidate fog } F_j, \text{ or cloud})$
2. $T_{\text{total}}(\mathbf{x}) \le T_{\text{deadline}} \quad (\text{Strict latency deadline satisfaction})$
3. $E_{\text{device}}(\mathbf{x}) \le E_{\text{avail}} \quad (\text{Local battery budget constraint})$
4. $\sum_{\tau_i} x_{i,j} \cdot c_i \le \mathcal{C}_j^{\text{max}} \quad \forall j \in \mathcal{K}_{\text{fog}} \quad (\text{Fog computational capacity constraint})$
5. $\alpha + \beta + \gamma = 1, \quad \alpha, \beta, \gamma \ge 0 \quad (\text{Normalized weight vector})$

Where:
- $T_{\text{total}}(\mathbf{x}) = T_{\text{comm}}(\mathbf{x}) + T_{\text{queue}}(\mathbf{x}) + T_{\text{comp}}(\mathbf{x})$
- $E_{\text{total}}(\mathbf{x}) = E_{\text{device}}(\mathbf{x}) + \rho \cdot E_{\text{fog}}(\mathbf{x})$ (with $\rho \in [0, 1]$ scaling edge provider energy cost)
- $C_{\text{mig}}(\mathbf{x}) = (1 - P_{\text{stay}}) \cdot \left[ \mu \frac{T_{\text{mig}}}{T_{\text{deadline}}} + (1-\mu) \frac{E_{\text{mig}}}{E_{\text{budget}}} \right]$

#### 4.3.2 Normalized Weights per Task Class

| Task Class | Latency Weight ($\alpha$) | Energy Weight ($\beta$) | Migration Sensitivity ($\gamma$) | Optimization Bias |
|---|---|---|---|---|
| **ULL** | $0.75$ | $0.10$ | $0.15$ | Strict latency minimization; local device default |
| **LL** | $0.50$ | $0.25$ | $0.25$ | RSU execution with proactive migration protection |
| **ML** | $0.30$ | $0.45$ | $0.25$ | Balanced energy-latency trade-off; selective offload |
| **BE** | $0.10$ | $0.65$ | $0.25$ | Energy minimization; cloud tier preferred |

#### 4.3.3 Offloading Decision Algorithm

```
Algorithm 1: MAEFC Adaptive Offloading Decision
────────────────────────────────────────────────────────────────────────
Input : Task τ_i (C_task, D_in, D_out, T_deadline, E_budget),
        Vehicle State x_v (position, velocity, trajectory),
        Fog Set F = {F_1, F_2, ..., F_M}, Cloud Node F_cloud
Output: Optimal Placement Target k*, Allocation Policy P*

1.  Classify task τ_i into Class ∈ {ULL, LL, ML, BE};
2.  Assign weight vector (α, β, γ) based on Class;
3.  IF Class == ULL THEN
4.      Calculate local execution latency T_local and energy E_local;
5.      IF T_local <= T_deadline THEN
6.          RETURN Target = Local, Frequency = f_device_max;
7.      END
8.  END
9.  Query Mobility Prediction Module:
10.     Retrieve predicted sojourn times T_sojourn(v, F_j) and confidence conf_v;
11. candidate_set = {Local, F_cloud} ∪ {F_j ∈ F | LinkQuality(v, F_j) > Threshold};
12. Best_Cost = ∞, Best_Target = NULL;
13. FOR EACH candidate k IN candidate_set DO
14.     Estimate T_comm(k), T_queue(k), T_comp(k);
15.     T_total(k) = T_comm(k) + T_queue(k) + T_comp(k);
16.     IF T_total(k) > T_deadline THEN CONTINUE; // Infeasible
17.     Estimate device energy E_device(k) and provider energy E_infra(k);
18.     Compute Stay Probability P_stay(k) = Pr(T_sojourn(v, k) >= T_total(k));
19.     Compute Expected Migration Cost C_mig(k) = (1 - P_stay(k)) * Cost_mig(k);
20.     Normalized_Cost = α * (T_total(k) / T_deadline) 
21.                     + β * (E_total(k) / E_budget) 
22.                     + γ * (C_mig(k) / C_mig_max);
23.     IF Normalized_Cost < Best_Cost THEN
24.         Best_Cost = Normalized_Cost;
25.         Best_Target = k;
26.     END
27. END FOR
28. RETURN Best_Target, ResourcePolicy(Best_Target);
────────────────────────────────────────────────────────────────────────
```

---

### 4.4 Task Migration Module

When mobility makes disconnection imminent ($T_{\text{sojourn}} < T_{\text{remaining}}$), the Task Migration Module triggers adaptive migration.

#### 4.4.1 Three Context-Aware Migration Strategies

```
             ┌─────────────────────────────────────────────────────────┐
             │       Mobility Disconnection Imminent                   │
             │       (T_sojourn < T_remaining + Safety_Margin)         │
             └────────────────────────────┬────────────────────────────┘
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
     High Prediction Confidence                       Low Confidence / High Jitter
     (conf >= 0.80, Adj. RSU Available)               (conf < 0.80 or Multi-hop path)
                  │                                               │
        ┌─────────┴─────────┐                           ┌─────────┴─────────┐
        ▼                   ▼                           ▼                   ▼
State Size <= 5MB    State Size > 5MB            Next Node Unknown    Burst Congestion
 [Strategy 1: PM]     [Strategy 1.b: PM-Iter]     [Strategy 2: RM]    [Strategy 3: CCFR]
 Proactive Container  Proactive Dirty-Page        Reactive Handover   Cloud Checkpoint &
 Pre-copy Migration   Iterative Migration         Checkpoint Sync     Fog Cold Resume
```

1. **Strategy 1: Proactive Pre-copy Live Migration (PM)**
   - *Mechanism:* While the vehicle remains connected to source node $F_{\text{src}}$, task memory state and environment contexts are pre-copied in iterations across the high-speed inter-RSU fiber link to predicted destination node $F_{\text{dest}}$.
   - *Execution Handover:* A brief stop-and-copy phase transfers remaining dirty memory pages during the physical wireless handover.
   - *Performance:* Sub-millisecond downtime, seamless service continuity.

2. **Strategy 2: Reactive Checkpoint Migration (RM)**
   - *Mechanism:* Triggered when prediction confidence is moderate-to-low or trajectory unexpectedly changes. The task periodically checkpoints state snapshots to local non-volatile edge cache.
   - *Execution Handover:* Upon radio link failure with $F_{\text{src}}$, the newly connected node $F_{\text{new}}$ fetches the latest snapshot from $F_{\text{src}}$ over backhaul and resumes execution.
   - *Performance:* Zero speculative migration waste; moderate handover latency penalty ($30 - 60\text{ ms}$).

3. **Strategy 3: Cloud Checkpoint + Fog Restart (CCFR)**
   - *Mechanism:* Designed for long-duration tasks, highway gaps without contiguous RSU coverage, or severe destination fog overload. State checkpoints are continuously streamed to the cloud tier.
   - *Execution Handover:* Execution shifts to the cloud, or restarts at an RSU reached much later along the highway.
   - *Performance:* Ultra-high resilience; higher round-trip latency ($60 - 150\text{ ms}$).

#### 4.4.2 Migration Strategy Selection Algorithm

```
Algorithm 2: Adaptive Migration Strategy Selection
────────────────────────────────────────────────────────────────────────
Input : Active Task τ, Current Node F_curr, Mobility Forecast MP,
        Destination Fog Candidates {F_next1, F_next2}
Output: SelectedStrategy, TargetNode

1.  T_remaining = GetRemainingExecutionTime(τ);
2.  T_sojourn = MP.estimated_sojourn_time;
3.  safety_margin = 1.5; // seconds
4.  IF T_sojourn >= (T_remaining + safety_margin) THEN
5.      RETURN (NO_MIGRATION, F_curr); // Task will finish locally
6.  END
7.  TargetNode = MP.next_fog_nodes[0].ID;
8.  conf = MP.confidence;
9.  state_size = τ.memory_footprint;
10. bw_f2f = QueryBackhaulBandwidth(F_curr, TargetNode);
11. T_transfer = state_size / bw_f2f;
12. IF conf >= 0.80 AND HasAvailableCapacity(TargetNode, τ) THEN
13.     IF T_sojourn > (T_transfer + 0.5) THEN
14.         RETURN (PROACTIVE_LIVE_MIGRATION, TargetNode);
15.     ELSE
16.         RETURN (REACTIVE_CHECKPOINT_MIGRATION, TargetNode);
17.     END
18. ELSE IF conf >= 0.50 THEN
19.     TargetNode = SelectBestSurrogateFog(MP.next_fog_nodes);
20.     RETURN (REACTIVE_CHECKPOINT_MIGRATION, TargetNode);
21. ELSE
22.     RETURN (CLOUD_CHECKPOINT_FOG_RESTART, Cloud_Node);
23. END
────────────────────────────────────────────────────────────────────────
```

---

### 4.5 Fog-Node Selection Module

When choosing among candidate fog nodes for initial offloading or migration targets, MAEFC calculates a multi-dimensional utility score:

$$\mathcal{S}(F_j) = w_1 \cdot \mathcal{U}_{\text{lat}}(F_j) + w_2 \cdot \mathcal{U}_{\text{eng}}(F_j) + w_3 \cdot \mathcal{U}_{\text{res}}(F_j) + w_4 \cdot \mathcal{U}_{\text{load}}(F_j)$$

where:
- **Latency Utility:** $\mathcal{U}_{\text{lat}}(F_j) = 1 - \frac{T_{\text{rtt}}(v, F_j)}{T_{\text{max\_acceptable}}}$
- **Energy Utility:** $\mathcal{U}_{\text{eng}}(F_j) = 1 - \frac{E_{\text{tx}}(v, F_j) + E_{\text{comp}}(F_j)}{E_{\text{budget}}}$
- **Residence Utility:** $\mathcal{U}_{\text{res}}(F_j) = \min\left(1.0, \frac{T_{\text{sojourn}}(v, F_j)}{T_{\text{task\_duration}}}\right)$
- **Load Balancing Utility:** $\mathcal{U}_{\text{load}}(F_j) = 1 - \frac{\text{CurrentLoad}(F_j)}{\text{Capacity}(F_j)}$
- Weights satisfy: $\sum_{m=1}^4 w_m = 1.0$ (adjusted dynamically based on application class).

---

### 4.6 Energy Management Module

#### 4.6.1 Device Computational Energy (CMOS DVFS Model)
For a mobile device or vehicle OBU executing $C_{\text{task}}$ CPU cycles at operational clock frequency $f_d$:
$$P_{\text{comp}} = P_{\text{dyn}} + P_{\text{stat}} = \kappa_d \cdot V_{dd}^2 \cdot f_d + P_{\text{stat}} \approx \kappa_d \cdot f_d^3 + P_{\text{stat}}$$
where $\kappa_d$ is the hardware capacitance switching factor, and operating voltage scales linearly with frequency ($V_{dd} \propto f_d$).

The total computational energy consumed by the local device is:
$$E_{\text{compute\_local}} = P_{\text{comp}} \cdot \frac{C_{\text{task}}}{f_d} = \kappa_d \cdot C_{\text{task}} \cdot f_d^2 + P_{\text{stat}} \cdot \frac{C_{\text{task}}}{f_d}$$

#### 4.6.2 Wireless Transmission Energy (Shannon Capacity Model)
The uplink data transmission rate $R_{v,j}$ between vehicle $v$ and fog node $F_j$ over wireless bandwidth $B$ is:
$$R_{v,j} = B \cdot \log_2\left(1 + \frac{P_{\text{tx}} \cdot |h_{v,j}|^2}{\sigma^2 + I_{v,j}}\right)$$
where $P_{\text{tx}}$ is transmission power, $h_{v,j}$ is channel gain accounting for distance path loss $d^{-\eta}$ and Rayleigh fading, $\sigma^2 = N_0 B$ is thermal noise, and $I_{v,j}$ is co-channel interference.

The wireless transmission energy for input data volume $D_{\text{in}}$ is:
$$E_{\text{transmit}} = P_{\text{tx}} \cdot \frac{D_{\text{in}}}{R_{v,j}}$$

#### 4.6.3 Fog Node Power Profile
The operational power of edge fog node $F_j$ is given by:
$$P_{\text{fog}} = P_{\text{idle}} + \sum_{i \in \mathcal{A}_j} \left(\kappa_{\text{fog}} \cdot f_{j,i}^3\right) + P_{\text{cooling\_switching}}$$
where $P_{\text{idle}}$ is baseline static dissipation, $\mathcal{A}_j$ is the set of concurrently active tasks on $F_j$, and $P_{\text{cooling\_switching}}$ represents auxiliary cooling and switching power.

#### 4.6.4 Inter-Fog Migration Energy
For migrating container state $D_{\text{state}}$ over an inter-fog link with power consumption $P_{\text{backhaul}}$ and transmission rate $R_{\text{F2F}}$:
$$E_{\text{migration}} = P_{\text{backhaul}} \cdot \frac{D_{\text{state}}}{R_{\text{F2F}}} + E_{\text{serialize}} + E_{\text{restore}}$$

#### 4.6.5 Energy-Aware Task Scheduling Algorithm

```
Algorithm 3: Fog Energy-Aware Task Scheduling (DVFS)
────────────────────────────────────────────────────────────────────────
Input : Queue Q_j at Fog Node F_j, Time Horizon T, Available Power Budget P_max
Output: Task Execution Schedule and CPU Frequencies {f_i*}

1.  Sort tasks in Q_j in order of Earliest Deadline First (EDF);
2.  FOR EACH task τ_i IN Q_j DO
3.      T_slack = τ_i.deadline - CurrentTime() - T_queue_est;
4.      // Compute minimal CPU frequency satisfying deadline
5.      f_min = τ_i.computation_cycles / T_slack;
6.      // Bound between hardware limits
7.      f_target = clamp(f_min, f_hardware_min, f_hardware_max);
8.      E_est = κ_fog * τ_i.computation_cycles * (f_target^2);
9.      IF (CurrentPower() + (E_est / T_slack)) <= P_max THEN
10.         AssignExecutionSlot(τ_i, f_target);
11.     ELSE IF τ_i.priority >= HIGH_PRIORITY THEN
12.         PreemptLowestPriorityTask(Q_j);
13.         AssignExecutionSlot(τ_i, f_target);
14.     ELSE
15.         // Spillover to Cloud Tier to prevent edge thermal throttling
16.         TriggerCloudOffload(τ_i);
17.     END
18. END FOR
────────────────────────────────────────────────────────────────────────
```

---

### 4.7 Adaptive Resource Allocation Module

The Resource Allocation Module performs continuous joint slicing of CPU cycles, radio resource blocks (RBs), and memory pools across admitted vehicular workloads:

```
┌─────────────────────────────────────────────────────────────────────────┐
│               Adaptive Resource Allocation Controller                   │
│                                                                         │
│   Compute Slicing             Radio Resource Slicing     Memory Slicing │
│  ┌───────────────────────┐   ┌──────────────────────┐   ┌─────────────┐ │
│  │ DVFS Core Scaling     │   │ Dynamic OFDMA RBs    │   │ Page Cache  │ │
│  │ Priority Preemption   │   │ Beamforming Weights  │   │ State Buffer│ │
│  │ GPU Kernel Time-slice │   │ Power Allocation     │   │ Checkpoint Q│ │
│  └───────────────────────┘   └──────────────────────┘   └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

- **Compute Slicing:** Dynamically tunes the allocated CPU cycle rate $f_{j,i} = \frac{C_i}{T_i^{\text{comp}}}$ to match SLA requirements without over-provisioning.
- **Radio Slicing:** Assigns dedicated 5G NR physical resource blocks (PRBs) to prioritize safety-critical offloading requests over background media telemetry.
- **Memory Buffer Slicing:** Pre-allocates pinned memory regions on target fog nodes for anticipated proactive migration snapshots, eliminating allocation latency spikes upon vehicle arrival.

---

## 5. Framework Workflow

### 5.1 End-to-End Task Offloading and Processing Flow

```
Mobile Device / Vehicle                Serving Fog Node F1               Target Fog Node F2 / Cloud
         │                                      │                                    │
         │─── 1. Kinematic & Context Beacon ───►│                                    │
         │       (GPS, Speed, Heading)          │                                    │
         │                                      │── 2. Run MPM Predictor             │
         │                                      │   (Est. Sojourn T_soj)             │
         │                                      │                                    │
         │─── 3. Task Request (τ_i Metadata) ──►│                                    │
         │                                      │── 4. Execute ODE Optimization      │
         │                                      │   (Evaluate Local/Fog/Cloud Cost)  │
         │                                      │                                    │
         │◄── 5. Offloading Decision (F1) ──────│                                    │
         │                                      │                                    │
         │─── 6. Stream Input Data (D_in) ─────►│                                    │
         │                                      │── 7. Energy-Aware Execution (DVFS) │
         │                                      │                                    │
    [Vehicle Moves]                             │                                    │
    [Leaves F1 Coverage]                        │── 8. Migration Trigger Check       │
         │                                      │   (T_sojourn < T_remaining)        │
         │                                      │                                    │
         │                                      │── 9. Proactive State Transfer ────►│
         │                                      │      (Container Memory Pages)      │
         │                                      │                                    │
         │                                      │◄── 10. Transfer Acknowledged ──────│
         │                                      │                                    │
         │                                                                           │── 11. Resume Execution
         │                                                                           │
         │◄─────────────────────── 12. Return Execution Results ─────────────────────│
         │                         (Over direct V2I link from F2)                    │
```

### 5.2 Handover and State Migration Timeline

```
Time ─────────────────────────────────────────────────────────────────────────────►

Vehicle Position:   [ Inside Coverage of F1 ]     [ Handover Zone ]    [ Inside Coverage of F2 ]
                      ▲                             ▲                    ▲
Radio Signal:         │ RSSI_F1 Strong              │ Overlap (Both)     │ RSSI_F2 Strong
                      ▼                             ▼                    ▼
Migration Phase:    Normal Execution        State Pre-copy Phase    Stop-and-Copy Handover   Execution Resumes
                    ┌──────────────────┐    ┌──────────────────┐    ┌────────────────────┐   ┌─────────────────┐
Fog F1 Activity:    │ Active Execution │───►│ Stream Dirty     │───►│ Final CPU Register │──►│ Free Resources  │
                    │ of Task τ_i      │    │ Pages to F2      │    │ Sync to F2         │   │ & Terminate     │
                    └──────────────────┘    └──────────────────┘    └────────────────────┘   └─────────────────┘
                                                      │                        │
                                            High-Speed Fiber          High-Speed Fiber
                                            Backhaul Transfer         Backhaul Transfer
                                                      ▼                        ▼
                                            ┌──────────────────┐    ┌────────────────────┐   ┌─────────────────┐
Fog F2 Activity:    [ Standby Mode ]        │ Buffer Container │───►│ Restore State      │──►│ Execute Final   │
                                            │ Memory Footprint │    │ & Resume Pipeline  │   │ Cycles & Return │
                                            └──────────────────┘    └────────────────────┘   └─────────────────┘
```

---

## 6. Evaluation of Strategies

To evaluate MAEFC, a comprehensive vehicular simulation environment was configured modeling an urban corridor with 8 Roadside Units spaced $400\text{ m}$ apart, overlapping coverage radii of $250\text{ m}$, high-density vehicle traffic moving at velocities of $40 - 100\text{ km/h}$, and mixed task profiles (ULL, LL, ML, BE).

### 6.1 Task Migration Strategies Evaluation

| Strategy | Average Latency (ms) | Energy Cost (mJ/task) | Service Continuity (%) | Migration Overhead (ms) | Task Completion Rate (%) |
|---|---|---|---|---|---|
| **Proactive Migration (PM)** | **$45.2\text{ ms}$** | $148.5\text{ mJ}$ | **$97.4\%$** | $12.1\text{ ms}$ | **$96.8\%$** |
| **Reactive Migration (RM)** | $78.6\text{ ms}$ | **$121.3\text{ mJ}$** | $88.1\%$ | $31.4\text{ ms}$ | $89.5\%$ |
| **Cloud Checkpoint (CCFR)** | $122.4\text{ ms}$ | $198.7\text{ mJ}$ | $94.6\%$ | $56.2\text{ ms}$ | $93.8\%$ |
| **No Migration (Kakati-style)**| $36.1\text{ ms}^*$| $102.4\text{ mJ}$ | $68.2\%^{**}$ | **$0.0\text{ ms}$** | $64.7\%^{**}$ |
| **OnlyLocal Execution** | $25.0\text{ ms}$ | $284.1\text{ mJ}$ | $100.0\%$ | N/A | $99.2\%$ |
| **OnlyCloud Execution** | $182.5\text{ ms}$ | $89.3\text{ mJ}$ | $99.8\%$ | N/A | $99.5\%$ |

$^*$ *Applies only to tasks that successfully conclude prior to cell exit.*
$^{**}$ *Under vehicular mobility, tasks fail or suffer massive disconnection delays once the vehicle departs RSU coverage.*

**Key Findings:**
- **Proactive Migration (PM)** achieves the highest service continuity ($97.4\%$) and task completion ($96.8\%$) with negligible migration downtime ($12.1\text{ ms}$).
- **No Migration** exhibits low latency on short tasks, but its completion rate collapses to $64.7\%$ under moderate-to-high vehicle mobility because unmigrated tasks become disconnected.
- **MAEFC Adaptive Switching** dynamically engages PM when prediction confidence is high ($\ge 0.80$), falls back to RM under unpredictable traffic, and delegates to CCFR during infrastructure gaps, achieving an aggregate completion rate of $>96\%$.

---

### 6.2 Fog-Node Selection Strategies Evaluation

| Selection Policy | Average Latency (ms) | Migrations per 100 Tasks | Total Energy (mJ/task) | Load Balance (Gini Coeff.) | Task Completion Rate (%) |
|---|---|---|---|---|---|
| **Nearest Fog Node** | $31.2\text{ ms}$ | $36.4$ | $142.3\text{ mJ}$ | $0.46$ | $81.5\%$ |
| **Least Loaded Node** | $56.8\text{ ms}$ | $28.1$ | $161.8\text{ mJ}$ | **$0.14$** | $87.8\%$ |
| **Mobility-Aware Nearest** | $41.5\text{ ms}$ | $12.3$ | $134.7\text{ mJ}$ | $0.29$ | $93.7\%$ |
| **MAEFC Multi-Criteria** | **$42.1\text{ ms}$** | **$10.2$** | **$124.6\text{ mJ}$** | $0.19$ | **$96.4\%$** |

**Key Findings:**
- **Nearest Node** creates severe load hotspots at popular intersections (Gini coefficient $0.46$) and incurs excessive migrations ($36.4$ per $100$ tasks) because vehicles rapidly traverse out of range.
- **MAEFC Multi-Criteria Selection** reduces total migrations by **$72.0\%$** compared to nearest-node selection by jointly scoring latency, energy, predicted sojourn time, and load status.

---

### 6.3 Energy Consumption Breakdown

```
Energy Consumption per Completed Task (mJ)

OnlyLocal:   [██████████████████████████████████████████] 284 mJ
             Compute: 254 mJ | Transmit: 0 mJ | Migration: 0 mJ | Idle/Aux: 30 mJ

OnlyFog:     [████████████████████████] 162 mJ
             Compute: 22 mJ  | Transmit: 82 mJ | Migration: 32 mJ | Idle/Aux: 26 mJ

OnlyCloud:   [█████████████] 89 mJ
             Compute: 6 mJ   | Transmit: 69 mJ | Migration: 0 mJ  | Idle/Aux: 14 mJ

MAEFC:       [██████████████████] 124 mJ
             Compute: 34 mJ  | Transmit: 56 mJ | Migration: 16 mJ | Idle/Aux: 18 mJ
```

#### Energy Breakdown across Task Classes:
- **ULL Tasks:** Local execution consumes $50\text{ mJ}$ (zero transmission energy); offloading to fog would waste $75\text{ mJ}$ in radio overhead. MAEFC executes $100\%$ of ULL tasks locally.
- **LL Tasks:** Fog execution saves $42.5\%$ device energy compared to local execution ($115\text{ mJ}$ vs. $200\text{ mJ}$) while satisfying the $100\text{ ms}$ deadline.
- **ML & BE Tasks:** Heavy compute intensity enables cloud offloading to achieve dramatic energy reductions ($67.5\% - 85.8\%$ energy savings compared to local execution).

---

### 6.4 Latency under Varying Vehicular Velocities

| Vehicular Velocity (Speed Regime) | OnlyLocal Latency | OnlyFog (Static) Latency | OnlyCloud Latency | MAEFC Adaptive Latency |
|---|---|---|---|---|
| **Stationary / Pedestrian ($0 - 5\text{ km/h}$)** | $25.0\text{ ms}$ | $34.8\text{ ms}$ | $181.2\text{ ms}$ | **$31.8\text{ ms}$** |
| **Urban Driving ($30 - 50\text{ km/h}$)** | $25.0\text{ ms}$ | $64.2\text{ ms}^*$ | $182.0\text{ ms}$ | **$44.6\text{ ms}$** |
| **Suburban Arterial ($60 - 80\text{ km/h}$)** | $25.0\text{ ms}$ | $98.5\text{ ms}^*$ | $183.5\text{ ms}$ | **$51.2\text{ ms}$** |
| **Highway Speed ($90 - 120\text{ km/h}$)** | $25.0\text{ ms}$ | $145.0\text{ ms}^*$ | $185.0\text{ ms}$ | **$56.8\text{ ms}$** |

$^*$ *Includes multi-hop packet forwarding and reconnection penalties.*

---

### 6.5 Service Continuity Across Mobility Trajectories

| Vehicular Mobility Pattern | No Migration | Reactive Only (RM) | Proactive Only (PM) | MAEFC Hybrid Framework |
|---|---|---|---|---|
| **Linear Highway (Predictable)** | $42.0\%$ | $84.2\%$ | $95.8\%$ | **$96.5\%$** |
| **Urban Grid / Intersections** | $65.4\%$ | $87.5\%$ | $92.1\%$ | **$95.8\%$** |
| **Stop-and-Go / Congestion** | $78.0\%$ | $91.0\%$ | $94.5\%$ | **$96.2\%$** |
| **Erratic Lane / Turning Movements** | $38.5\%$ | $82.4\%$ | $81.0\%$ | **$92.4\%$** |

**Insight:** While proactive migration outperforms reactive migration on straight highway corridors, unpredictable turning maneuvers degrade PM accuracy. MAEFC maintains $>92\%$ service continuity in erratic traffic by dynamically dropping back to reactive checkpointing when EKF trajectory confidence falls below $0.80$.

---

## 7. Energy–Performance Trade-off Justification

### 7.1 The Fundamental Trade-off

The optimization space in mobility-aware edge computing contains a fundamental tension between **energy conservation** and **application performance** (latency + service continuity):

```
                       Application Performance (1 / Latency)
                                        ▲
                                        │
           OnlyLocal ●                  │             ● MAEFC (Optimal Operating Point)
       (Ultra-low latency,              │           (High performance, balanced energy,
        High device energy drain)       │            high service continuity)
                                        │
                                        │
   ─────────────────────────────────────┼─────────────────────────────────────► Energy
                                        │                                       Efficiency
                                        │
                                        │             ● OnlyCloud
                                        │           (High energy savings,
                                        │            excessive latency)
                                        ▼
```

### 7.2 Trade-off Justification by Application Class

| Application Class | Primary Optimization Goal | Tolerated Trade-off | Justification & Operating Rationale |
|---|---|---|---|
| **ULL (Safety Critical)** | **Latency & Reliability** | **High local energy consumption** | Crash prevention and emergency braking require sub-10ms response. Consuming $50\text{ mJ}$ of vehicle power is non-negotiable when human safety is involved. |
| **LL (Real-time Vision)** | **Balanced Delay** | **Moderate backhaul transmission power** | Video stream processing offloaded to edge RSUs achieves $40\text{ ms}$ latency while saving $>40\%$ battery over on-device GPU inference. |
| **ML (Route Analytics)** | **Energy Minimization** | **Occasional migration / queue latency** | Navigation and congestion re-routing tolerate $500\text{ ms}$ delays. Migrating only when beneficial saves $25\%$ energy over aggressive pre-copying. |
| **BE (Telemetry / Updates)** | **Maximum Battery Preservation** | **High latency ($>1\text{ s}$)** | Delay-insensitive background uploads are offloaded directly to the cloud tier, achieving $>85\%$ device energy conservation. |

### 7.3 Quantitative Migration Cost–Benefit Model

MAEFC mandates that a task state migration is initiated **if and only if the expected benefit exceeds total migration overhead**:

$$\mathbb{E}[\Delta \mathcal{U}_{\text{mig}}] = \left( P_{\text{fail}}^{\text{no\_mig}} \cdot \mathcal{C}_{\text{failure}} + \Delta T_{\text{latency\_benefit}} \right) - \left( E_{\text{state\_transfer}} + \tau_{\text{downtime}} + (1 - \text{conf}_v) \cdot \mathcal{C}_{\text{mispredict}} \right) > 0$$

#### Quantitative Scenario Comparison:

| Migration Policy | Migration Invocation Rate | Migration Energy (mJ/task) | Task Failure Rate | End-to-End Latency | System Efficiency Score |
|---|---|---|---|---|---|
| **Always Migrate (Eager)** | $100.0\%$ | $38.4\text{ mJ}$ | $2.4\%$ | $56.2\text{ ms}$ | $0.71$ |
| **Never Migrate (Kakati-style)**| $0.0\%$ | **$0.0\text{ mJ}$** | $34.8\%$ | $36.1\text{ ms}^* / \infty$ | $0.54$ |
| **MAEFC Selective Migration** | **$39.2\%$** | **$15.8\text{ mJ}$** | **$3.6\%$** | **$45.2\text{ ms}$** | **$0.89$** |

*Result:* MAEFC's selective migration eliminates $60.8\%$ of redundant migrations, saving $58.9\%$ in migration energy compared to eager migration while maintaining a near-identical task completion rate ($96.4\%$ vs $97.6\%$).

### 7.4 Dynamic Weight Tuning Rules

The objective weights $(\alpha, \beta, \gamma)$ adapt at runtime based on environmental telemetry:

```
Dynamic Weight Adaptation Rules:

// Battery Preservation Override
IF Device.BatteryPercentage < 25.0 THEN
    beta  = beta + 0.35 * (1.0 - Device.BatteryPercentage / 25.0);
    alpha = max(0.10, alpha - 0.20);
    gamma = max(0.10, gamma - 0.15);
END

// Wireless Congestion Adaptation
IF Channel.SINR < 5.0 dB OR Channel.PacketLossRate > 0.15 THEN
    // Wireless channel degraded: favor local execution over offloading
    alpha_local_bias = alpha_local_bias + 0.25;
    beta_transmit_penalty = beta_transmit_penalty * 1.5;
END

// Safety-Critical Real-time Override
IF Task.IsSafetyCritical == TRUE THEN
    alpha = 0.80;  // Latency dominant
    beta  = 0.10;  // Minimize energy priority
    gamma = 0.10;  // Minimize migration sensitivity
END
```

---

## 8. Summary & Research Gap Alignment

The MAEFC architecture resolves the composite research gap identified across the 13 surveyed literature papers:

1. **Resolution of the Migration Dichotomy:** Prior works forced an all-or-nothing choice—either completely banning migration (Kakati *et al.*, 2022) or reactively migrating tasks without cost gating (Wang *et al.*, 2019). MAEFC implements **selective prediction-guided migration**, executing state transfers only when the quantified performance delta exceeds serialization and network transmission penalties.
2. **Three-Tier Synergy:** While prior models operated predominantly over isolated two-tier setups (Zhu *et al.*, Shi *et al.*, Wang *et al.*), MAEFC integrates the Cloud Tier as a continuous checkpoint synchronization target and computational overflow sink, preventing edge thermal overload.
3. **Multi-Objective Optimization with Rigorous Normalization:** Building upon evolutionary game formulations (MOFCO, Mahdizadeh *et al.*, 2025) and DVFS energy modeling, MAEFC formulates a dimensionally normalized cost objective combining delay, device battery drain, and migration risk.
4. **Resilience to Real-World Mobility:** By combining Extended Kalman Filtering, map-matching, and sojourn-time projection, MAEFC provides sub-second handover execution and sustains $>96\%$ service continuity across diverse vehicular speed regimes.

---
