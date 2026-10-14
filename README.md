# Uber System Design — 3D Explorer

I wanted to understand how Uber actually matches a million+ riders and drivers in real time, so I read through their engineering blog posts and open-source repos and built a small interactive visualizer to make sense of it.

It's a single-file Three.js app — no build step, just open the HTML — plus a longer writeup I turned into a Notion doc.

## What's in here

**`uber_system_design_explorer.html`** — an interactive 3D scene with four modes:

- **H3 hex grid** — Uber's geospatial index, visualized as actual hexagons you can orbit around
- **Hungarian matching** — run a batch and watch the matching arcs draw between drivers and riders
- **Surge + Ringpop shards** — see how cells get hashed across shards and how surge pricing gets smoothed spatially
- **Kalman + Viterbi** — simulates noisy GPS getting cleaned up and snapped onto roads

## Why hexagons, not squares

This was the part that got me down the rabbit hole in the first place. With a square grid, the diagonal neighbors are farther away than the adjacent ones (by a factor of √2), so "find everything within 3km" queries get distorted depending on direction. Hexagons have six equidistant neighbors, so the math is actually clean — Uber's H3 library uses this, projected onto an icosahedron to cover the globe.

## Why not just match the nearest driver

Naive first-come-first-served matching is greedy and actually hurts total throughput — you can end up stealing a driver who would've been perfect for someone two minutes away, just because they asked first. Uber batches requests in short windows (a few seconds) and solves it as a bipartite matching problem with the Hungarian algorithm, optimizing for everyone's wait time at once rather than one rider at a time.

## Other stuff covered in the report

- RIBs — Uber's mobile architecture pattern
- CRP + DeepETA for routing and arrival predictions
- Extended Kalman Filter + Viterbi HMM for cleaning up GPS in places like dense downtowns
- Schemaless storage + Cadence workflows for the payment/ledger side

## Running it

No build step needed — the whole thing is one HTML file.

```bash
git clone https://github.com/Asterioxer/uber-architecture-visualizer
cd uber-architecture-visualizer
python3 -m http.server 8080
```

Then open `http://localhost:8080/uber_system_design_explorer.html`.

There's also a "Copy for Notion" button in the app if you want to pull the writeup into your own Notion workspace — it keeps headings, math, and tables intact.

## Open-source projects referenced

Most of what's in the report comes from Uber's own public repos and engineering blog:

- [uber/h3](https://github.com/uber/h3) — the hexagonal spatial index
- [uber-node/ringpop-node](https://github.com/uber-node/ringpop-node) — consistent hashing + SWIM gossip
- [uber/cadence](https://github.com/uber/cadence) — workflow orchestration
- [uber/RIBs](https://github.com/uber/RIBs) — mobile architecture
- [apache/pinot](https://github.com/apache/pinot) — real-time OLAP

## License

MIT — fork it, use it for interview prep, whatever's useful.
