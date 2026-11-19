# Transit Project — Status (2026-11-18, ~13:15 PST)

## Done
- [x] Graph built with networkx (venv in project folder): `transit_graph.graphml` — 6,271 nodes, 7,025 ties, 4 components, largest 6,238 (99.47%). Matches the earlier hand count.
- [x] Exact betweenness centrality for all stops (99.5 s, no sampling): `betweenness_all.json`, ranked `top20_betweenness.csv`.
- [x] Resilience simulation, 0–50% removal in 1% steps, 3 strategies: `resilience_results.csv` + chart `resilience_curve.png` (with King County credit line).
- [x] Average clustering (0.0141); exact diameter (93 stops) and average shortest path (28.67 stops) of the largest component: `path_stats.json`.
- [x] Merge-rule experiment: `merge_stats.json`, `transit_graph_merged.graphml`.
- [x] Running log: `LEARNINGS.md` (methods, numbers, surprises, limitations — ready to mine for the proposal).
- [x] Second fragmentation measure (step 6): 300 sampled stop pairs (seed 20261118) tracked through the same removal simulation — share still connected + average shortest path at each step: `path_resilience_results.csv`, two-panel chart `path_resilience.png`, code `step6_path_resilience.py`.

## Findings headlines
- **Top chokepoint is the SR 520 bridge corridor.** #1 stop: SR 520 & 92nd Ave NE (betweenness 0.195, degree only 6). Six of the top 20 stops are on SR 520. Second cluster: the Renton corridor (Renton Transit Center, Southport Dr, Rainier Ave S).
- **Brokers confirmed:** Bellevue Transit Center – Bay 6 has degree 2 but the #9-highest betweenness (0.128). 18 of the top 20 are high-betweenness / modest-degree bridges. No downtown or Link station makes the top 20 — downtown's parallel routes spread shortest paths out; fragility lives in the suburban chains.
- **Resilience result inverts the textbook story:** removing highest-DEGREE stops fragments the network fastest (largest component falls below 50% at just 5% removed, and to 0.069 by 10%), ahead of betweenness-targeted (below 50% at 12%) and random (14%; mean of 10 trials). Reason: the degree attack strips out the whole junction layer (only 382 stops have degree >= 4) that holds the chains together, while top betweenness stops sit in series on a few corridors, so a static betweenness attack wastes cuts on already-detached tails.
- **Merge rule:** parent_station is blank for every stop in this feed (a finding in itself). Merging by identical name + coordinates rounded to 3 dp gives **5,390 nodes / 6,685 ties**, largest component 99.46% — the duplicates are real (881 stop_ids absorbed) but merging doesn't change the connectivity picture.
- The 4 components: mainland (6,238), Vashon Island buses (29), and the two Water Taxi routes (2 stops each) — a stop_id modeling artifact, documented in LEARNINGS.md.
- **Second measure (step 6) corroborates and sharpens the story:** sampled-pair connectivity falls below 50% even earlier than the LCC does — at 4% removed (degree), 7% (betweenness), 8% (random) — and surviving trips get far longer under targeted attack before they become impossible (mean path peaks at 94 stops under betweenness, ~6% removed, vs 29.7 baseline). After the peak the average path falls only because just short local pairs survive — panels must be read together.

## Unfinished / blocked
- Nothing blocked. Optional extensions not yet run (listed in LEARNINGS.md): adaptive betweenness attack, resilience on the merged graph, geographic map figure sized by betweenness, trip-weighted ties.
- Note: the proposal docx (Seattle_Transit_Proposal.docx) lives in Ty's other conversation, not this workspace — the dataset paragraph in `dataset_paragraph.md` can be updated with the merged counts (5,390 / 6,685) alongside the raw counts whenever Ty is ready.
