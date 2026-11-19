# External templates and the early-LSST template gap: a latency-parametrized forecast

```{abstract}
Rubin's Data Management strategy has been to build its difference-imaging templates from previous visits.
As a consequence, early in the survey many images cannot be differenced due to lack of templates,
and how long that lasts depends on two things: how quickly the survey accumulates visits (set by the observing
strategy), and how quickly those visits can be combined and deployed as templates in Alert Production (set by Data
Management's template strategy). This note quantifies the gap and the value of sourcing external $griz$ DECam
templates without assuming any particular Rubin template strategy. Instead, we describe Rubin template
availability at each position by two parameters: the number of visits a template needs ($N$) and the delay from having
$N$ visits to having a template in AP (the latency $L$), and we report alert completeness and
differenceable-visit fractions over a grid of both. The main result is that DECam templates make
year-1 alert completeness nearly insensitive to both parameters: across $L = 0$–12 months and
$N = 4$–10, completeness with DECam stays within 0.67–0.81, whereas without DECam it ranges from 0.75
down to 0.11. Applying DM's DP2 template-visit selection (a 1.4″ PSF ceiling and a transparency
cut) lowers Rubin-only completeness by 0.01–0.09 but changes it by at most 0.03 with DECam. The one
practical DM-side choice that matters is when a tract switches from DECam to Rubin templates: switching as soon
as any Rubin template exists in a tract removes most of DECam's gain in all-four-band coverage and
about a third of its year-1 completeness gain on a semiannual build calendar, while waiting for full Rubin
coverage keeps nearly all of both. The campaign's current curation, which withholds DECam templates from
tract-bands that already hold a DP2 template, costs less than one point of completeness against
full-coverage switching. DECam
therefore lets the Rubin template strategy be chosen for purity and operational
convenience at small cost in completeness. This is a living document and will be updated as the
template strategy and the year-1 observing strategy are settled.
```

