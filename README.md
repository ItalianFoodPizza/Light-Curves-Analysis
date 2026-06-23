# Automated Exoplanet Detection via Light Curve Analysis

I built this project because I've always been drawn to the scale of the universe and the questions we haven't answered yet — whether other planets like Earth exist out there is one of those questions. This pipeline is my attempt to approach that question using real data from NASA's Kepler space telescope and write code that does the analysis automatically, the same way professional astronomers do it.

This is an independent research project built from scratch over several months as part of my STEM portfolio. Every result in this pipeline is independently detected by my code before being cross-checked against NASA's catalog — so nothing here is just borrowed from a database.

---

## What This Project Does

When a planet orbits a star, it periodically passes in front of that star from our perspective. This causes a tiny, repeating dip in the star's brightness — called a transit. By analyzing the brightness over time (called a light curve), we can detect these dips and figure out whether a planet is there, how long it takes to orbit, and how far it sits from its star.

This pipeline automates that entire process across hundreds of stars.

Specifically it:
- Downloads real photometric data directly from NASA's Kepler mission
- Cleans the data to remove noise and instrument artifacts
- Runs a Box Least Squares (BLS) algorithm to detect repeating transit signals
- Cross-matches detected periods against NASA's catalog to independently verify findings
- Calculates orbital distances using Kepler's Third Law
- Estimates habitable zone boundaries for each star using the Stefan-Boltzmann Law
- Ranks candidates by an Earth Similarity Index (ESI)

---

## How the Pipeline Works

### v1 — Proving the concept (Kepler-10)
I started with a star I knew had confirmed planets — Kepler-10. The goal was to verify that my pipeline could independently detect a known planet before applying it to unknown candidates. I downloaded one quarter of Kepler data, cleaned it, ran BLS, and folded the light curve to reveal the transit dip. My pipeline detected Kepler-10b with a period of ~0.837 days, matching the confirmed value.

### v2 — Scaling to 50 stars
Once I knew the method worked, I automated it across 50 of the brightest Kepler candidates. For each star the pipeline runs the full detection sequence automatically and outputs a ranked table of signals by strength. I then classified each result as strong, moderate, or false positive based on how closely my detected period matched NASA's catalog. About 80% were false positives — consistent with what published literature reports for single-quarter BLS analysis.

### v3 — Calculating physical properties
For all candidates with complete stellar data, I calculated:
- Orbital distance using Kepler's Third Law
- Stellar luminosity using the Stefan-Boltzmann Law
- Habitable zone inner and outer boundaries (calculated individually per star, not fixed at solar values)
- Planet radius in Earth radii and planet type classification

Running this across 2,303 candidates revealed that 73.1% of all Kepler candidates orbit closer to their star than Mercury orbits the Sun — a direct demonstration of Kepler's detection bias toward short-period planets.

### v4 — Independent verification (main pipeline)
This is the core of the project. Instead of trusting NASA's detected periods, I run BLS detection independently on each star's light curve and only calculate physical properties if my detection agrees with NASA's catalog within 20%. The cross-match also checks harmonics (0.5x, 1.5x, 2x, 3x the catalog period) since BLS sometimes finds aliases of the true period.

Out of 500 stars screened:
- **177 independently verified** (35.4%)
- **323 not verified** — BLS found an inconsistent signal
- **0 failed** — all stars had available data

### v5 — ESI scoring and final analysis
For all 177 verified candidates I calculated an Earth Similarity Index (ESI) combining planet radius similarity to Earth, proximity to the star's habitable zone, and stellar temperature similarity to the Sun. The highest ESI in my verified sample was 0.354 (K01619.01, a rocky planet about 0.66 Earth radii) — well below Earth (1.0) or Mars (~0.7), which is consistent with the detection bias finding.

---

## Key Findings

| Finding | Value |
|---|---|
| Stars screened | 500 |
| Independently verified | 177 (35.4%) |
| Habitable zone candidates | 0 |
| Planets orbiting closer than Mercury | 73.1% of all Kepler candidates |
| Kepler's Third Law | Verified visually from 177 real candidates |
| Highest ESI candidate | K01619.01 (ESI = 0.354) |

**The zero habitable zone result is not a failure — it's a finding.** It demonstrates two compounding detection biases:

1. Single-quarter BLS analysis systematically recovers only short-period planets because they produce the clearest signals with limited data
2. The sample is dominated by hot luminous stars whose habitable zones sit far beyond what Kepler can detect

This is exactly why next-generation missions like PLATO and the Habitable Worlds Observatory are being designed — to detect longer-period planets around quieter, cooler stars.

---

## Tools and Libraries

| Tool | Purpose |
|---|---|
| Python | Core language |
| Lightkurve | Kepler/TESS data access and light curve analysis |
| Astroquery | NASA Exoplanet Archive queries |
| Pandas | Data manipulation and results tables |
| NumPy | Numerical calculations (Kepler's Third Law, Stefan-Boltzmann) |
| Matplotlib | All visualizations |

---

## How to Run This

**Install dependencies:**
```bash
pip install lightkurve astroquery pandas numpy matplotlib
```

**Run in order:**
1. Start with `notebooks/lightcurve_test.ipynb` to verify your setup
2. Run `notebooks/v4_unified_pipeline.ipynb` for the main pipeline
3. Run `notebooks/v5_final_analysis.ipynb` for ESI scoring and final figures

The first run will download data from NASA and cache it locally in `data/lk_cache/`. Subsequent runs will use the cache and run significantly faster.

**Expected runtime:** 45–90 minutes for 500 stars on first run, ~15 minutes on cached runs.

---

## Data Source

All data comes from NASA's Kepler mission via the NASA Exoplanet Archive, accessed through the `astroquery` and `lightkurve` Python libraries. The Kepler mission observed over 150,000 stars for 4 years (2009–2018) specifically to find transiting exoplanets.

Kepler Objects of Interest (KOI) catalog:
https://exoplanetarchive.ipac.caltech.edu/

---

## Limitations

- **Single quarter of data** per star reduces sensitivity to long-period planets (beyond ~50 days)
- **20% period match threshold** may include some residual false positives from harmonic aliases
- **Sample selection** (kepmag < 14) skews toward bright, often luminous stars with distant habitable zones
- **Stellar property uncertainties** from the KOI catalog propagate into orbital distance and habitable zone calculations
- **BLS search cap at 50 days** means Earth-analog planets (365-day period) cannot be detected with this approach

---

## What I Learned

Going into this I knew I wanted to work with astronomical data, but I had never written Python before starting this project. Learning the language while building something I actually cared about made a significant difference — every new concept (Pandas, BLS, Kepler's Third Law in code) had an immediate application I could see working.

The most surprising finding was that zero habitable zone candidates is actually the most scientifically interesting result. I expected to find some — the fact that the pipeline found none, and that the reason is explainable through detection geometry and stellar physics, is a more complete scientific story than a list of candidates would have been.
