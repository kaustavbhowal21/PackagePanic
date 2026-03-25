
  

---

  

# GSoC 2026: Project Proposal for OpenStreetMap

**Project Theme:** High-Performance Logistics & Combinatorial Optimization

  

## **1. Project Title - PackagePanic**

**Developing a High-Performance ALNS Framework for Coupled Pickup-and-Delivery Optimization on OSM Networks**

  

---

  

## **2. Executive Summary**

Modern logistics requires more than simple point-to-point routing; it demands the solving of the **Capacitated Vehicle Routing Problem with Pickup and Delivery and Time Windows (CVRP-PDTW)**. While OpenStreetMap (OSM) provides the world's most granular road data, there remains a gap in open-source C++ tooling capable of solving these NP-hard problems at scale.

  

This project proposes a specialized C++ optimization engine designed to ingest OSM-based cost matrices and produce cost-minimized feasible solutions. The engine utilizes an **Adaptive Large Neighborhood Search (ALNS)** metaheuristic, prioritizing a **Regret-$k$ Insertion** logic to handle strict constraints including vehicle weight capacities, cost-per-kilometer metrics, and coupled time windows. A standout feature of this proposal is the inclusion of a **Pre-Processing Profiler** that analyzes problem complexity to suggest an optimal execution runtime to the user.

  

---

  

## **3. Technical Requirements & Constraints**

The engine will strictly enforce a multi-dimensional feasibility matrix for every potential route modification:

  

*  **Coupled Time Windows:** Every request is defined by a discrete interval $[T_{Earliest\_Pickup}, T_{Latest\_Delivery}]$. The engine ensures that the pickup occurs before the delivery and that both actions fit within the global vehicle schedule.

*  **Volumetric Load Management:** The system tracks the fluctuating weight of the vehicle at every node. A move is only feasible if:

$$\sum  \text{Pickups} - \sum  \text{Deliveries} \leq  \text{Vehicle\_Capacity}_{max}$$

*  **Economic Objective Function:** The search focuses on minimizing a total cost function $Z$, defined as:

$$Z = \sum (\text{Distance} \times  \text{Cost\_Per\_Km}) + \text{Fleet\_Activation\_Fixed\_Costs}$$

*  **User-Defined Convergence:** The ALNS loop will execute for a duration specified by the user, ensuring the "Global Best" solution is captured within the allocated compute budget.

  

---

  

## **4. Proposed Innovation: The Complexity Profiler**

To bridge the gap between academic algorithms and production software, this tool will include an intelligent **Runtime Suggestion Engine**. Upon reading the input data (Nodes, Weights, and Windows), the program will calculate a **Complexity Score ($\sigma$)**:

$$\sigma = f(N^2, \text{Sparsity}, \text{Window\_Tightness})$$

Before the optimization begins, the program will output a recommendation (e.g., *"Based on 500 requests and high spatial sparsity, 60 seconds is recommended for 95% convergence"*), allowing the user to make informed decisions on the time-vs-quality trade-off.

  

---

  

## **5. 12-Week Implementation Roadmap (2026)**

  

### **Pre-Coding: Community Bonding Period**

**Dates: May 1 – May 24, 2026**

*  **Activity:** Establish a weekly meeting cadence with OSM mentors. Set up the development environment, finalize the choice of C++ libraries (e.g., `libosmium`, `Boost.Graph`), and refine the JSON schema for input/output.

*  **Goal:** Have a "Hello World" OSM parser running before the first day of coding.

  

### **Phase 1: Foundation & Data Ingestion**

**Dates: May 25 – June 14, 2026 (Weeks 1–3)**

*  **Milestones:**

* Implement the C++ `ServiceRequest` and `Vehicle` classes.

* Build the distance/time matrix extractor to interface with OSM PBF files.

*  **Deliverable:** A functional CLI that reads a list of coordinates and outputs a valid distance matrix.

  

### **Phase 2: The ALNS Engine & Core Heuristics**

**Dates: June 15 – July 5, 2026 (Weeks 4–6)**

