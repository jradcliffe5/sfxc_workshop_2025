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
# SFXC workshop 2025 • Template



## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)

## Introduction
Wide-field VLBI is a specialised correlation mode that …

**Folder structure**
```text
tutorial/
├─ index.html
└─ (images, assets, etc.)
```

Only in-line math works on the Github-pages site:

$\gamma = \lim_{n\to\infty}\left(\sum_{k=1}^n \frac{1}{k} - \ln(n)\right)$

## Data download
_Add links and instructions for obtaining the relevant datasets here._

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
