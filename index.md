# External templates and the early-LSST template gap: a latency-parametrized forecast

```{abstract}
Difference imaging needs a template at every position and in every band. Rubin builds its templates
from its own visits, so early in the survey many positions cannot yet be differenced, and how long
that lasts depends on two things: how quickly the survey accumulates visits (set by the observing
strategy), and how quickly those visits become templates in Alert Production (set by Data
Management's template strategy). This note quantifies the gap and the value of external DECam
templates without assuming any particular Rubin template strategy. We describe Rubin template
availability by two parameters, the number of visits a template needs ($N$) and the delay from having
$N$ visits to having a template in AP (the latency $L$), and we report alert completeness and
differenceable-visit fractions over a grid of both. The main result is that DECam templates make
year-1 alert completeness nearly insensitive to both parameters: across $L = 0$–12 months and
$N = 4$–10, completeness with DECam stays within 0.71–0.81, whereas without DECam it ranges from 0.75
down to 0.21. Applying DM's DP2 template-visit selection (a 1.4″ PSF ceiling and a transparency
cut) lowers Rubin-only completeness by 0.03–0.10 but changes it by at most 0.03 with DECam. DECam
therefore lets the Rubin template strategy be chosen for purity and operational
convenience at small cost in completeness. This is a living document and will be updated as the
template strategy and the year-1 observing strategy are settled.
```

## Add content here

See the [Documenteer documentation](https://documenteer.lsst.io/technotes/index.html) for tips on how to write and configure your new technote.
