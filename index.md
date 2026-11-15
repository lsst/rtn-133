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
$N = 4$–10, completeness with DECam stays within 0.71–0.81, whereas without DECam it ranges from 0.75
down to 0.21. Applying DM's DP2 template-visit selection (a 1.4″ PSF ceiling and a transparency
cut) lowers Rubin-only completeness by 0.03–0.10 but changes it by at most 0.03 with DECam. The one
practical DM-side choice that matters is when a tract switches from DECam to Rubin templates: switching as soon
as any Rubin template exists in a tract removes most of DECam's gain in all-four-band coverage and
about a third of its year-1 completeness gain, while waiting for full Rubin coverage keeps nearly all
of both. DECam
therefore lets the Rubin template strategy be chosen for purity and operational
convenience at small cost in completeness. This is a living document and will be updated as the
template strategy and the year-1 observing strategy are settled.
```

:::{note}
**Draft, not for circulation.** Numbers are provisional and come from the runs listed in
{ref}`sec-provenance`. RTN-133 (<https://rtn-133.lsst.io>). Authors: Brian Stalder, Sean Patrick MacBride.
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

DP2 templates are in AP at $t_0$ where DP2 built them.

### DECam templates

DECam template inputs are taken from the 203,399 DELVE DR3 exposures in $griz$ {cite:p}`Tan2025,Anbajagane2025`
(and Drlica-Wagner et al., in prep.), already cut to PSF FWHM ≤ 1.5″ and $t_{\rm eff} \ge 0.2$. We
model the campaign's selection as the best-seeing six exposures per `lsst_cells_v2` patch and band.
DELVE flags or excludes exposures and CCDs (<https://github.com/delve-survey/exclude_list>) mainly when
they are contaminated by scattered and reflected light from bright stars, identified with the DES
ray-tracing procedure {cite:p}`Kent2020`, or when the sky-background estimate failed
{cite:p}`DrlicaWagner2022`; the input list keeps them, and removing them changes nothing material.

Template depths follow the effective-exposure-time formalism of {cite:t}`Neilsen2016`, with one offset
per band inside and outside the DES footprint. In $g$, $r$ and $i$ the offsets match the median
point-source depths of {cite:t}`Tan2025` (S/N = 10, converted to S/N = 5). No footprint-wide $z$
median is published for DELVE DR3; we calibrate $z$ at SZ-cluster positions to DES DR2 {cite:p}`DESDR2` and Legacy
Surveys {cite:p}`Dey2019` depths. The DECADE depth {cite:p}`Anbajagane2025` suggests our $z$ depths outside DES are about
0.1 mag conservative; we carry ±0.15 mag as the $z$ systematic and ±0.1 mag in $r$. These depths have
not yet been validated against built DECam templates.

**Deployment.** The campaign processes and deploys DECam templates in 30° × 30° regions of sky
("chunks"), in full-declination columns (Dec +30° to −90°) advancing in RA from the
RA 300°–330° column {cite:p}`DMTN-362`. Chunk 0 (RA 300°–330°, Dec 0° to +30°) was deployed at USDF on 2026 November 10.
The campaign expects about a week for the rest of that column and then about 10 days per full column going forward, which
would complete the footprint on about 2027 March 7. Unless stated otherwise, the results below model
the deployment as staged in observability order from 2026 November 15 to 2027 January 1. With the same
completion date, the campaign's chunk order changes year-1 completeness by at most 0.006 and
$G_{\rm visit}$ by 0.001; the campaign's expected schedule, finishing two months later, costs 1–2 points
of year-1 completeness (see {ref}`sec-switching`). When the footprint is complete matters; the
order does not. For comparison, deploying the whole DECam footprint at $t_0$
would raise year-1 completeness with DECam on the semiannual calendar from 0.763 to 0.800. DECam templates are used band by band wherever no
Rubin template is in AP, except where we model the campaign's per-tract switching rules
({ref}`sec-switching`). The minimum number of inputs for a DECam template to be accepted, $N_{\min}$,
is a parameter (default 1).

## Method

### Readiness and template availability

For each WFD position $p$ (HEALPix {cite:p}`Gorski2005` $N_{\rm side} = 256$) and band $b$, the readiness time
$t_{\rm ready}(p, b, N)$ is the time of the $N$-th visit, counting pre-survey visits and simulated
visits from $t_0$. Two models turn readiness into a template in AP:

- **Model A, continuous incremental builds.** A template enters AP at $t_{\rm ready} + L$. At time $t$
  the template is the coadd of all visits up to $t - L$, so it keeps deepening.
- **Model B, periodic builds.** Data cutoffs every $P$ months from $t_0$, each deployed $D$ months
  later. A position gets a template at the first deployment whose cutoff follows its readiness; the
  template is the coadd up to that cutoff. This model's effective latency is about $P/2 + D$.

Placed at their effective latency, Model B points fall on the Model A curves for the differenceable
fractions, so we present results against $L$ and mark Model B points on the same axis.
For alert completeness without DECam, periodic builds sit up to about 0.05 below continuous builds at
the same effective latency, because their templates stay frozen, and so shallower, between builds. As a
reference case throughout we use the **semiannual calendar** (data cutoffs every six months, each deployed
three months later), which behaves like $L \approx 6$–7 months.

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
first survey nights failed the transparency cut at 8% in $griz$, almost all in one half-night; we use
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
ready before $t_0$ from pre-survey visits. Across five years, 2.3% of WFD $griz$ visits are taken
before their position is ready at $N = 5$, rising to 17.6% in $u$, for which DECam provides no
templates.

```{figure} figures/fig_floor_readiness.png
:name: fig-readiness

