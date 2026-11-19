# How Resilient Is Seattle's Transit Network?

A network analysis of the King County Metro GTFS schedule feed. Each stop is a
node, and two stops are tied when a vehicle serves them back to back on a trip.
The project measures the network's structure (degree, clustering, components,
path lengths), ranks stops by betweenness centrality, and tests how the network
breaks when stops are removed at random, by highest degree, and by highest
betweenness.

**Live page:** https://sirisuw.github.io/Seattle-transit-network_clone/

## Main findings

- The SR 520 floating bridge corridor is the biggest chokepoint. All five of the
  highest-betweenness stops are on it.
- Removing the highest-degree stops fragments the network fastest. At 5% of stops
  removed, the largest connected piece falls from 99.5% to about 42% of stops.
  Betweenness-targeted removal halves it at about 14% removed, and random removal
  at about 17%.

## Files

| File | What it is |
|---|---|
| `index.html` | The knitted report (this is the page that GitHub Pages serves) |
| `index.Rmd` | The R Markdown source for the report |
| `nodes.csv` | 6,271 stops: id, name, x, y, degree, betweenness |
| `edges.csv` | 7,025 ties: from, to |

## Run it yourself

1. Keep `index.Rmd`, `nodes.csv` and `edges.csv` in the same folder.
2. Install the packages: `readr`, `dplyr`, `tidyr`, `tibble`, `knitr`, `igraph`,
   `ggraph`, `ggplot2`, `rmarkdown`.
3. Open `index.Rmd` in RStudio and click Knit, or run
   `rmarkdown::render("index.Rmd")`. It takes about a minute.

## Data

King County Metro GTFS feed, version FAL26-161.5 (service October 29, 2026 to
March 26, 2027), downloaded November 18, 2026 from
https://metro.kingcounty.gov/GTFS/google_transit.zip

Transit scheduling, geographic, and real-time data provided by permission of
King County.

Analysis in R with igraph and ggraph.
