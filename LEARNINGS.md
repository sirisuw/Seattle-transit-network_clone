# Transit Project — Learnings Log

Running record for Ty's Seattle transit network resilience project (King County Metro GTFS, feed FAL26-161.5). Started 2026-11-18.

## 2026-11-18 11:33 PST — Step 1: Graph build (networkx)
- Created `venv/` in the project folder and installed networkx 3.7 / pandas 3.1.1 / matplotlib 3.11.2 (system python has pandas but no networkx, and PEP 668 blocks a plain pip install).
- Built the graph with the starter code's logic: nodes = stop_ids served by trips, ties = unique undirected consecutive stop pairs from stop_times.txt (sorted by trip_id, stop_sequence). Saved `transit_graph.graphml` (with stop name + lat/lon as node attributes) and `graph_stats.json`.
- Results match the earlier hand count exactly: **6,271 nodes, 7,025 ties, 4 components, largest component 6,238 stops (99.47%)**, average degree 2.24, max degree 14, 83.64% of stops have degree exactly 2.
- **Average clustering coefficient = 0.0141** — essentially zero. As hypothesized, the stop graph is chain-like: stops on a route almost never close into triangles. Good methods-note material: this network is nothing like a social network topologically.

## 2026-11-18 11:35 PST — Data check: parent_station is EMPTY
- stops.txt has a parent_station column, but it is blank for all 6,271 stops (location_type is 0 for every stop too). So the merge-rule experiment cannot use parent_station; falling back to the planned alternative: merge stops with identical stop_name + coordinates rounded to 3 decimals (~111 m). This is itself a finding worth reporting in the proposal: King County's feed does not populate the GTFS station hierarchy.

