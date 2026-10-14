# 🚕 Uber System Design: 3D Architecture Explorer & Notion Engineering Report

[![Three.js](https://img.shields.io/badge/Three.js-r128-black?style=for-the-badge&logo=three.js)](https://threejs.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.x-38B2AC?style=for-the-badge&logo=tailwind-css)](https://tailwindcss.com/)
[![Uber H3](https://img.shields.io/badge/Spatial_Index-Uber_H3-276EF1?style=for-the-badge)](https://github.com/uber/h3)
[![Notion Ready](https://img.shields.io/badge/Notion-Ready_Markdown-000000?style=for-the-badge&logo=notion)](https://www.notion.so/)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg?style=for-the-badge)](LICENSE)

> **An interactive 3D WebGL/Three.js geospatial simulator and Principal/Staff-level technical research report dissecting Uber's end-to-end real-time architecture, algorithms, mathematical models, and open-source ecosystem.**

---

## ✨ Project Overview

This repository combines two complementary resources designed for **Software Engineers, System Architects, and Technical Interviewers**:

1. **Interactive 3D System Design Explorer (`uber_system_design_explorer.html`):** A zero-dependency, single-file Three.js + Tailwind CSS studio that visually simulates Uber's core distributed and geospatial algorithms in real time alongside an interactive bipartite cost-matrix calculator.
2. **Complete Notion Engineering Research Report:** A comprehensive, publication-ready technical reference covering every layer of Uber's stack—equipped with a **one-click "Copy for Notion"** exporter inside the app.

---

## 🎮 Interactive 3D Simulator Modes

The Three.js viewport features 4 real-time simulation modes that sync directly with the chapters of the engineering report:

| Mode | Engineering Concept Visualized | Interactive Controls |
| :--- | :--- | :--- |
| **1. H3 Hex Grid & $k$-Ring** | Visualizes Uber's 64-bit flat-topped hexagonal spatial index ($k=0$ origin through $k=3$ equidistant radial rings, $N = 1 + 3k(k+1) = 37$ cells), active driver spheres, and pending rider beacons. | Orbit, Zoom, & Reset Camera |
| **2. Hungarian Batch Match** | Simulates DISCO's 2–5 second batched bipartite matching window, drawing 3D Quadratic Bezier arcs for optimal driver-to-rider assignments via the **Kuhn-Munkres ($O(V^3)$)** algorithm. | **⚡ Run Hungarian Batch** button + live 3×3 Greedy vs. Optimal Matrix Calculator |
| **3. Surge & Ringpop Shards** | Colors H3 cells by their **Ringpop consistent hash ring shard** (`XXHash`) and dynamically extrudes 3D hexagon heights using **Apache Flink $k$-ring smoothed Surge multipliers**. | **🔥 Spike Demand (Surge)** button |
| **4. Kalman + Viterbi Map** | Simulates urban-canyon multipath GPS jitter (red spheres) being fused with 100Hz IMU data (**Extended Kalman Filter**) and snapped onto the green road-network centerline via **Viterbi HMM** emission/transition probabilities. | Full 3D inspection of emission vectors |

---

## 🏗️ End-to-End Architecture Covered in the Report

```text
+-------------------+      QUIC / HTTP/3 (Protobuf)      +-----------------------+
| Driver / Rider    | ================================>  | Edge API Gateway      |
| Mobile App (RIBs) | <================================  | ( Envoy + RAMEN Push ) |
+-------------------+      4s GPS Ping / Push Events     +-----------------------+
                                                                     |
                         +-------------------------------------------+------------------------------------------+
                         |                                           |                                          |
                         v (gRPC / TChannel)                         v (Kafka Telemetry Stream)                 v (gRPC)
             +-----------------------+                   +-----------------------+                  +-----------------------+
             | DISCO Dispatch Engine |                   | Apache Flink + Pinot  |                  | Trip State Machine    |
             | (Ringpop + H3 Memory) |                   | (Surge & Kalman HMM)  |                  | (Cadence + Docstore)  |
             +-----------------------+                   +-----------------------+                  +-----------------------+
```

### Deep-Dive Chapters & Mathematical Formulations Included:
1. **Client & Edge Ingestion Layer:**
   * **RIBs (Router, Interactor, Builder):** State-driven mobile architecture decoupling business logic from view trees across iOS and Android.
   * **QUIC / HTTP/3 & RAMEN Push:** 0-RTT connection migration across cellular handoffs and monotonically sequenced gRPC/HTTP2 bidirectional push streams.
2. **Geospatial Indexing (`Uber H3`):**
   * Comparative benchmarks: **Geohash** vs. **Quadtree / R-Tree** vs. **Google S2** vs. **Uber H3**.
   * Bit-by-bit breakdown of the **64-bit H3 Integer Index** (Mode, Resolution $0\text{–}15$, 122 Base Cells, and 15 Aperture-7 child digits).
3. **Distributed Dispatch Engine (`DISCO` & `Ringpop`):**
   * In-memory spatial sharding over consistent hash rings (`XXHash`) and **TChannel/gRPC** 1-hop request forwarding.
   * **SWIM Gossip Protocol:** $O(1)$ per-node failure detection via direct `ping`, indirect `ping-req(k)`, and piggybacked suspicion states.
4. **Batched Matching & Routing Graphs:**
   * **Hungarian Algorithm (Kuhn-Munkres):** Global bipartite minimum-weight matching in $O(V^3)$ vs. Greedy FCFS.
   * **Contraction Hierarchies (CH)** vs. **Customizable Route Planning (CRP / Multi-Level Dijkstra)** + **DeepETA** residual Transformer models.
5. **Urban Canyon Telemetry & Surge Smoothing:**
   * **Extended Kalman Filter (EKF):** Predict/Update matrix equations fusing 100Hz IMU sensors with 0.25Hz GPS pings.
   * **Viterbi Hidden Markov Model (HMM):** Gaussian emission and road-network transition probabilities in $O(T \cdot M^2)$.
   * **Hexagonal Convolution Kernel:** $k$-ring spatial smoothing to eliminate street-boundary surge pricing cliffs.
6. **Persistence, Financial Ledgers & OLAP:**
   * **Schemaless & Docstore:** Append-only immutable JSON cells (`row_key`, `column_key`, `ref_key`) with buffered failover writes.
   * **Cadence / Temporal Sagas & Double-Entry Ledgers:** Idempotent payment workflows and balanced `DEBIT`/`CREDIT` auditing.
   * **Kafka + Apache Flink + Apache Pinot:** Sub-50ms real-time analytics at 10+ TiB/minute scale.

---

## 🚀 Quick Start (Zero Build Step Required)

Because the entire 3D Explorer and Notion Exporter are bundled into a single self-contained HTML file using CDN modules, you can run it immediately:

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/uber-system-design-3d-explorer.git
cd uber-system-design-3d-explorer
```

### 2. Launch the 3D Explorer
Simply open `uber_system_design_explorer.html` in any modern browser, or serve it locally:
```bash
# Using Python
python3 -m http.server 8080

# Or using Node.js npx
npx serve .
```
Then visit `http://localhost:8080/uber_system_design_explorer.html`.

### 3. Export to Notion in One Click
1. Click the **"Copy for Notion"** button in the top-right header of the web app (or click **"View Raw MD"** to inspect the raw Markdown).
2. Open any Notion page and press `Cmd + V` (macOS) or `Ctrl + V` (Windows/Linux).
3. Notion will automatically render all headings, callout blocks, LaTeX math equations (`$$`), ASCII diagrams, and GitHub comparison tables.

---

## 📦 Uber Open-Source Stack Referenced

| Project | Domain | GitHub Repository |
| :--- | :--- | :--- |
| **H3** | Hierarchical Hexagonal Spatial Index | [`uber/h3`](https://github.com/uber/h3) |
| **Ringpop** | SWIM Gossip + Consistent Hash Ring | [`uber-node/ringpop-node`](https://github.com/uber-node/ringpop-node) |
| **Cadence** | Distributed Workflow Orchestration | [`uber/cadence`](https://github.com/uber/cadence) |
| **RIBs** | Cross-Platform Mobile Architecture | [`uber/RIBs`](https://github.com/uber/RIBs) |
| **Jaeger** | Distributed Tracing (CNCF) | [`jaegertracing/jaeger`](https://github.com/jaegertracing/jaeger) |
| **M3 / M3DB** | Large-Scale Metrics TSDB | [`m3db/m3`](https://github.com/m3db/m3) |
| **Apache Pinot** | Real-Time Distributed OLAP Datastore | [`apache/pinot`](https://github.com/apache/pinot) |
| **Apache Hudi** | Streaming Data Lakehouse Storage | [`apache/hudi`](https://github.com/apache/hudi) |
| **Peloton** | Unified Cluster Resource Scheduler | [`uber/peloton`](https://github.com/uber/peloton) |
| **Kraken** | P2P Docker Container Registry | [`uber/kraken`](https://github.com/uber/kraken) |
| **Ludwig / Horovod** | Declarative Deep Learning & Distributed GPU Training | [`ludwig-ai/ludwig`](https://github.com/ludwig-ai/ludwig) |
| **Kepler.gl / deck.gl** | GPU-Accelerated Geospatial Visualization | [`keplergl/kepler.gl`](https://github.com/keplergl/kepler.gl) |

---

## 📄 License

This project is licensed under the **MIT License** — feel free to fork, customize, and share for educational and system design interview preparation.