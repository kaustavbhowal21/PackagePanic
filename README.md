***

# GSoC 2026 Project Proposal: OpenStreetMap
**Project Title:** High-Performance ALNS Framework for Coupled Pickup-and-Delivery Optimization on OSM Networks
**Project Theme:** C++ Performance Engineering, Combinatorial Optimization, and Full-Stack Geospatial Visualization

---

## **1. Executive Summary**
Modern supply chain logistics requires solving the **Capacitated Vehicle Routing Problem with Pickup and Delivery and Time Windows (CVRP-PDTW)**. While OpenStreetMap (OSM) provides the world's most granular and up-to-date road network data, the open-source ecosystem currently lacks a dedicated, high-performance C++ solver capable of computing complex, multi-constrained routing topologies at an enterprise scale.

This project bridges the gap between static geospatial data and dynamic routing requirements by introducing a specialized optimization engine. Built natively in C++, the engine leverages an **Adaptive Large Neighborhood Search (ALNS)** metaheuristic. By prioritizing advanced **Regret-k Insertion** logic and enforcing strict physical constraints (volumetric weight capacities, cost-per-kilometer metrics, and tightly coupled time windows), this engine transforms raw OSM distance matrices into cost-optimized, universally feasible routing schedules. 

To ensure accessibility for the broader OSM community, the core C++ engine will be wrapped in a **FastAPI backend** and visualized through an interactive **React/Leaflet web dashboard**, making algorithmic logistics visually accessible and instantly deployable.

---

## **2. Core Algorithmic Architecture (ALNS)**
The core of the solver relies on the ALNS metaheuristic, which systematically explores the solution space by partially destroying and repairing routing assignments. The engine will implement the following specific operators:

* **Destroy Operators (Ruin):**
    * *Random Removal:* Ensures search diversity and prevents stagnation in local optima.
    * *Worst-Cost Removal:* Greedily targets and removes the most expensive nodes in the current configuration.
    * *Shaw (Relatedness) Removal:* Removes requests that are highly similar in distance, time window, or payload size, effectively reshuffling dense geographic clusters.
* **Repair Operators (Recreate):**
    * *Greedy Insertion:* Inserts removed requests at their cheapest possible valid position.
    * *Regret-2 and Regret-k Insertion:* A forward-looking heuristic that calculates the cost difference (regret) between a request's best and $k$-th best insertion positions. This prevents "stranding" difficult requests until the end of the routing phase.
* **Adaptive Weight Adjustment:** The probability of choosing a specific operator is dynamically updated based on its historical success in improving the global objective function $Z$ during the current run.

---

## **3. Technical Constraints & Mathematical Model**
The C++ engine strictly enforces a multi-dimensional feasibility matrix for every proposed route modification. An insertion is immediately rejected if it violates any of the following:

**1. Coupled Time Windows:** Every service request dictates an strict interval $[T_{Earliest\_Pickup}, T_{Latest\_Delivery}]$. The engine verifies chronological precedence (Pickup strictly occurs before Delivery) and ensures total route duration accommodates all defined windows.

**2. Volumetric Load Management:** The system tracks fluctuating node-by-node vehicle weight. A sequence modification is only mathematically feasible if the cumulative load never exceeds the assigned vehicle's maximum limit:
```math
\sum \text{Pickups} - \sum \text{Deliveries} \leq \text{Vehicle\_Capacity}_{max}
```

**3. Economic Objective Function:** The search optimizes for a minimized total cost function $Z$, penalizing excessive distance, fleet activation, and sub-optimal priority routing:
```math
Z = \sum (\text{Distance} \times \text{Cost\_Per\_Km}) + \text{Fleet\_Activation\_Fixed\_Costs}
```

---

## **4. Proposed Innovation: The Complexity Profiler**
A major challenge in deploying metaheuristics is deciding the termination criterion. To bridge the gap between academic theory and production environments, this project introduces a **Runtime Suggestion Engine**. 

Upon loading the initial OSM JSON request, the C++ pre-processor calculates a **Complexity Score** ($\sigma$) by analyzing search-space density:
```math
\sigma = f(N^2, \text{Sparsity}, \text{Window\_Tightness})
```
By analyzing the tightness of time windows (which prunes the search tree) and geographic sparsity, the program outputs a data-driven computation recommendation (e.g., *"System identifies highly constrained time windows; suggests 45 seconds for optimal convergence"*).

---

## **5. System Architecture & Full-Stack Integration**
The project utilizes a decoupled, three-tier architecture ensuring the heavy computational load is isolated from the user interface.

1.  **Core Optimization Engine (C++17/20):** High-performance implementation using the Standard Template Library (STL) for memory-safe graph representations, operator overloading for matrix manipulation, and `libosmium` for direct PBF parsing.
2.  **API Orchestrator (FastAPI/Python):** A lightweight REST API that handles JSON schema validation, sub-processes the C++ binary, and serializes the optimized output into standard GeoJSON formats.
3.  **Interactive Visualization (React.js + Leaflet):** A dynamic web dashboard allowing users to define fleet constraints, set ALNS execution time limits, and visually play back the resulting multi-vehicle routes on an OSM basemap.

---

## **6. Detailed 12-Week Implementation Timeline (GSoC 2026)**

### **Pre-Coding: Community Bonding Period**
**Dates: May 1 – May 24, 2026**
* Establish a weekly communication cadence with OSM mentors via IRC/Slack.
* Finalize C++ environment setup, select unit testing frameworks (`GoogleTest`), and configure the CMake build pipeline.
* Agree upon the strict JSON schema for the Input/Output API contracts.

