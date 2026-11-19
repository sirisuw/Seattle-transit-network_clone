# How Resilient Is Seattle's Transit Network?

A network analysis of the King County Metro GTFS schedule feed: stops are nodes,
consecutive stops on a trip are ties. The project measures the network's structure
(degree, betweenness, components, path lengths) and stress-tests it with random,
degree-targeted, and betweenness-targeted stop removals.

**Headline findings:** the SR 520 floating-bridge corridor is the region's top
chokepoint (all top-5 betweenness stops), and degree-targeted removal fragments
the network fastest — the opposite of the usual scale-free textbook story.

## Files

- `final_project_final.Rmd` — the R Markdown project (knit this). Rename to
  `index.Rmd` if you want GitHub Pages to serve it as the landing page.
- `nodes.csv` / `edges.csv` — the graph data; keep them beside the Rmd when knitting.
- `figures/` — 300 dpi charts (resilience curve, path-length panels, geographic
  betweenness map, before/after removal map).
- `PUBLISHING.md` — knitting + GitHub Pages / Posit Connect Cloud steps and checklist.
- `dataset_paragraph.md` — the proposal's data-set section (for the proposal docx).
- `LEARNINGS.md` / `STATUS.md` — methods log and results summary (reference).

## Data

King County Metro GTFS feed, version FAL26-161.5 (service Oct 29, 2026 – Mar 26, 2027),
downloaded November 18, 2026 from https://metro.kingcounty.gov/GTFS/google_transit.zip
Transit scheduling, geographic, and real-time data provided by permission of King County.

Analysis in R (igraph, ggraph). Exploratory verification in Python (networkx, pandas).
