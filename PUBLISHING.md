# Publishing the Final Project

## Files that must stay together

- `final_project.Rmd` — the project (rename it if you like, e.g. `index.Rmd` for GitHub Pages)
- `nodes.csv` — 6,271 stops (id, name, x, y, degree, betweenness)
- `edges.csv` — 7,025 ties (from, to)

The Rmd reads the two CSVs by relative path, so keep them in the same folder when you knit and when you publish.

## Knit it

Open `final_project.Rmd` in RStudio and press **Knit** (or run
`rmarkdown::render("final_project.Rmd")`). Output: `final_project.html` with a
floating table of contents.

Slow chunks to expect: the resilience simulation and the pair-statistics chunk
(they re-run component/shortest-path calculations at every removal step). If
knitting takes too long, set those two chunks to `cache = TRUE` so re-knits are
fast while you edit the text.

Packages needed: `readr`, `dplyr`, `tidyr`, `tibble`, `igraph`, `ggraph`,
`ggplot2`, `knitr`, `rmarkdown`.

## Option A — GitHub Pages (free, good for employers)

1. Create a GitHub repo, e.g. `seattle-transit-network`.
2. Add `final_project.Rmd`, `nodes.csv`, `edges.csv`, and the knitted
   `final_project.html` renamed to `index.html`.
3. Repo **Settings → Pages** → deploy from the `main` branch, root folder.
4. Your page appears at `https://<username>.github.io/seattle-transit-network/`.
   Submit that link.

## Option B — Posit Connect Cloud (free tier)

1. Sign in at <https://connect.posit.cloud> with GitHub.
2. **New Content → R Markdown**, point it at the repo containing the Rmd + CSVs
   (or upload them), and publish. It knits on their servers.
3. Submit the published page link.

## Before you submit — checklist

- [ ] Your name is in the YAML author field (and group members, if any).
- [ ] The page knits cleanly from a fresh R session (restart R, knit once).
- [ ] The King County credit line appears in the data section and figure captions.
- [ ] No figure uses the synthetic demo network — all plots come from the GTFS data.
- [ ] The submitted link opens in a private/incognito window (i.e., it's public).