### **Phase 1: Foundation, Data Ingestion & Initial Solvers**
**Dates: May 25 – June 14, 2026 (Weeks 1–3)**
* **Week 1:** Implement robust C++ data structures (`ServiceRequest`, `Vehicle`, `RouteSequence`). Apply advanced template comparisons for fast matrix lookups.
* **Week 2:** Build the distance/time matrix extractor interfacing with OSM files via `libosmium`.
* **Week 3:** Develop the baseline Greedy Insertion algorithm to guarantee an initial feasible (non-optimized) solution. 

### **Phase 2: Core ALNS Metaheuristic Development**
**Dates: June 15 – July 5, 2026 (Weeks 4–6)**
* **Week 4:** Implement strict constraint-checking subroutines for Volumetric Load and Coupled Time Windows.
* **Week 5:** Code the **Regret-2 and Regret-k Repair** operators.
* **Week 6:** Implement the primary **Destroy** operators (Random, Worst-Removal) and establish the adaptive weight scoring loop.

> ### **GSoC Midterm Evaluation Window: July 6 – July 10, 2026**
> **Midterm Deliverable:** A fully functional, strictly-constrained C++ CLI application that reliably improves an initial greedy solution using the ALNS loop within a user-defined execution time.

### **Phase 3: Advanced Heuristics & Full-Stack API Integration**
**Dates: July 13 – August 2, 2026 (Weeks 7–9)**
* **Week 7:** Finalize the Shaw (Relatedness) Removal operator. Develop the algorithmic **Complexity Profiler** for runtime prediction.
* **Week 8:** Develop the Python FastAPI backend to act as the RESTful wrapper for the compiled C++ engine.
* **Week 9:** Begin development of the React.js/Leaflet frontend to accept JSON uploads and render mapping data.

### **Phase 4: UI Completion, Benchmarking & Documentation**
**Dates: August 3 – August 16, 2026 (Weeks 10–11)**
* **Week 10:** Complete the web dashboard, enabling map-based visualization of vehicle routing, timeline sliders, and capacity tracking.
* **Week 11:** Execute massive stress testing against established CVRP-PDTW datasets (e.g., Li & Lim benchmarks) to empirically validate the mathematical integrity of the engine. Target $\pm 2-5\%$ of best-known heuristic solutions.

> ### **GSoC Final Submission & Evaluation Window: August 17 – August 24, 2026**
> **Final Deliverable:** Submission of the complete, documented GitHub repository. This includes the high-performance C++ source code, the full-stack visualization web app, Docker deployment configurations, and comprehensive benchmarking reports against standard datasets.

---

## **7. Testing, CI/CD, and Quality Assurance**

To ensure the engine is both robust and mathematically sound, the project will follow a three-tiered testing hierarchy: **Unit Testing** (Logic), **Integration Testing** (OSM Data), and **Benchmarking** (Algorithmic Performance).

  

#### **7.1. Algorithmic Benchmarking (The "Gold Standard")**

The primary method for proving the ALNS engine works is to compare its results against globally recognized VRP datasets.

*  **Li & Lim PDPTW Instances:** I will run the solver against the standard Li & Lim benchmarks for **Pickup and Delivery Problems with Time Windows**.

*  **Success Metric:** The goal is to achieve a solution cost within **±2–5% of the known best-published solutions** for these instances within the user-defined time limit.

*  **Convergence Visualization:** I will implement a logging feature that exports the "Best Cost vs. Iteration Number" to a CSV, allowing for the generation of convergence graphs to prove the "Adaptive" nature of the search is actually improving the solution over time.

  

#### **7.2. Constraint Integrity & Stress Testing**

Because this engine handles strict physical constraints (Weight and Time), I will implement a suite of **Adversarial Test Cases**:

*  **The "Overload" Test:** Input a single package that exceeds the capacity of all available vehicles. The engine must gracefully report a "Feasibility Error" rather than crashing or producing an invalid route.

*  **The "Impossible Window" Test:** Create a pickup at 12:00 PM and a delivery at 12:05 PM where the OSM distance matrix requires at least 20 minutes of travel. The engine must identify this as an unserviced request.

*  **Memory Leak Detection:** Since this is a core C++ project, I will use **Valgrind** and **AddressSanitizer** during Phase 4 to ensure that large-scale distance matrices (1,000x1,000) do not cause memory leaks or segmentation faults during long ALNS loops.

  

#### **7.3. Predictor Accuracy Validation**

The **Complexity Profiler** (Runtime Suggestion Engine) needs its own validation.

*  **Methodology:** I will run a batch of 50 different problem sets (varying $N$, vehicle counts, and time-window tightness).

*  **Validation:** I will measure the time it takes for the ALNS loop to reach "Diminishing Returns" (where the cost improves by less than 0.01% over 1,000 iterations).

*  **Goal:** The "Suggested Runtime" should be within **15%** of this actual observed convergence time.

  

#### **7.4. CI/CD & Unit Testing**

*  **Framework:** Utilizing **GitHub Actions** with **GoogleTest (gTest)**.

*  **Automated Checks:** Every Pull Request will trigger a build to ensure that new "Destroy/Repair" operators do not break the core cost-function logic.

*  **Precedence Verification:** Specific unit tests will verify that for every route, the `Pickup_Index` is always less than the `Delivery_Index` in the execution array.

---
