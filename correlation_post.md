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
3. [Setting up your environment](#setup-your-environment)
4. [Gathering data](#gather-data)
5. [Translate into MeasurementSet](#translate-to-measurementset)
6. (optional) [Data inspection using `jiveplot`]()
7. [Export to FITS-IDI]

# Introduction/background
At JIVE it was decided long ago (~1997, that was in the previous millenium) to use the [AIPS++/CASA MeasurementSet v2](https://casacore.github.io/casacore-notes/229.pdf) format ("MS", "MSv2" hereafter) as internal data format.

The reasons for choosing it as "internal" data format were simple:
- no VLBI data reduction supported in the `AIPS++` (former name of `CASA`, it's [complicated](https://code.jive.eu/verkout/jive-casa/commit/d6d6ad3ac7b2af6f21d60f2a859ef47375760699)) data reduction package (DRP) at the time
- but it was a more modern data structure
- `AIPS++` had scripting language that give direct access to the data; none of the other data formats had that (no, [not Python](https://casa.nrao.edu/aips2_docs/glish/glish.html), Python [had it's v1.2 release in 1995](https://en.wikipedia.org/wiki/History_of_Python) - and it _definitely_ didn't come with batteries included back then)
- it would allow the correlator builders $\infty$ freedom of choice how to capture or format the correlator output
- and finally, it would only be a matter of time before the `AIPS++` project would be the VLBI DRP of choice ... (it took until ~2019, a matter of time indeed)

This decision, to use MeasurementSet as intermediate/internal data format has proven to be a very, very, good one. It has allowed JIVE to support multiple correlators, with equally different output data formats, each only having to provide code to decode the data format so it could be written out as MeasurementSet.

All other post-correlation workflow tools, which operate on the MeasurementSet directly, are, and have been, entirely agnostic about which correlator produced the data. Not bad!

As archival data product, which ends up in the [EVN Archive](https://archive.jive.eu/) and gets distributed to the scientists, the well-documented [FITS-IDI](https://fits.gsfc.nasa.gov/registry/fitsidi.html) format ("FITS-IDI") was chosen.

# Canonical post-correlation workflow

The typical post-correlation workflow at JIVE can be summarized as follows:

- [setup your environment](#setup-your-environment)
- [gather correlator output and experiment meta data](#gather-data) (=VEX file)
- [translate into MeasurementSet](#translate-to-measurementset)
- run scripts (optional):
     - that fix known issues
     - that flag bad/missing data
     - plot data from the MS
- (optional) add calibration tables
- export to FITS-IDI

# Setup your environment

In the following steps several tools will be needed. They need to be present on your post-correlation system. On the workshop cluster the compiled binaries will already have been installed for you, as documented below. The Python modules used in this document are not: because of Python3 package management strategies these have to be installed in a virtual environment of your own by yourself - also documented below.

## The C/C++ compiled tools

The necessary tools can be compiled from the following git repositories and following the build instructions therein:
- the [myvex](https://code.jive.eu/verkout/myvex.git) VEX-parsing library (a dependency for the next item)
- the [jive-casa](https://code.jive.eu/verkout/jive-casa.git) data format translation programs

On the workshop cluster the tools are installed under `/data/verkout/tools/bin`, i.e. to make them easily usable do this in your shell:

```bash
$> export PATH=/data/verkout/tools/bin:${PATH}
$> hash -r

# verify they work; expect output as below
$> j2ms2 --version
j2ms2: Version 1.0.6 git:master@95009aa

```
and you're good to go.

Remember to do this in each new shell, or, add the `/data/verkout/tools/bin` to your `${PATH}` variable in your `~/.profile` or `~/.bashrc` exactly as in the snippet above (the `hash -r` is not needed in that case).

## The python MS plotting package
JIVE software engineers are much like software engineers elsewhere: if something doesn't work, or is too slow, or is cumbersome - let's write something different ourselves!

Visualizing data from a MeasurementSet has always been painful, and in the beginning non-existant even. Based upon the in-house developed [not python](https://casa.nrao.edu/aips2_docs/glish/glish.html) application  [jivegui-ms2](https://code.jive.eu/verkout/jivegui-ms2.git), eventually, when Python _did_ become the scripting language of `CASA`, the [`jiveplot`](https://github.com/haavee/jiveplot.git) package got developed.

Because of Python3's externally managed package installation (e.g. through `apt`, `yum` or what have you) the `jiveplot` package needs to be installed in a [virtual environment](https://docs.python.org/3/library/venv.html) ("venv"). Fortunately, these days that's reasonably simple:

```bash
# Create a directory where multiple virtual environments can be created
$> mkdir ${HOME}/venvs
$> cd ${HOME}/venvs

# Each "venv" is identified by a name,
# choose something descriptive, e.g. 'jiveplot'
$> python3 -m venv jiveplot
```

Now the "venv" is created. But a "venv" must be **activated** before your system actually **uses** the "venv".

The command to activate (or switch to) a specific "venv" in the current shell is as follows:

```bash
# Note the leading '. ' (dot and space)
$> . ${HOME}/venvs/<venv-name>/bin/activate
```
Substitute e.g. `jiveplot` to activate the venv that was just created.

Within the activated `jiveplot` venv, installing the `jiveplot` Python package should be as easy as:

```bash
$> pip3 install jiveplot
```

The package is published on the Python Package Index (PyPI) [here](https://pypi.org/project/jiveplot/)


# Gather data

The SFXC correlator control file determines where the correlator generates outputs and how to name the output file(s). For the post-correlation workflow it is important that the file names end in `.cor`.

It is recommended to, if not already done through the automated tooling, to organise the experiment folder and output and VEX file as indicated here:

```text
/path/to/EXPERIMENT/
               ├─ EXPERIMENT.vix
               ├─ <EXPERIMENT>_<SCANx>.cor
               └─ <EXPERIMENT>_<SCANy>.cor
```

At JIVE, where an experiment is correlated in multiple independent jobs, the experiment folder is laid out like this:
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

If the VEX-file isn't named like the directory it is in, a simple [symbolic link](https://www.lenovo.com/us/en/glossary/symbolic-link/) will fix that readily:

```bash
$> ln -s some_file_name.vix EXPERIMENT.vix
```


# Translate to MeasurementSet

Assuming the data is gathered in an EXPERIMENT-specific folder as described under [gather data](#gather-data), and your [environment is set up](#setup-your-environment) the conversion to MeasurementSet is done using the [`j2ms2` (jay to em es too)](https://code.jive.eu/verkout/jive-casa/j2ms2.md) tool.

Depending on how the data is organised in the EXPERIMENT directory, translating SFXC Correlator data to a MeasurementSet called "EXPERIMENT.ms" is as simple as:

```bash
# All `.cor` files in EXPERIMENT directory
# (Or be explicit in exactly which one(s) to translate)
$> j2ms2 -o EXPERIMENT.ms *.cor

# ... or, when having subjobs
# (Here it does all correlator data from all subjobs,
#  but it is possible to be explicit by naming the `.cor`
#  files to be translated individually)
$> j2ms2 -o EXPERIMENT.ms */*.cor
```

This will append the specifed `.cor`'s data to `EXPERIMENT.ms`, creating it if it doesn't exist.

Notes:
- if `EXPERIMENT.ms` already exists, all data specified on the `j2ms2` command line will be **appended** to that MS. Usually this is desirable behaviour, but please see the [notes on `j2ms2`](https://code.jive.eu/verkout/jive-casa/src/branch/master/j2ms2.md#what-j2ms2-does-not-do)
- the name of the MS is rather immaterial, for demonstration purposes it is always `EXPERIMENT.ms` but the `EXPERIMENT` part in this documentation is to be interpreted as _placeholder_.





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
