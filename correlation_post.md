<!-- MathJax -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script> 
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']],
        displayMath: [['$$','$$']]
      }
    });
</script> 

<script type="text/javascript">
var pcs = document.lastModified.split(" ")[0].split("/");
var date = pcs[1] + '/' + pcs[0] + '/' + pcs[2];
onload = function(){
    document.getElementById("lastModified").innerHTML = "Page last modified on " + date;
}
		</script>

<link href="styles.css" rel="stylesheet" />

<!-- Prism CSS -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism.min.css" />
<link id="prism-dark" rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css" disabled />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/plugins/line-numbers/prism-line-numbers.min.css" />

<!-- Prism JS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>

[Return to the homepage](index.md)
# SFXC workshop 2025 • Post-correlation workflow

With the SFXC installation come some tools that allow inspection of the `.cor` files that the correlator outputs directly.

This, however, is not suitable for proper data quality assessment, and those `.cor` files cannot be distributed to a scientist: none of the viable VLBI Radio Data Processing systems (AIPS, CASA, Miriad, HOPS, ...) can use the `.cor` files directly

This section explains how the workflow at [JIVE](https://jive.eu) addresses these issues.

# On this page
1. [Introduction](#introduction)
2. [Canonical post-correlation workflow](#canonical-post-correlation-workflow)

# Introduction/background
At JIVE it was decided long ago (~1997, that was in the previous millenium) to use the [AIPS++/CASA MeasurementSet v2](https://casacore.github.io/casacore-notes/229.pdf) format ("MS", "MSv2" hereafter) as internal data format.

The reasons for choosing it as "internal" data format were simple:
- no VLBI data reduction supported in the `AIPS++` (former name of `CASA`, it's [complicated](https://code.jive.eu/verkout/jive-casa/commit/d6d6ad3ac7b2af6f21d60f2a859ef47375760699)) data reduction package (DRP) at the time
- but it was a more modern data structure
- `AIPS++` had scripting language that give direct access to the data; none of the other data formats had that (no, [not Python](https://casa.nrao.edu/aips2_docs/glish/glish.html), Python [had it's v1.2 release in 1995](https://en.wikipedia.org/wiki/History_of_Python) - and it _definitely_ didn't come with batteries included back then)
- it would allow the correlator builders $\infty$ freedom of choice how to capture or format the correlator output
- and finally, it would only be a matter of time before the `AIPS++` project would be the VLBI DRP of choice ... (it took until ~2019, a matter of time indeed)

This decision, to use CASA MeasurementSet as intermediate/internal data format has proven to be a very, very, good one. It has allowed JIVE to support multiple correlators, with equally different output data formats, each only having to provide code to decode the data format so it could be written out as MeasurementSet.

All other post-correlation workflow tools, which operate on the MeasurementSet directly, are, and have been, entirely agnostic about which correlator produced the data. Not bad!

As archival data product, which ends up in the [EVN Archive](https://archive.jive.eu/) and gets distributed to the scientists, the well-documented [FITS-IDI](https://fits.gsfc.nasa.gov/registry/fitsidi.html) format ("FITS-IDI") was chosen.

# Canonical post-correlation workflow

The typical post-correlation workflow at JIVE can be summarized as follows:

- [gather correlator output and experiment meta data](#gather-data) (=VEX file)
- translate into MeasurementSet
- run scripts (optional):
     - that fix known issues
     - that flag bad/missing data
     - plot data from the MS
- (optional) add calibration tables
- export to FITS-IDI

## Gather data

The SFXC correlator generates outputs as `<job number>/<exp>_<scan>.cor`, or just `<exp>_<scan>.cor`, e.g. `N24L2_No0001.cor`. These jobs must be collected, together with the VEX-file that was used for the correlation in a folder structure like this:

```text
/path/to/EXPERIMENT/
               ├─ EXPERIMENT.vix
               ├─ <JOBx> 
                     ├─ <EXPERIMENT>_<SCANx>.cor
                     └─ <EXPERIMENT>_<SCANy>.cor
               ├─ <JOBy> 
                     ├─ <EXPERIMENT>_<SCANz>.cor
                     └─ <EXPERIMENT>_<SCANi>.cor
```

Only in-line math works on the Github-pages site:

$\gamma = \lim_{n\to\infty}\left(\sum_{k=1}^n \frac{1}{k} - \ln(n)\right)$

## Project setup
> **Tip**: Use Prism for syntax highlighting and line numbers.

**Include Prism (core, Python, line numbers):**
```html
<!-- Prism core & theme -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism.min.css" />
<link id="prism-dark" rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css" disabled />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/plugins/line-numbers/prism-line-numbers.min.css" />
```

## Correlator preparation
### A1. Calculate wide-field correlation parameters
The multiple phase centre observing mode of the correlator requires two parameters to be set correctly:

- `fft_size_correlation` — number of frequency points $N_\mathrm{FFT}$ (power of 2).
- `sub_integr_time` — sub integration time $t_{\mathrm{int,sub}}$ in microseconds.

Typical constraints include **time and bandwidth smearing** so that the farthest phase centre position is kept below an acceptable level (often **1%** at JIVE).

### A2. Edit the control (ctrl) file
Ensure multi-phase centre correlation is enabled:

```text
multi_phase_center = true
```

### A3. Edit the VEX file
Define each phase centre in the `$SOURCE` section (example):
```text
def RFC1;
  source_name = RFC1;
  ra = 12h26m22.5068s;
  dec = 64d06'22.046";
  ref_coord_frame = J2000;
enddef;
```

Include the new source names in all relevant scans in `$SCHED` (example):
```text
scan No0005;
  start=2022y108d15h50m09s; mode=EFF_BAND_32; source=J1229+6335;source=RFC1;
  station=Ef:    0 sec:  132 sec:      0.000000000 GB:   : &cw   : 1;
  station=O8:    0 sec:  132 sec:      0.000000000 GB:   :       : 1;
  station=Tr:    0 sec:  132 sec:      0.000000000 GB:   : &ccw  : 1;
  station=Mc:    0 sec:  132 sec:      0.000000000 GB:   : &ccw  : 1;
  station=Nt:    0 sec:  132 sec:      0.000000000 GB:   : &ccw  : 1;
endscan;
```

### Example Python block (from the tutorial)
```python
from __future__ import annotations
from dataclasses import dataclass

@dataclass
class Greeter:
    prefix: str = "Hello"

    def greet(self, name: str) -> str:
        return f"{self.prefix}, {name}!"

if __name__ == "__main__":
    g = Greeter()
    for who in ("Alice", "Bob", "Charlie"):
        print(g.greet(who))
```

## Running the correlator
### B1. Execute the correlator
_Add concrete run commands and environment details here._

## Current & future developments

## Resources
### Technical papers/memos on wide-field correlation
1. Deller, A. T., et al., “DiFX-2: A More Flexible, Efficient, Robust, and Powerful Software Correlator”, *PASP*, 123(901), 275 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011PASP..123..275D/abstract>
2. Morgan, J. S., et al., “VLBI imaging throughout the primary beam using accurate UV shifting”, *A&A*, 526, A140 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011A%26A...526A.140M/abstract>
3. Keimpema, A., et al., “The SFXC software correlator for very long baseline interferometry: algorithms and implementation”, *Experimental Astronomy*, 39(2), 259–279 (2015). doi: <https://ui.adsabs.harvard.edu/abs/2015ExA....39..259K/abstract>

_Content built by XX._ <i><span id="lastModified"></span></i>

_Built with ♥ — Markdown + HTML + CSS + Prism.js + a bit of AI + Jack Radcliffe (2025)_

<!-- Custom Script: funcs.js -->
<script>
    const copy = (el) => {
      const pre = document.querySelector(el);
      if (!pre) return;
      const code = pre.innerText;
      navigator.clipboard.writeText(code).then(() => {
        const btn = document.querySelector(`[data-copy="${el}"]`);
        if (!btn) return;
        const old = btn.textContent;
        btn.textContent = 'Copied!';
        setTimeout(() => (btn.textContent = old), 1500);
      });
    };
    document.addEventListener('click', (e) => {
      const t = e.target;
      if (t.matches('.copy-btn')) {
        const target = t.getAttribute('data-copy');
        copy(target);
      }
    });
</script>