## 2026-11-18 11:36 PST — Step 2: Betweenness centrality (exact)
- Ran EXACT betweenness (networkx Brandes algorithm, normalized) on the full graph — finished in 99.5 s, no sampling needed. Full values in `betweenness_all.json`; ranked list in `top20_betweenness.csv`.
- **Top of the ranking is dominated by the SR 520 floating-bridge corridor**: #1 SR 520 & 92nd Ave NE (0.195, degree 6), #2 SR 520 & Evergreen Point Rd (0.186, degree 7), #3 SR 520 & Evergreen Point Rd — the opposite-direction stop (0.163, degree 6), #4 SR 520 & 92nd Ave NE (0.141, degree 4), #5 SR 520 & NE 40th St (0.137, degree 4), plus #19 the other NE 40th St stop (0.109). The bridge across Lake Washington is the network's single biggest chokepoint, exactly the "hidden bridge" pattern the project is looking for.
- **The brokers are real**: 18 of the top 20 qualify as high-betweenness/modest-degree bridges (betweenness >= 0.005 with degree <= 6). Most extreme: **Bellevue Transit Center – Bay 6 has degree 2 but betweenness 0.128 (rank 9)** — a stop that looks like an ordinary chain stop by degree but carries an outsized share of shortest paths. Its pair, Bellevue TC Bay 11, is #15 (0.115, degree 4).
- Second cluster: the Renton corridor (Rainier Ave S & S 7th St #6, Renton Transit Center Bay 4 #7, Southport Dr stops #8/#16, Park Ave N #10) — the chain of stops linking Renton's routes together is a structural bottleneck.
- **Surprise: no downtown Seattle stop and no Link light rail station appears in the top 20.** Interpretation: downtown has many parallel paths (a grid of overlapping routes), so shortest paths spread out and no single stop dominates. Betweenness concentrates instead in the suburbs, where service is a few long chains hanging off single arterial corridors. Worth a paragraph in the final report.
- Also visible in the ranking: direction-split duplicate stops (two separate stop_ids for SR 520 & Evergreen Point Rd, two for NE 40th St) each carry large betweenness — a preview of why the merge rule matters.

## 2026-11-18 11:37 PST — Step 3: Resilience simulation (the headline result — and a surprise)
- Setup (`step3_resilience.py`, data in `resilience_results.csv`, chart in `resilience_curve.png`): removed stops in 1% steps from 0% to 50% under three strategies, tracking the largest connected component (LCC) as a fraction of the original 6,271 stops. (a) Random: 10 trials, seed 20261118, mean reported with min–max band. (b) Betweenness-targeted: static order from the exact betweenness computed once. (c) Degree-targeted: static order by degree.
- **The result inverts the textbook scale-free story.** Largest component remaining:
  - 5% removed → random 0.834 | betweenness 0.806 | **degree 0.255**
  - 10% removed → random 0.637 | betweenness 0.548 | **degree 0.069**
  - 20% removed → random 0.264 | betweenness 0.093 | degree 0.009
  - First drop below 50% of the original network: degree-targeted at **5%**, betweenness-targeted at 12%, random at 14%.
- Why degree wins here (interpretation for the report): this network's "hubs" are not optional extras — the ~380 stops with degree >= 4 (transit centers, park & rides, junctions where routes cross) are the glue holding thousands of degree-2 chain stops together. Removing the whole junction layer at once (which is what the first ~5% of the degree order does — there are only 127 stops with degree >= 5 and 382 with degree >= 4) shatters every chain simultaneously; the LCC falls off a cliff between 4% (0.635) and 5% (0.255). By contrast, the top betweenness stops are clustered in series on a few corridors (six of the top 20 are on SR 520 alone): cutting the same chain five times detaches the same tail five times, so a static betweenness attack wastes removals. A recomputed (adaptive) betweenness attack would likely do better — worth listing as a limitation/future step.
- Also note: the network is not especially robust even to random failure (LCC halves by ~14% random removal). With average degree 2.24 there is little redundancy to absorb losses — very different from the classic "scale-free networks shrug off random failure" result, because this network is not scale-free by degree (max degree 14).
- Supporting stat: only 167 stops (2.7%) are articulation points (cut vertices) — overlapping routes give many segments a parallel alternative, which is why random removal degrades the network gradually instead of instantly.

## 2026-11-18 11:38 PST — Step 4: Clustering + path lengths
- Average clustering coefficient (from Step 1): **0.0141**.
- Diameter and average shortest path of the largest component, computed EXACTLY (all-pairs BFS, 23 s — no sampling needed; `path_stats.json`): **diameter = 93 stops; average shortest path = 28.67 stops.** Long paths are the flip side of the chain structure: a cross-county journey passes through dozens of stops, and there are few shortcuts.

## 2026-11-18 11:39 PST — Step 5: Merge-rule experiment (name + rounded coordinates)
- parent_station is unusable (blank for every stop — see 11:35 entry), so merged stops sharing (stop_name, lat rounded to 3 dp, lon rounded to 3 dp) — roughly a 111 m cell (`step5_merge.py`, `merge_stats.json`, `transit_graph_merged.graphml`).
- Result: **5,390 nodes** (881 stop_ids absorbed; 879 locations had 2–3 stop_ids, e.g. opposite-direction pairs and transit-center bays) and **6,685 ties**. Components stay at 4; largest component 5,361 stops = **99.46%** (vs 99.47% unmerged — merging does not change the connectivity story). Average degree rises 2.24 → 2.48 because merged stops pool their connections.
- Takeaway for the proposal: report BOTH counts — 6,271 stops / 7,025 ties on raw stop_ids, 5,390 locations / 6,685 ties after merging — and treat the merged graph as the more realistic "physical stop" network for the final analysis.

## 2026-11-18 11:40 PST — What the 4 components actually are
- Mainland network: 6,238 stops.
- Vashon Island bus network: 29 stops — a real bus system, but no stop_id is shared with the mainland, so it sits disconnected (in reality riders transfer via the Vashon ferry dock).
- West Seattle Water Taxi: 2 stops. Vashon Water Taxi: 2 stops. Both dock at Colman Dock but under different stop_ids ("West Seattle Water Taxi & Colman Dock" vs "Vashon Island Water Taxi & Colman Dock"), so they connect to nothing in the stop graph.
- Lesson: the "4 components" are a data-modeling artifact of GTFS stop_ids, not a claim that Vashon has no transit. Walking transfers at shared docks/stations are invisible unless stops are merged or transfer edges are added — a clean, concrete limitation to state in the proposal (and the coordinate merge at 3 dp does NOT fix the water taxi case, because the two Colman Dock stop_ids have different names and slightly different coordinates).

## Open threads / suggested next steps
- Adaptive (recomputed) betweenness attack for a fairer comparison with the degree attack.
- Repeat the resilience simulation on the MERGED graph to check the degree-cliff survives merging.
- Map figure: plot stops geographically (lat/lon attributes are saved in the GraphML), sized by betweenness — the proposal's showcase visual.
- Weighted extension: weight ties by number of trips (from trips.txt/stop_times counts) and compare.

## 2026-11-18 13:14 PST — Step 6: second fragmentation measure (sampled-pair path length)
- Extension of the step-3 simulation with identical setup (0–50% removal, 1% steps; random = mean of 10 trials, seed 20261118; static betweenness and degree orders). New: a fixed sample of 300 stop pairs drawn from the ORIGINAL largest component (seed 20261118). At each step: share of sampled pairs whose endpoints both survive and are still connected, and the mean shortest path (stops) over the pairs still connected. Code: `step6_path_resilience.py`; data: `path_resilience_results.csv`; two-panel chart: `path_resilience.png`.
- Sanity check: sampled baseline average path at 0% removed is 29.7 stops, close to the exact all-pairs value of 28.67 — the 300-pair sample is representative.
- **Pair connectivity collapses much faster than the largest-component fraction.** Share of sampled pairs still connected falls below 50% at just 4% removed (degree attack), 7% (betweenness), 8% (random) — vs the LCC halving at 5% / 12% / 14%. Reason: a pair dies if EITHER endpoint is removed or the path between them is cut, so pair connectivity compounds two failure modes. At 5% removed: random 0.675, betweenness 0.627, degree 0.123. By 20% removed the degree attack leaves 0 of 300 sampled pairs connected.
- **Trips get much longer before they become impossible (under targeted attack).** Mean path among still-connected pairs spikes to 94.3 stops under the betweenness attack (~6% removed, vs 29.7 baseline) and 158.4 stops under the degree attack (~4% removed, computed over the few surviving pairs) — cutting bridges forces long detours along the chains. Random removal barely stretches paths (peak 33.7 at 4%).
- After the peak, the average path FALLS under all strategies — a survivorship effect, not recovery: only short, local pairs remain connected, so the mean drops (random: 18.6 stops at 30% removed, over just 1.1% of pairs). Any write-up must present panels A and B together, or the falling path length looks like improvement.
- Ranking note: on THIS measure the degree attack is again fastest, betweenness second, random slowest for pair survival — consistent with step 3, and betweenness does relatively worse here than on the LCC measure because its cuts sever specific origin–destination paths even when both endpoints stay in the largest component.

## Correction entry — 2026-11-18 ~15:45 PST (R knit verification)
Ty knitted final_project.Rmd in R (76 s) and audited the prose against the code output. Three discrepancies found and fixed in final_project_final.Rmd:
1. **igraph naming bug:** `names(sort(V(g)$betweenness_r))` returns nothing in Ty's igraph version (attribute vector is unnamed), so the betweenness attack silently removed nothing. Fix: build orders as `V(g)$name[order(-value, V(g)$name)]` for betweenness, degree, and top5 — the name tie-break also makes results reproducible.
2. **Resilience values differ between the Python exploratory run and the R run** (ranking identical): degree halves the network at 5% removed with ~42% remaining at 5% (Python said ~26%); betweenness halves at ~14% (Python: 12%); random at ~17% (Python: 14%). Cause: tie-breaking among thousands of equal-degree stops. The Rmd prose now uses the R values, since the R page is the submitted artifact, and a reproducibility note about tie sensitivity was added.
3. **Merged tie count:** the R merge chunk prints 6,976 ties, not 6,685. Replicating the R chunk's exact logic in Python on nodes.csv/edges.csv also gives 5,390 nodes / 6,976 ties, so the earlier Python step5 grouping differed slightly; 6,976 is authoritative for the page. Node count (5,390) agreed in both.
Lesson: prose numbers must come from the same code that ships in the artifact; cross-language verification numbers are a check on rankings, not a source for prose.
