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
# SFXC workshop 2025 • Geodesy



## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)

## Introduction

SFXC can be used to produce correlated data sets in Mark4 format for
use with the HOPS post-processing package.  After post-processing the
results can be used for geodesy and absolute astrometry.

## Data download

This tutorial uses the same N24L2 dataset as some of the previous tutorials.
If you did do so already, you can download this dataset from:

<https://archive.jive.nl/sfxc-workshop/n24l2>

On the workshop cluster the data is available available in the
`/data/n24l2` directory.

For this tutorial you will need the HOPS software.  This tutorial has
been tested using HOPS 3.26, but earlier versions should work as well
If this software is not installed already on your system you can
download the HOPS source code from:

<https://web.mit.edu/haystack-www/hops/hops-3.26-4329.tar.gz>

On the workshop cluster you can find the HOPS source code in the
`/data/src` directory.  A pre-compiled version is available in
`/home/kettenis/hops/x86_64-3.26`.

## Building HOPS (optional)

Instructions for building HOPS are provided in the README.txt file
that comes with the source code.  Installing shouldn't be difficult;
the hardest part is probably setting up its dependencies, in
particular the PGPLOT package.  The necessary dependencies have
already been installed on the workshop cluster.

```text
mkdir hops
cd hops
tar xfz /data/src/hops-3.26-4329.tar.gz
mkdir bld-3.26
cd bld-3.26
../hops-3.26/configure
make install
```

## Basic Correlation

In order to create Mark4 data products we'll need to have some
correlated data.  You can re-use some of the data that you have
correlated earlier in the workshop.  But if you want to start fresh,
here is an example control file that will work:

```text
{
    "cross_polarize": true,
    "number_channels": 32,
    "data_sources": {
        "De": [
            "file:///data/n24l2/files/n24l2_de_no0005.vdif"
        ],
        "Cm": [
            "file:///data/n24l2/files/n24l2_cm_no0005.vdif"
        ],
        "Ef": [
            "file:///data/n24l2/files/n24l2_ef_no0005"
        ],
        "Hh": [
            "file:///data/n24l2/files/n24l2_hh_no0005"
        ]
    },
    "stations": [
        "Cm",
        "De",
        "Ef",
        "Hh"
    ],
    "start": "2024y144d12h47m00s",
    "stop": "2024y144d12h47m20s",
    "channels": [
        "CH01",
        "CH02",
        "CH03",
        "CH04",
        "CH05",
        "CH06",
        "CH07",
        "CH08"
    ],
    "setup_station": "Ef",
    "exper_name": "N24L2",
    "output_file": "file:///home/workshop99/n24l2-geo/N24L2_No0005.cor",
    "integr_time": 2.0,
    "slices_per_integration": 4,
    "delay_directory": "file:///home/workshop99/n24l2-geo/delays",
    "scans": [
        "No0005"
    ]
}
```

Put this in a file called `N24L2_No0005.ctrl` and make sure you adjust
the `output_file` and `delay_directory` parameters to point to
appropriate locations.  You can use the `n24l2.vix` VEX file that
comes with the data for this correlation.  Here is a quick reminder on
how to correlate:

```text
# Set PATH and CALC_DIR
# Copy n24l2.vix to your working directory
gen_all_delay_files.py n24l2.vix N24L2_No0005.ctrl
salloc -n16
mpirun sfxc N24L2_No0005.ctrl n24l2.vix
exit
```

## Geodesy post-processing

To convert the correlator output into Mark4 format you will need to
use the `sfxck2mark4.py` tool.  This tool isn't automatically
installed, but you can run it straight from the source directory.
Assuming you checked out the SFXC source code in `~/src/sfxc` you can
convert the data above by:

```text
~/src/sfxc/sfxc2mark4/sfxc2mark4.py n24l2.vix N24L2_No0005.ctrl
```

This will write output in the `1234` subdirectory.  Take a look at the
contents of that directory.  Can you guess what the cryptic file names
mean?

Now we can use the fourfit tools from HOPS to do some fringe-fitting
and produce the famous plots.  First we have to activate the HOPS
environment:

```text
. ~/hops/x86_64-3.26/bin/hops.bash
```

This should produce a message like:

```text
Setup HOPS v3.26 with HOPS_ROOT=/home/workshop25/hops for x86_64-3.26
```

On the workshop cluster this also produces a bash error that seems to
be harmless.

You can now run `fourfit` to produce plots in test mode, using the
`-t` option, to inspect the data.  If you are on a machine with X (or
have set up X forwarding), you can use the `-x` option to get output
on your screen:

```text
fourfit -t -x 1234
```

Alternatively you can create a PDF file with the plots by using the
`-d ps2pdf` option:

```text
fourfit -t -d ps2pdf:n24l2 1234
```




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