*  **Milestones:**

* Develop the **Regret-2 and Regret-3 Insertion** logic.

* Integrate strict weight capacity and pickup-delivery precedence checks.

*  **Midterm Milestone:** Achieve a "Greedy + Regret" solver that finds a feasible (non-optimized) solution for 100+ nodes.

  

>  **Midterm Evaluation: July 6 – July 10, 2026**

  

### **Phase 3: Optimization & Complexity Profiling**

**Dates: July 13 – August 2, 2026 (Weeks 7–9)**

*  **Milestones:**

* Implement "Destroy" operators (Worst-Removal, Shaw Removal) to allow the search to escape local optima.

* Develop the **Complexity Profiler** to calculate suggested runtimes based on $N$ and time-window density.

* Refine the adaptive weight-adjustment logic (the "A" in ALNS).

  

### **Phase 4: Refinement & Community Integration**

**Dates: August 3 – August 23, 2026 (Weeks 10–12)**

*  **Milestones:**

* Benchmarking against Solomon test instances to verify mathematical accuracy.

* Stress testing with large-scale OSM datasets (1,000+ nodes).

* Finalize documentation, clean up the C++ codebase, and prepare the final project report.

  

>  **Final Submission Deadline: August 24, 2026**

  

---

  

### **6. Evaluation & Testing Strategy**

  

To ensure the engine is both robust and mathematically sound, the project will follow a three-tiered testing hierarchy: **Unit Testing** (Logic), **Integration Testing** (OSM Data), and **Benchmarking** (Algorithmic Performance).

  

#### **6.1. Algorithmic Benchmarking (The "Gold Standard")**

The primary method for proving the ALNS engine works is to compare its results against globally recognized VRP datasets.

*  **Li & Lim PDPTW Instances:** I will run the solver against the standard Li & Lim benchmarks for **Pickup and Delivery Problems with Time Windows**.

*  **Success Metric:** The goal is to achieve a solution cost within **±2–5% of the known best-published solutions** for these instances within the user-defined time limit.

*  **Convergence Visualization:** I will implement a logging feature that exports the "Best Cost vs. Iteration Number" to a CSV, allowing for the generation of convergence graphs to prove the "Adaptive" nature of the search is actually improving the solution over time.

  

#### **6.2. Constraint Integrity & Stress Testing**

Because this engine handles strict physical constraints (Weight and Time), I will implement a suite of **Adversarial Test Cases**:

*  **The "Overload" Test:** Input a single package that exceeds the capacity of all available vehicles. The engine must gracefully report a "Feasibility Error" rather than crashing or producing an invalid route.

*  **The "Impossible Window" Test:** Create a pickup at 12:00 PM and a delivery at 12:05 PM where the OSM distance matrix requires at least 20 minutes of travel. The engine must identify this as an unserviced request.

*  **Memory Leak Detection:** Since this is a core C++ project, I will use **Valgrind** and **AddressSanitizer** during Phase 4 to ensure that large-scale distance matrices (1,000x1,000) do not cause memory leaks or segmentation faults during long ALNS loops.

  

#### **6.3. Predictor Accuracy Validation**

The **Complexity Profiler** (Runtime Suggestion Engine) needs its own validation.

*  **Methodology:** I will run a batch of 50 different problem sets (varying $N$, vehicle counts, and time-window tightness).

*  **Validation:** I will measure the time it takes for the ALNS loop to reach "Diminishing Returns" (where the cost improves by less than 0.01% over 1,000 iterations).

*  **Goal:** The "Suggested Runtime" should be within **15%** of this actual observed convergence time.

  

#### **6.4. CI/CD & Unit Testing**

*  **Framework:** Utilizing **GitHub Actions** with **GoogleTest (gTest)**.

*  **Automated Checks:** Every Pull Request will trigger a build to ensure that new "Destroy/Repair" operators do not break the core cost-function logic.

*  **Precedence Verification:** Specific unit tests will verify that for every route, the `Pickup_Index` is always less than the `Delivery_Index` in the execution array.

---