:::{note}
**Draft for review.** Numbers are provisional and will change as the Rubin template strategy, the
year-1 observing strategy and the DECam deployment schedule are settled (see {ref}`sec-provenance`).
Comments are welcome in the DECam templates Slack channel or as issues on
[lsst/rtn-133](https://github.com/lsst/rtn-133/issues).
:::

## Purpose and scope

The DECam template campaign is going ahead, so this note is not an argument for it. Its purpose is to
quantify the factors that are still open (namely the Rubin incremental template strategy, the year-1
observing strategy and the DECam deployment details) and show what each choice costs in alert
performance, with and without DECam templates. It therefore avoids modelling any single Rubin
template policy. Instead it separates:

- **what the survey controls:** when each position reaches its $N$-th visit in each band;
- **what DM controls:** the visits a template needs, $N$, which visits qualify (quality cuts), and the
  latency $L$ from readiness to AP;
- **what DECam provides:** where external templates exist, how deep they are, and when they are deployed.

The baseline plan builds AP templates incrementally from LSSTCam data {cite:p}`RTN-011`, but those details are
not yet settled. The DECam
template campaign itself, including its image selection, deployment and alert-volume expectations, is
described in {cite:t}`DMTN-362`.

## Inputs

### Survey simulation and the effective start

The LSST {cite:p}`Ivezic2019` officially began on 2026 June 29. However a major storm kept the telescope off sky from 2026 July 14, and the survey
resumed on 2026 October 27 after also accomplishing a three-week planned maintenance period. We take the resumption (2026 October 27, 12:00 UTC night rollover) as the effective survey start $t_0$ in the survey simulation.

The simulated visits after $t_0$ come from the `baseline_v5.3.7_10yrs` simulation in the version frozen on
2026 October 31 (SHA-256 `eab6b75d…`); simulated visits before $t_0$ are dropped.
All real LSSTCam science visits taken before $t_0$ are used as pre-survey visits in template
construction, with the same quality cuts as LSST data. We take their metadata and quality metrics from
the Consolidated Database (94,966 visits, 2025 April 25 to 2026 July 14), which includes the first two
weeks of the LSST. Since these visits were taken during commissioning and early operations (when engineering studies and upgrades were being conducted in parallel), they are
heavily weighted to $i$ and $z$: at least five visits reach 14,400 deg² of the WFD footprint in $i$ and
12,012 deg² in $z$, but only 3,538 deg² in $g$ and 2,734 deg² in all four of $griz$.

At $t_0$ the only Rubin templates in AP are the DP2 templates, at the depth of their own input visits
(data up to 2026 January 1). The other pre-survey visits count toward readiness and template depth, but
reach AP templates only after $t_0$ (see {ref}`sec-availability`).

### DECam templates

DECam template inputs are taken from the 203,399 DELVE DR3 exposures in $griz$ {cite:p}`Tan2025,Anbajagane2025`
(and Drlica-Wagner et al., in prep.), already cut to PSF FWHM ≤ 1.5″ and $t_{\rm eff} \ge 0.2$. We
model the campaign's selection as the best-seeing six exposures per `lsst_cells_v2` patch and band.
DELVE flags or excludes exposures and CCDs (<https://github.com/delve-survey/exclude_list>) mainly when
they are contaminated by scattered and reflected light from bright stars, identified with the DES
ray-tracing procedure {cite:p}`Kent2020`, or when the sky-background estimate failed
{cite:p}`DrlicaWagner2022`. The campaign applies this exclude list {cite:p}`DMTN-362`; our model keeps
those exposures, which changes nothing material.

Template depths follow the effective-exposure-time formalism of {cite:t}`Neilsen2016`, with one offset
per band inside and outside the DES footprint. In $g$, $r$ and $i$ the offsets match the median
point-source depths of {cite:t}`Tan2025` (S/N = 10, converted to S/N = 5). No footprint-wide $z$
median is published for DELVE DR3; we calibrate $z$ at SZ-cluster positions to DES DR2 {cite:p}`DESDR2` and Legacy
Surveys {cite:p}`Dey2019` depths. The DECADE depth {cite:p}`Anbajagane2025` suggests our $z$ depths outside DES are about
0.1 mag conservative; we carry ±0.15 mag as the $z$ systematic and ±0.1 mag in $r$. These depths have
not yet been validated against built DECam templates.

**Deployment.** The campaign processes and deploys DECam templates in 30° × 30° regions of sky
("chunks"), in full-declination columns (Dec +30° to −90°) advancing in RA from the
RA 300°–330° column {cite:p}`DMTN-362`. Chunk 0 (RA 300°–330°, Dec 0° to +30°; about 500 deg² after the
campaign's cuts) was vetted at USDF in early November; as of 2026 November 19 it had been curated but not yet
configured for use in AP {cite:p}`DMTN-362`. The campaign's expected schedule, which we model from chunk 0 on
2026 November 10, allows about a week for the rest of that column and then about 10 days per full column, which
would complete the footprint on about 2027 March 7. The campaign's timing is set by the LIGO–Virgo–KAGRA
Intermediate Run 1 (IR1), now expected to begin in mid-January 2027 {cite:p}`DMTN-362`; the expected schedule
completes the footprint about seven weeks into the run. Unless stated otherwise, the results below model
the deployment as staged in observability order from 2026 November 15 to 2027 January 1. With the same
completion date, the campaign's chunk order changes year-1 completeness by at most 0.005 and
$G_{\rm visit}$ by 0.001; the campaign's expected schedule, finishing two months later, costs 1–2.5 points
of year-1 completeness (see {ref}`sec-switching`). When the footprint is complete matters; the
order does not. For comparison, deploying the whole DECam footprint at $t_0$
would raise year-1 completeness with DECam on the semiannual calendar from 0.763 to 0.800. DECam templates are used band by band wherever no
Rubin template is in AP, except where we model the campaign's per-tract switching rules
({ref}`sec-switching`). The minimum number of inputs for a DECam template to be accepted, $N_{\min}$,
is a parameter (default 1).

## Method

(sec-availability)=
### Readiness and template availability

For each WFD position $p$ (HEALPix {cite:p}`Gorski2005` $N_{\rm side} = 256$) and band $b$, the readiness time
$t_{\rm ready}(p, b, N)$ is the time of the $N$-th visit, counting pre-survey visits and simulated
visits from $t_0$. Two models turn readiness into a template in AP:

- **Model A, continuous incremental builds.** A template enters AP at $\max(t_{\rm ready}, t_0) + L$:
  positions already ready from pre-survey visits wait until $t_0 + L$, because apart from DP2 no
  templates had been built from those visits at $t_0$. At time $t$ the template is the coadd of all
  visits up to $t - L$, so it keeps deepening.
- **Model B, periodic builds.** Data cutoffs at $t_0$ and every $P$ months after, each deployed $D$
  months later. A position gets a template at the first deployment whose cutoff follows its readiness;
  the template is the coadd up to that cutoff. Positions ready at $t_0$ get their template at $t_0 + D$;
  for positions that become ready during the survey the latency averages about $P/2 + D$.

In both models DP2 templates are used from $t_0$ wherever DP2 built them, until the model's own template
for that position, built from the same visits plus later ones and so deeper, reaches AP.

We present results against $L$ and mark each Model B case at its effective latency: the Model A latency
that gives the same $G_{\rm visit}$. Because Model B's first build, at $t_0$, delivers the pre-survey head
start after only $D$ months, its effective latency is shorter than $P/2 + D$. Placed this way, Model B
points lie on the Model A curves for the differenceable fractions, but for Rubin-only completeness they
can sit well above them (by up to 0.14 for annual builds deployed three months after their cutoff), so
for completeness the mapping is only approximate. As a reference case throughout we use the
**semiannual calendar**: a first build from data up to $t_0$, deployed on 2027 January 1, then semiannual
builds, each deployed about three months after its cutoff. Its effective latency is 4.2 months
(5.9 months if matched on $G_{\rm visit,4}$).

### Template visit selection

Which visits may enter a Rubin template is a third DM-side input. We consider two settings: **none**
(every visit counts) and **DP2/DRP**, the selection DM used for the DP2 templates and set in the current
`drp_pipe` LSSTCam configuration (`BestSeeingQuantileSelectVisitsTask`): PSF FWHM ≤ 1.4″,
`effTimeZeroPointScale` ≥ 0.75, and the best-seeing third of the qualifying visits, but at least 12. In
the first two years nearly every position has 12 or fewer qualifying visits, so the hard cuts matter and
the quantile rarely does. With the cuts on, $N$ counts qualifying visits only, and template depth comes
from the selected visits. Pre-survey visits use their measured PSF size and zero point. For simulated
visits the PSF cut uses `seeingFwhmGeom` (closest to DM's PSF-based FWHM), and the transparency term
reflects airmass only, because the simulation applies no partial-cloud extinction. The simulated pass rates
(0.93 in $griz$ in year 1, against 0.72 for the pre-survey visits) are therefore an upper limit. The
first three survey nights after $t_0$ (dayobs 20261027, 20261029 and 20261030) failed the transparency cut
at 8% in $griz$, almost all in one half-night; we use
those rates only for a stress test. Whether incremental AP templates will use this selection is not
yet decided.

### Metrics

- **Differenceable visits.** $G_{\rm visit}$: the gain from DECam in the fraction of WFD $griz$ visits
  in years 1–2 that have a template in their own band at the visit's time. $G_{\rm visit,4}$: the same,
  requiring templates in all four of $griz$ at that position, which is what colour-based classification
  needs.
- **Alert completeness.** Transients with asymmetric Gaussian light curves on a grid of 33 peak
  magnitudes (17–25) and 12 timescales (2–100 d FWHM) are injected at random WFD positions and peak
  times. A transient is detected if at least two epochs more than 30 min apart exceed the
  difference-image $5\sigma$ limit, with the template's noise included. The **reference** differences
  every visit against an uncontaminated template built from all visits of that band in years 1–5,
  excluding the transient's active window (peak −30 d to +3 FWHM), and requires that template to have at
  least $N$ visits. **Completeness** is the fraction of reference detections also detected at alert
  time. At alert time the template follows the model in force and includes any transient light in its
  input visits (flux-subtraction contamination model).
- **Why transients are missed.** Each reference detection missed at alert time is assigned to *no
  template* (fewer than two epochs in the active window had any template), *too shallow*, or
  *contamination* (recovered when contamination is switched off).

Unless stated otherwise, completeness is the flat mean over the 33 × 12 grid for transients peaking in
year 1, with $N = 5$ and $N_{\min} = 1$. Pooled values (weighted by the injected population) are higher and
are given where noted.

## Results

### How fast the survey becomes template-ready

```{list-table} Readiness in all four of griz, WFD footprint
:header-rows: 1

* - N
  - Ready at t0
  - Ready by +6 mo
  - Ready by +12 mo
  - Median wait (months)
* - 4
  - 18.6%
  - 63.8%
  - 89.7%
  - 4.2
* - 5
  - 13.7%
  - 52.3%
  - 80.6%
  - 5.4
* - 6
  - 10.7%
  - 40.6%
  - 68.4%
  - 6.6
* - 10
  - 3.2%
  - 6.9%
  - 24.3%
  - 17.6
```

At $N = 5$, $g$ (median 5.1 months) and $u$ (7.1 months) are the slow bands; $i$ and $z$ are mostly
ready before $t_0$ from pre-survey visits (ready, but apart from DP2 not yet built into templates at $t_0$;
see {ref}`sec-availability`). Across five years, 2.3% of WFD $griz$ visits are taken
before their position is ready at $N = 5$, rising to 17.6% in $u$, for which DECam provides no
templates.

```{figure} figures/fig_floor_readiness.png
:name: fig-readiness

Readiness for $N = 4$, 5, 6 and 10, counting pre-survey and simulated visits. Top: months after $t_0$ (2026 October 27) until every one of $griz$ has its $N$-th visit, per WFD position (grey: ready at $t_0$; red: not within 24 months). Bottom: cumulative fraction of the WFD footprint ready against time, per band and for all of $griz$ (dashed). Maps use a Mollweide projection centred on RA 0°, with RA increasing to the left.
```

{numref}`fig-readiness-decam` shows the same quantities with DECam templates included: the time until a
template is in AP in every one of $griz$, with Rubin templates at $L = 0$ (DP2 at $t_0$, others at
readiness but no earlier than $t_0$) and DECam templates on the campaign's expected schedule with its
curation as implemented ({ref}`sec-switching`). With DECam, 71% of the WFD footprint has templates in
all four bands by +3 months at $N = 5$, against 26% from Rubin alone, and the median wait falls from
5.4 to 2.0 months. On the expected IR1 start (2027 January 15), 7,666 deg² of the WFD footprint still
lacks templates in all of $griz$ at $N = 5$, against 15,375 deg² with Rubin templates only; by the
campaign's expected completion (2027 March 7) this falls to 1,479 deg², against 12,378 deg². Where a
full-coverage switch withdraws DECam before a position's own Rubin template arrives (2.0–5.7% of WFD
positions for $N = 4$–10), the map shows the time from which a template is continuously available.

```{list-table} Template availability in all four of griz with DECam, WFD footprint (Rubin-only readiness from the table above in brackets)
:header-rows: 1

* - N
  - At t0
  - By +3 mo
  - By +6 mo
  - By +12 mo
  - Median wait (months)
* - 4
  - 20.5% (18.6%)
  - 74.9% (36.9%)
  - 96.4% (63.8%)
  - 99.4% (89.7%)
  - 1.8 (4.2)
* - 5
  - 16.7% (13.7%)
  - 70.7% (26.3%)
  - 94.6% (52.3%)
  - 98.7% (80.6%)
  - 2.0 (5.4)
* - 6
  - 14.7% (10.7%)
  - 67.5% (18.5%)
  - 92.6% (40.6%)
  - 97.7% (68.4%)
  - 2.3 (6.6)
* - 10
  - 12.2% (3.2%)
  - 62.0% (4.1%)
  - 84.3% (6.9%)
  - 92.9% (24.3%)
  - 2.5 (17.6)
```

The values with DECam at $t_0$ include DP2 templates at positions that are not yet ready, which the
Rubin-only readiness does not.

```{figure} figures/fig_floor_readiness_decam.png
:name: fig-readiness-decam

As {numref}`fig-readiness`, with DECam templates. Top: months after $t_0$ (2026 October 27) until a template is in AP in every one of $griz$, from Rubin ($L = 0$; DP2 at $t_0$) or DECam (campaign's expected schedule, $N_{\min} = 1$; no DECam in tract-bands holding a DP2 template, full-Rubin-coverage switching elsewhere); grey: in AP at $t_0$; red: not within 24 months. Bottom: cumulative fraction of the WFD footprint with a template, per band and for all of $griz$ (black dashed), with the Rubin-only readiness for all of $griz$ from {numref}`fig-readiness` (grey dashed); vertical lines mark the expected IR1 start (2027 January 15) and the expected DECam completion (2027 March 7). RA increases to the left.
```

### DECam value against latency

```{list-table} Gain from DECam in differenceable visits, years 1–2 (N = 5)
:header-rows: 1

* - Latency L
  - G_visit
  - G_visit,4
* - 0
  - 0.040
  - 0.177
* - 1 month
  - 0.068
  - 0.232
* - 3 months
  - 0.114
  - 0.305
* - 6 months
  - 0.168
  - 0.338
* - 12 months
  - 0.335
  - 0.522
* - Semiannual calendar (effective L ≈ 4.2 mo)
  - 0.136
  - 0.337
```

As {numref}`fig-lat-gain` shows, even with zero latency DECam adds 0.18 to the all-four-band fraction: this is set by the survey speed and strategy, not by DM. The gain then grows with latency. Single-band $G_{\rm visit}$ grows fastest at long latency,
because the pre-survey head start, mostly in $i$ and $z$, reaches AP only at $t_0 + L$; few positions
were ready in all four bands at $t_0$, so this matters much less for $G_{\rm visit,4}$.

```{figure} figures/fig_lat_gain.png
:name: fig-lat-gain

Gain from DECam templates in differenceable visits over years 1–2, $G_{\rm visit}$ (left) and $G_{\rm visit,4}$ (right), against Rubin template latency $L$ (Model A) for $N = 4$, 5 and 6. Model B points are placed at their effective latency (the Model A latency with the same $G_{\rm visit}$); the star marks the semiannual calendar.
```

### Alert completeness against latency and N

```{list-table} Year-1 alert completeness (flat mean)
:header-rows: 1

* - Latency L
  - Rubin only, N = 4 / 5 / 6 / 10
  - Rubin + DECam, N = 4 / 5 / 6 / 10
* - 0
  - 0.75 / 0.71 / 0.68 / 0.53
  - 0.81 / 0.80 / 0.80 / 0.77
* - 3 months
  - 0.62 / 0.58 / 0.54 / 0.37
  - 0.78 / 0.77 / 0.77 / 0.74
* - 12 months
  - 0.14 / 0.14 / 0.13 / 0.11
  - 0.67 / 0.67 / 0.68 / 0.69
```

Without DECam, completeness falls steeply with both latency and $N$; at $L = 12$ months no Rubin template
other than DP2's reaches AP in year 1, and Rubin-only completeness falls to 0.11–0.14. With DECam the
surface is nearly flat: moving from $N = 4$ to $N = 6$ costs 7 points at $L = 0$ without DECam and about 1 point with it.
These numbers describe completeness for real transients only; the reasons to prefer a larger $N$
(artifact rejection, PSF homogeneity, purity) are not modelled here (see {ref}`sec-limitations`). The
result is best read as the completeness cost of each choice of $N$ and $L$, which DECam keeps small.

```{figure} figures/fig_lat_completeness.png
:name: fig-lat-completeness

Year-1 alert completeness (flat mean over the transient grid) against latency $L$ (Model A), Rubin templates only (left) and Rubin plus DECam (right), for $N = 4$, 5, 6 and 10. Model B points are placed at their effective latency from {numref}`fig-lat-gain`; the star marks the semiannual calendar ($N = 5$).
```

```{figure} figures/fig_lat_completeness_vs_N.png
:name: fig-lat-completeness-vs-n

Year-1 alert completeness against the number of visits per Rubin template, $N$, at latencies of 0, 1, 3 and 6 months, with DECam templates (solid) and Rubin templates only (dashed). Purity, the main reason to prefer a larger $N$, is not modelled.
```

### Why transients are missed

```{list-table} Year-1 misses as a fraction of reference detections (N = 5, pooled)
:header-rows: 1

* - Case
  - Rubin only: no template / too shallow / contamination
  - Rubin + DECam: no template / too shallow / contamination
* - L = 0
  - 0.029 / 0.083 / 0.039
  - 0.012 / 0.046 / 0.022
* - L = 1 month
  - 0.062 / 0.111 / 0.013
  - 0.025 / 0.057 / 0.003
* - L = 3 months
  - 0.133 / 0.146 / 0.006
  - 0.031 / 0.072 / 0.001
* - L = 6 months
  - 0.299 / 0.153 / 0.003
  - 0.045 / 0.083 / 0.000
* - L = 12 months
  - 0.691 / 0.118 / 0.000
  - 0.089 / 0.094 / 0.000
* - Semiannual calendar
  - 0.138 / 0.159 / 0.001
  - 0.029 / 0.075 / 0.000
```

Without DECam, templates that exist but are too shallow cost more than missing ones only up to about
three months; at longer latencies missing templates dominate. With DECam, shallow templates are the
larger loss at every latency. Self-contamination matters only for latencies of a month or less, so instantaneous builds
are not free for slow transients.

```{figure} figures/fig_lat_miss_reasons.png
:name: fig-lat-miss-reasons

Why year-1 transients are missed at alert time ($N = 5$): no template, template too shallow, or self-contamination, as fractions of reference detections (pooled), at Model A latencies of 0–12 months and for the semiannual calendar, with Rubin templates only (left) and with DECam (right).
```

### Effect of template visit selection

```{list-table} Effect of DM's DP2/DRP template-visit selection (N = 5, all of griz for readiness)
:header-rows: 1

* - Quantity
  - None
  - DP2/DRP
  - DP2/DRP + bad-night stress test
* - WFD area with ≥5 qualifying pre-survey visits in all of griz
  - 2,734 deg²
  - 970 deg²
  - 970 deg²
* - WFD ready in all of griz by +6 months
  - 0.523
  - 0.415
  - –
* - Median wait for all of griz (months)
  - 5.4
  - 6.4
  - –
* - G_visit,4: L = 0 / semiannual calendar
  - 0.177 / 0.337
  - 0.213 / 0.372
  - 0.321 / 0.470
* - Completeness, Rubin only: L = 0 / 3 mo / calendar
  - 0.714 / 0.577 / 0.547
  - 0.677 / 0.512 / 0.464
  - 0.651 / 0.494 / 0.455
* - Completeness, Rubin + DECam: L = 0 / 3 mo / calendar
  - 0.803 / 0.774 / 0.763
  - 0.792 / 0.759 / 0.741
  - 0.790 / 0.753 / 0.740
```

The cuts remove most of the pre-survey head start: only 970 deg² of the WFD footprint has five
qualifying pre-survey visits in all four bands, against 2,734 deg² without cuts. Readiness slows most in
$g$ and $r$, the bands with the worst seeing. Without DECam, year-1 completeness drops by 0.01–0.09
across the grid (0.04–0.08 in the table); with DECam it drops by at most 0.03, and DECam's own gain grows: on the
semiannual calendar $G_{\rm visit}$ rises from 0.136 to 0.176 (about 30%) and $G_{\rm visit,4}$ from 0.337
to 0.372. With the cuts, the semiannual calendar's effective
latency is 4.7 months, against 4.2 without. We applied a stress test that fails whole simulated nights at the per-band
transparency fail rates of those first three nights after $t_0$ (25% in $g$, 29% in $r$, under 1% in $i$ and $z$;
three random seeds; failing half-nights instead gives similar results). Those rates come from three nights dominated by one bad
half-night, so the column is an upper bound on weather losses, not an estimate. It mainly moves
$G_{\rm visit,4}$; completeness changes by about 0.03 at most without DECam and by less than 0.01 with it.

(sec-switching)=
### Switching tracts from DECam to Rubin templates

The campaign uses one template style per tract and band, never mixing Rubin and DECam templates within a
tract {cite:p}`DMTN-362`. In curating chunk 0 the campaign withheld DECam templates from tract-bands
that already hold a DP2 template {cite:p}`DMTN-362`. When other tract-bands switch to Rubin templates is
not yet fixed, and it turns out to be the DM-side choice that matters most. We compare five rules, all with
the campaign's expected deployment schedule (chunk 0 from 2026 November 10, complete about 2027 March 7). For reference, the table
also gives per-position substitution with the observability-order deployment used elsewhere. The rules are:

- **per position:** each position uses its own Rubin template where one exists and DECam otherwise
  (the substitution used elsewhere in this note);
- **any Rubin template:** a tract-band switches to Rubin as soon as any position in it has a Rubin
  template, and its DECam templates are withdrawn;
- **full Rubin coverage:** a tract-band stays on DECam until Rubin templates cover at least 95% of its WFD
  positions, then switches entirely to Rubin; before the switch, positions without a DECam template use
  their Rubin template if they have one;
- **full Rubin coverage, strict:** as above, but before the switch a tract-band with any DECam coverage
  uses DECam only, never Rubin (tract-bands with no DECam coverage stay Rubin per position);
- **campaign as implemented:** tract-bands that hold a DP2 template (any DP2 `template_coadd` in that
  tract and band) use Rubin templates only from $t_0$, DP2's and then the model's own; all other
  tract-bands follow the full-Rubin-coverage rule.

```{list-table} Effect of the switching rule (N = 5, no visit cuts; year-1 completeness with DECam)
:header-rows: 1

* - Case
  - Rubin only
  - Observability order, per position
  - Campaign schedule, per position
  - Campaign schedule, any Rubin template
  - Campaign schedule, full Rubin coverage
  - Campaign schedule, full Rubin coverage, strict
  - Campaign schedule, as implemented
* - Completeness, L = 0
  - 0.714
  - 0.803
  - 0.791
  - 0.753
  - 0.793
  - 0.786
  - 0.791
* - Completeness, L = 3 mo
  - 0.577
  - 0.774
  - 0.750
  - 0.701
  - 0.747
  - 0.746
  - 0.745
* - Completeness, L = 12 mo
  - 0.139
  - 0.673
  - 0.648
  - 0.640
  - 0.648
  - 0.647
  - 0.641
* - Completeness, semiannual calendar
  - 0.547
  - 0.763
  - 0.742
  - 0.680
  - 0.738
  - 0.736
  - 0.736
* - G_visit, semiannual calendar
  - –
  - 0.136
  - 0.126
  - 0.072
  - 0.125
  - 0.119
  - 0.122
* - G_visit,4, semiannual calendar
  - –
  - 0.337
  - 0.324
  - 0.127
  - 0.322
  - 0.312
  - 0.312
```

Switching a tract as soon as any Rubin template exists in it removes most of the DECam gain in
all-four-band coverage (61% of it on the semiannual calendar) and costs 1–6 points of year-1 completeness
relative to per-position switching on the same schedule (3–8 points relative to the observability-order
reference). For any $L > 0$, only the 15% of tract-bands that hold DP2 templates have switched at $t_0$;
the loss comes later, as each Rubin template that appears withdraws DECam from its whole tract-band long
before Rubin covers it. At $L = 12$ months the cost is small (0.8 points), because almost no Rubin
templates appear in year 1. Waiting for full Rubin
coverage keeps almost all of the per-position result, and the strict version costs only a further 0.1–0.7
points of completeness and about 1 point of $G_{\rm visit}$ and $G_{\rm visit,4}$. The strict rule leaves visits without any
template only in tract-bands that DECam covers partially: 1,444 tract-bands with 874 deg² of WFD area
(summed over $griz$) lacking a DECam template, mostly at the $|b| = 10^\circ$ edge of the DECam footprint and
in coverage holes; note that no WFD tract reaches the Dec +30° edge.

The campaign's curation as implemented costs 0.2–0.7 points against the full-coverage rule, because DP2
already covers 87% of the WFD area in the tract-bands that hold its templates. The rest, 1,675 deg² summed
over $griz$ (1,315 deg² of it where a DECam template would otherwise be available), has no template at
$t_0$. It shrinks to 900 deg² by +3 months, both on the semiannual calendar and at $L = 3$ months, as
positions ready from pre-survey visits receive their first Rubin templates, and to 274–380 deg² by
+12 months. With DM's visit cuts the pattern is the same (semiannual calendar: 0.464 Rubin only,
0.719 per position, 0.656 any, 0.715 full, 0.712 strict, 0.711 as implemented).

### DECam acceptance and deployment date

```{list-table} Year-1 completeness with DECam against the acceptance minimum (N = 5)
:header-rows: 1

* - Case
  - N_min = 1
  - N_min = 3
  - N_min = 6
* - L = 0
  - 0.803
  - 0.801
  - 0.776
* - L = 3 months
  - 0.774
  - 0.768
  - 0.715
* - Semiannual calendar
  - 0.763
  - 0.755
  - 0.694
```

Requiring three inputs per DECam template costs almost nothing; requiring six costs 3–7 points, most
where DECam carries the most weight. Delaying DECam completion matters too: at $L = 0$, $G_{\rm visit,4}$
falls from 0.187 for completion on 2026 November 15 to 0.126 for 2027 June 1.

```{figure} figures/fig_lat_decam_date.png
:name: fig-lat-decam-date

$G_{\rm visit,4}$ against the DECam deployment completion date ($N = 5$), for latencies of 0, 1 and 3 months.
```

(sec-limitations)=
## Limitations

- **Purity is not modelled.** We count only real transients. Fewer template inputs, PSF variations
  between inputs and discontinuities at patch boundaries all raise the rate of spurious
  difference-image sources. These are the main reasons to choose a larger $N$, stricter visit cuts or a
  non-zero latency, and they are outside this note. DECam templates add their own. In an early COSMOS
  test, Rubin–DECam differencing produced about 20 times as many unfiltered diaSources as Rubin–Rubin
  differencing and about 4 times as many after quality and reliability cuts. The excess was spatially
  clustered and was traced to noise correlated by warping between the two pixel scales; accounting for
  it greatly reduced the excess, and difference images from the ELAIS-S1 pilot templates were judged
  satisfactory {cite:p}`DMTN-362`. No false-positive rate after that correction has been published yet.
  Some extra detections are real but unwanted: stars with proper motion over the roughly ten-year
  baseline leave dipoles, sources with extreme colours leave residuals from bandpass differences, and
  slowly varying sources such as AGN are detected more often {cite:p}`DMTN-362`. Completeness gains
  from DECam may therefore come with a higher alert rate.
- **DECam depths are modelled, not measured.** Measured depths of the first campaign templates will
  test the depth model directly.
- **Simulation alignment with $t_0$.** The simulation kept observing through the months
  lost to the storm; dropping those visits removes 29% of its first-year WFD $griz$ visits (33% in
  $g$), including template-tier visits the real scheduler would take after resumption. Shifting the
  whole simulation to start at $t_0$ lowers $G_{\rm visit,4}$ on the semiannual calendar by about 0.06
  (0.339 to 0.276 in earlier runs that used a DES-first deployment order; our reference value with
  the observability order is 0.337), but misaligns seasons and is not a clean bound. A simulation restarted at $t_0$ from the real
  visit history would remove this approximation.
- **Survey realization.** Two versions of the same baseline that share only about 38% of their early
  visits give period-averaged metrics within 0.001–0.01 of each other, but single-date metrics can
  differ by ~0.05.
- **Year-1 strategy.** The SCOC is revising the year-1 observing strategy, including the footprint. The
  readiness maps, and hence every result here, should be recomputed for the adopted strategy.
- **u and y bands** have no external templates; their gap is set by the survey alone.

(sec-provenance)=
## Provenance and updates

All numbers come from the authors' private analysis repository at commit `2985ed7`; the figures were
regenerated at `a357a50` with identical numbers. The inputs, by
SHA-256 prefix, are the frozen baseline above (`eab6b75d…`), `calendar.yaml` (`01737063…`, which also
holds the campaign chunk schedule), `template_qc.yaml` (`cd2c3090…`), `template_switch.yaml`
(`07a1a60d…`, the expected campaign schedule and tract switching rules) and the DP2 tract-band list
(`2222a41c…`, from the DP2 `template_coadd` datasets, queried 2026 November 19). Rerunning the grid for a new simulation, a new DECam deployment schedule or a decided Rubin template strategy takes about 1.5 hours. Each version of this note will record the inputs it used.

```{list-table} Version history
:header-rows: 1

* - Version
  - Date
  - Changes
* - 0.1
  - 2026-11-01
  - First draft from the latency grid.
* - 0.2
  - 2026-11-05
  - Template visit selection (DP2/DRP cuts) added as a DM-side input; bad-night stress test.
* - 0.3
  - 2026-11-09
  - RTN-133 handle and authors; DECam deployment described as the campaign's chunk sequence, with its effect quantified.
* - 0.4
  - 2026-11-11
  - DMTN-362 and RTN-011 cited; campaign's expected schedule; tract switching rules; DECam false-positive rates in Limitations.
* - 0.5
  - 2026-11-14
  - Proofreading; numbers made consistent with the tables; strict-rule $G_{\rm visit,4}$; bibliography and acknowledgments completed.
* - 0.6
  - 2026-11-19
  - Rubin templates other than DP2's reach AP no earlier than $t_0$ (Model A, $t_0 + L$); Model B effective latency redefined; campaign's DP2 curation added as a switching rule, with its template holes; template availability with DECam (new figure and table); DMTN-362 revision 2: chunk 0 status, exclude list, IR1 timing, causes of the COSMOS false-positive excess.
```

## Acknowledgments

The DELVE Survey gratefully acknowledges support from Fermilab LDRD (L2019.011), the NASA Fermi Guest
Investigator Program Cycle 9 (No. 91201), the National Science Foundation (AST-2108168, AST-2108169,
AST-2307126, AST-2407526), and the NSF-Simonyi Scholars program.

This material is based upon work supported in part by the National Science Foundation through
Cooperative Agreements AST-1258333 and AST-2241526 and Cooperative Support Agreements AST-1202910 and
AST-2211468 managed by the Association of Universities for Research in Astronomy (AURA), and the
Department of Energy under Contract No. DE-AC02-76SF00515 with the SLAC National Accelerator Laboratory
managed by Stanford University. Additional Rubin Observatory funding comes from private donations,
grants to universities, and in-kind support from LSST-DA Institutional Members.

This research draws upon DECam data as distributed by the Astro Data Archive at NSF NOIRLab, including
public archival data from the Dark Energy Survey (DES). NOIRLab is managed by the Association of
Universities for Research in Astronomy (AURA) under a cooperative agreement with the U.S. National
Science Foundation.

This project used data obtained with the Dark Energy Camera (DECam), which was constructed by the Dark
Energy Survey (DES) collaboration. Funding for the DES Projects has been provided by the US Department
of Energy, the US National Science Foundation, the Ministry of Science and Education of Spain, the
Science and Technology Facilities Council of the United Kingdom, the Higher Education Funding Council for
England, the National Center for Supercomputing Applications at the University of Illinois at
Urbana-Champaign, the Kavli Institute for Cosmological Physics at the University of Chicago, Center for
Cosmology and Astro-Particle Physics at the Ohio State University, the Mitchell Institute for Fundamental
Physics and Astronomy at Texas A&M University, Financiadora de Estudos e Projetos, Fundação Carlos Chagas
Filho de Amparo à Pesquisa do Estado do Rio de Janeiro, Conselho Nacional de Desenvolvimento Científico e
Tecnológico and the Ministério da Ciência, Tecnologia e Inovação, the Deutsche Forschungsgemeinschaft and
the Collaborating Institutions in the Dark Energy Survey.

The Collaborating Institutions are Argonne National Laboratory, the University of California at Santa
Cruz, the University of Cambridge, Centro de Investigaciones Enérgeticas, Medioambientales y
Tecnológicas–Madrid, the University of Chicago, University College London, the DES-Brazil Consortium, the
University of Edinburgh, the Eidgenössische Technische Hochschule (ETH) Zürich, Fermi National Accelerator
Laboratory, the University of Illinois at Urbana-Champaign, the Institut de Ciències de l’Espai
(IEEC/CSIC), the Institut de Física d’Altes Energies, Lawrence Berkeley National Laboratory, the
Ludwig-Maximilians Universität München and the associated Excellence Cluster Universe, the University of
Michigan, NSF NOIRLab, the University of Nottingham, the Ohio State University, the OzDES Membership
Consortium, the University of Pennsylvania, the University of Portsmouth, SLAC National Accelerator
Laboratory, Stanford University, the University of Sussex, and Texas A&M University.

Based on observations at NSF Cerro Tololo Inter-American Observatory, a program of NOIRLab (NOIRLab Prop.
IDs 2019A-0305, PI: A. Drlica-Wagner; and 2012B-0001, PI: J. Frieman), which is managed by the
Association of Universities for Research in Astronomy (AURA) under a cooperative agreement with the U.S.
National Science Foundation.

The Legacy Surveys consist of three individual and complementary projects: the Dark Energy Camera Legacy
Survey (DECaLS; Proposal ID #2014B-0404; PIs: David Schlegel and Arjun Dey), the Beijing-Arizona Sky
Survey (BASS; NOAO Prop. ID #2015A-0801; PIs: Zhou Xu and Xiaohui Fan), and the Mayall z-band Legacy
Survey (MzLS; Prop. ID #2016A-0453; PI: Arjun Dey). DECaLS, BASS and MzLS together include data
obtained, respectively, at the NSF Víctor M. Blanco 4-meter Telescope, Cerro Tololo Inter-American
Observatory, NSF NOIRLab; the Bok telescope, Steward Observatory, University of Arizona; and the NSF
Nicholas U. Mayall 4-meter Telescope, Kitt Peak National Observatory, NSF NOIRLab. The Legacy Surveys
project is honored to be permitted to conduct astronomical research on I'oligam Du’ag (Kitt Peak), a
mountain with particular significance to the Tohono O’odham Nation. BASS is a key project of the
Telescope Access Program (TAP), which has been funded by the National Astronomical Observatories of
China, the Chinese Academy of Sciences (the Strategic Priority Research Program “The Emergence of
Cosmological Structures” Grant # XDB09000000), and the Special Fund for Astronomy from the Ministry of
Finance. The BASS is also supported by the External Cooperation Program of Chinese Academy of Sciences
(Grant # 114A11KYSB20160057), and Chinese National Natural Science Foundation (Grant # 11433005). The
Legacy Survey team makes use of data products from the Near-Earth Object Wide-field Infrared Survey
Explorer (NEOWISE), which is a project of the Jet Propulsion Laboratory/California Institute of
Technology. NEOWISE is funded by the National Aeronautics and Space Administration. The Legacy Surveys
imaging of the DESI footprint is supported by the Director, Office of Science, Office of High Energy
Physics of the U.S. Department of Energy under Contract No. DE-AC02-05CH1123, by the National Energy
Research Scientific Computing Center, a DOE Office of Science User Facility under the same contract; and
by the U.S. National Science Foundation, Division of Astronomical Sciences under Contract No. AST-0950945
to NOAO.

This research uses services or data provided by the Astro Data Lab, which is part of the Community
Science and Data Center (CSDC) Program of NSF NOIRLab {cite:p}`Fitzpatrick2014,Nikutta2020`. NOIRLab is
operated by the Association of Universities for Research in Astronomy (AURA), Inc. under a cooperative
agreement with the U.S. National Science Foundation.

Analysis and plotting code were written and run by an AI agent (Claude, Anthropic) under the authors'
direction, and Claude assisted with drafting; the authors checked the results and are responsible for the
content.

## References

```{bibliography}
```