Readiness for $N = 4$, 5, 6 and 10, counting pre-survey and simulated visits. Top: months after the survey start until every one of $griz$ has its $N$-th visit, per WFD position (grey: ready at the start; red: not within 24 months). Bottom: cumulative fraction of the WFD footprint ready against time, per band and for all of $griz$ (dashed).
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
  - 0.067
  - 0.232
* - 3 months
  - 0.105
  - 0.304
* - 6 months
  - 0.127
  - 0.335
* - 12 months
  - 0.266
  - 0.517
* - Semiannual calendar (L ≈ 6–7 mo)
  - 0.136
  - 0.337
```

As {numref}`fig-lat-gain` shows, even with zero latency DECam adds 0.18 to the all-four-band fraction: this is set by the survey speed and strategy, not by DM. The gain then grows with latency.

```{figure} figures/fig_lat_gain.png
:name: fig-lat-gain

Gain from DECam templates in differenceable visits over years 1–2, $G_{\rm visit}$ (left) and $G_{\rm visit,4}$ (right), against Rubin template latency $L$ (Model A) for $N = 4$, 5 and 6. Model B points are placed at their effective latency; the star marks the semiannual calendar.
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
  - 0.68 / 0.63 / 0.57 / 0.38
  - 0.81 / 0.79 / 0.78 / 0.75
* - 12 months
  - 0.43 / 0.37 / 0.32 / 0.21
  - 0.74 / 0.73 / 0.72 / 0.71
```

Without DECam, completeness falls steeply with both latency and $N$. With DECam the surface is nearly
flat: moving from $N = 4$ to $N = 6$ costs 7 points at $L = 0$ without DECam and about 1 point with it.
These numbers describe completeness for real transients only; the reasons to prefer a larger $N$
(artifact rejection, PSF homogeneity, purity) are not modelled here (see {ref}`sec-limitations`). The
result is best read as the completeness cost of each choice of $N$ and $L$, which DECam keeps small.

```{figure} figures/fig_lat_completeness.png
:name: fig-lat-completeness

Year-1 alert completeness (flat mean over the transient grid) against latency $L$ (Model A), Rubin templates only (left) and Rubin plus DECam (right), for $N = 4$, 5, 6 and 10. Model B points are placed at their effective latency; the star marks the semiannual calendar ($N = 5$).
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
  - 0.049 / 0.112 / 0.011
  - 0.016 / 0.056 / 0.003
* - L = 3 months
  - 0.076 / 0.146 / 0.003
  - 0.018 / 0.066 / 0.000
* - L = 6 months
  - 0.112 / 0.158 / 0.002
  - 0.025 / 0.073 / 0.000
* - L = 12 months
  - 0.318 / 0.191 / 0.000
  - 0.044 / 0.091 / 0.000
* - Semiannual calendar
  - 0.138 / 0.159 / 0.001
  - 0.029 / 0.075 / 0.000
```

For latencies up to about six months, templates that exist but are too shallow cost more than missing
templates. Self-contamination matters only for latencies of a month or less, so instantaneous builds
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
  - 0.714 / 0.625 / 0.547
  - 0.677 / 0.549 / 0.464
  - 0.651 / 0.531 / 0.455
* - Completeness, Rubin + DECam: L = 0 / 3 mo / calendar
  - 0.803 / 0.793 / 0.763
  - 0.792 / 0.772 / 0.741
  - 0.790 / 0.766 / 0.740
```

The cuts remove most of the pre-survey head start: only 970 deg² of the WFD footprint has five
qualifying pre-survey visits in all four bands, against 2,734 deg² without cuts. Readiness slows most in
$g$ and $r$, the bands with the worst seeing. Without DECam, year-1 completeness drops by 0.03–0.10,
more at longer latency and larger $N$; with DECam it drops by at most 0.03, and DECam's own gain grows: on the
semiannual calendar $G_{\rm visit}$ rises from 0.136 to 0.176 (about 30%) and $G_{\rm visit,4}$ from 0.337
to 0.372. The cuts do not change the effective latency of the
semiannual calendar (6.8 months). We applied a stress test that fails whole simulated nights at the per-band
transparency fail rates of the first survey nights (25% in $g$, 29% in $r$, under 1% in $i$ and $z$;
three random seeds; failing half-nights instead gives similar results). Those rates come from three nights dominated by one bad
half-night, so the column is an upper bound on weather losses, not an estimate. It mainly moves
$G_{\rm visit,4}$; completeness changes by at most 0.03 without DECam and 0.006 with it.

(sec-switching)=
### Switching tracts from DECam to Rubin templates

The campaign uses one template style per tract and band, never mixing Rubin and DECam templates within a
tract {cite:p}`DMTN-362`. The switch is made per tract, but the condition that triggers it is not yet
fixed, and it turns out to be the DM-side choice that matters most. We compare four rules, all with the campaign's expected
deployment schedule (chunk 0 on 2026 November 10, complete about 2027 March 7). For reference, the table
also gives per-position substitution with the observability-order deployment used elsewhere. The rules are:

- **per position:** each position uses its own Rubin template where one exists and DECam otherwise
  (the substitution used elsewhere in this note);
- **any Rubin template:** a tract-band switches to Rubin as soon as any position in it has a Rubin
  template, and its DECam templates are withdrawn;
- **full Rubin coverage:** a tract-band stays on DECam until Rubin templates cover at least 95% of its WFD
  positions, then switches entirely to Rubin; before the switch, positions without a DECam template use
  their Rubin template if they have one;
- **full Rubin coverage, strict:** as above, but before the switch a tract-band with any DECam coverage
  uses DECam only, never Rubin (tract-bands with no DECam coverage stay Rubin per position).

```{list-table} Effect of the switching rule (N = 5, no visit cuts; year-1 completeness with DECam)
:header-rows: 1

* - Case
  - Rubin only
  - Observability order, per position
  - Campaign schedule, per position
  - Campaign schedule, any Rubin template
  - Campaign schedule, full Rubin coverage
  - Campaign schedule, full Rubin coverage, strict
* - Completeness, L = 0
  - 0.714
  - 0.803
  - 0.791
  - 0.753
  - 0.793
  - 0.786
* - Completeness, L = 3 mo
  - 0.625
  - 0.794
  - 0.778
  - 0.724
  - 0.775
  - 0.767
* - Completeness, L = 12 mo
  - 0.373
  - 0.729
  - 0.706
  - 0.664
  - 0.704
  - 0.701
* - Completeness, semiannual calendar
  - 0.547
  - 0.763
  - 0.742
  - 0.680
  - 0.738
  - 0.736
* - G_visit, semiannual calendar
  - –
  - 0.136
  - 0.126
  - 0.072
  - 0.125
  - 0.119
* - G_visit,4, semiannual calendar
  - –
  - 0.337
  - 0.324
  - 0.127
  - 0.322
  - 0.312
```

Switching a tract as soon as any Rubin template exists in it removes most of the DECam gain in
all-four-band coverage and costs 4–6 points of year-1 completeness relative to per-position switching
on the same schedule (5–8 points relative to the observability-order reference), because 59% of tract-bands already
hold some Rubin template at the survey start, from DP2 and pre-survey visits. Waiting for full Rubin
coverage keeps almost all of the per-position result, and the strict version costs only a further 0.2–0.8
points of completeness and about 1 point of $G_{\rm visit}$ and $G_{\rm visit,4}$. The strict rule leaves visits without any
template only in tract-bands that DECam covers partially: 1,444 tract-bands with 875 deg² of WFD area
(summed over $griz$) lacking a DECam template, mostly at the $|b| = 10^\circ$ edge of the DECam footprint and
in coverage holes; note that no WFD tract reaches the Dec +30° edge. With DM's visit cuts the pattern is the same
(semiannual calendar: 0.464 Rubin only, 0.719 per position, 0.656 any, 0.715 full, 0.712 strict).

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
  - 0.793
  - 0.788
  - 0.740
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
  non-zero latency, and they are outside this note. DECam templates add their own: in an early COSMOS
  test, Rubin–DECam differencing produced about 20 times as many unfiltered diaSources as Rubin–Rubin
  differencing and about 4 times as many after quality and reliability cuts, from proper motion between
  epochs, poorer template seeing, bandpass differences and underestimated difference-image variance
  {cite:p}`DMTN-362`. Completeness gains from DECam therefore come with a higher false-positive rate.
- **DECam depths are modelled, not measured.** Measured depths of the first campaign templates will
  test the depth model directly.
- **Simulation alignment with the delayed start.** The simulation kept observing through the months
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

All numbers come from the authors' private analysis repository at commit `7437883`, using the frozen baseline
above, `calendar.yaml` at `d88188d`, the latency-grid script (`f9aa928`, `7180687`) and the template-QC
option (`template_qc.yaml`, `7437883`), the campaign chunk schedule (`27255bb`), and the expected campaign schedule and tract switching rules
(`template_switch.yaml`, `c063b12`, `06dcdd0`). Rerunning the grid for a new simulation, a new DECam deployment schedule or a decided Rubin template strategy takes about 1.5 hours. Each version of this note will record the inputs it used.

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
