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
6. (optional) [Data inspection using `jiveplot`](#data-inspection-using-jiveplot)
7. (optional) [Export to FITS-IDI](#export-to-fits-idi)

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
     - [plot data from the MS](#data-inspection-using-jiveplot)
- (optional) add calibration tables
- (optional) [export to FITS-IDI](#export-to-fits-idi)

# Setup your environment

In the following steps several tools will be needed. They need to be present on your post-correlation system. On the workshop cluster the compiled binaries will already have been installed for you, as documented below. The Python modules used in this document are not: because of Python3 package management strategies these have to be installed in a virtual environment of your own by yourself - also documented below.

## The `jive-casa` C/C++ compiled tools

The [jive-casa](https://code.jive.eu/verkout/jive-casa.git) tools are absolutely necessary for the post-correlation workflow. If not available on the system, compiling from source is simple enough. However, before going there, test if the tool(s) are already on your system:

```bash
# Check if the tool is already available on your system,
# expect output similar to this.
$> j2ms2 --version
j2ms2: Version 1.0.6 git:master@95009aa
```

If not installed, the tools can be compiled from the following git repositories and following the build instructions therein. They're all CMake-ified projects.
- the [casacore](https://github.com/casacore/casacore) suite of C++ libraries for radio astronomy data processing<br>NOTE: may be installable throuh the system's package manager (`apt-get`, `yum`, ...), YMMV:
```bash
$> sudo {apt-get|yum|...} install casacore-dev
```
- the [myvex](https://code.jive.eu/verkout/myvex.git) VEX-parsing library, a dependency for the next item
- the [jive-casa](https://code.jive.eu/verkout/jive-casa.git) data format translation programs

## The Python based `jiveplot` MS plotting package
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
$> . ${HOME}/venvs/jiveplot/bin/activate
```
Or substitute `jiveplot` with the name of the venv of your choice.

Within the **activated** (`jiveplot`) venv, installing the `jiveplot` Python package should be as easy as:

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
If an error occurs about "not being able to find subband information" - please check the [mixed bandwidth](#mixed-bandwidth-correlation) note below first.

Notes:
- if `EXPERIMENT.ms` already exists, all data specified on the `j2ms2` command line will be **appended** to that MS. Usually this is desirable behaviour, but please see the [notes on `j2ms2`](https://code.jive.eu/verkout/jive-casa/src/branch/master/j2ms2.md#what-j2ms2-does-not-do)
- the name of the MS is rather immaterial, for demonstration purposes it is always `EXPERIMENT.ms` but the `EXPERIMENT` part in this documentation is to be interpreted as _placeholder_.


## Mixed-bandwidth correlation
A special mention needs to go out to "mixed bandwidth" correlation. Many stations in the EVN observe with different channel/IF/spectral window bandwidths. The scheduler takes care that for example one 64 MHz band of station X overlaps with 2 x 32 MHz bands of station Y - otherwise correlation would be impossible.

Because the VEX file is organised _per station_ this means there are different **frequency setups** in the VEX file. E.g. `setup_64MHz` for station X and `setup_32MHzx2` for station Y.

At correlation time this is usually fixed by assigning (or creating) a specific station's frequency setup as "how it's correlated". In the example here: the experiment will be correlated as 2 x 32 MHz bands - i.e. in the `setup_32MHzx1` mode.

`j2ms2` cannot by itself know how the data was correlated in a case like this. Its default behaviour is to take the **first frequency configuration** of the **first** station it finds in the VEX file.

As the [`j2ms2`](https://code.jive.eu/verkout/jive-casa/src/branch/master/j2ms2.md#notes-re-filler-and-experiment-options) documentation explains, the following command line option can be used to instruct `j2ms2` to use station Y's frequency configuration:

```bash
$> j2ms2 eo:setup_ref_station=Y [options] *.cor
```

# Data inspection using `jiveplot`

Before blindly throwing data reduction software at the data, it is recommended to do some data quality assessments. At JIVE the `jiveplot` package is used to create diagnostic plots from the raw data in a MeasurementSet.

For interferometric data a number of "standard plots" can tell "did the correlation actually work?", and summarise the data visually for quick inspection if any issues with the downstream data reduction can be foreseen.

The SFXC correlator produces _complex spectra_ as output by default, and that is what ends up in [the MeasurementSet](#translate-to-measurementset). One _complex spectrum_ per baseline, per source, per subband, per polarisation per integration time. In other words: even a small MS will contain _a lot_ of spectra.

In this section the `jplotter` command line interface (the "jcli") (from the [`jiveplot`](https://github.com/haavee/jiveplot.git) project) will be used to create those 'standard diagnostic plots'. It features _very_ short commands to type in (but they are 'mnemonics', mostly); the focus is on speedy interactive plotting in favour of readability. In this section the "jcli" commands that can be typed at the prompt are rendered **in boldface**.

After [having your environment set up](#setup-your-environment), the "jcli" can be entered:

```python
# for reasons, the program is called `jplotter`, not jiveplot, sorry
# it will drop you into the jplotter command line interface ('jcli')
$> jplotter
+++++++++++++++++++++ Welcome to cli +++++++++++++++++++
$Id: command.py,v 1.16 2015-11-04 13:30:10 jive_cc Exp $
  'exit' exits, 'list' lists, 'help' helps
jcli>
```
Feel free to see what `help` and `list` do. (Hint: `help` without arguments provides an overview of _all_ commands with a one-line summary of what they do, providing some inspiration.)

According to the `jiveplot`'s README.md the 5-second workflow is like this:
```python
# open a m(easurement) s(et)
jcli> ms /path/to/a/file.ms

# or this (featuring TAB-completion)
jcli> cd /path/to/a
jcli> ms file.ms

# (optional) select which data to plot
...

# select a p(lot) t(ype) using "pt <plottype>"
# l(ist) p(lottypes) ("lp") to see what's available
jcli> lp
...
jcli> pt <plottype>

# And ... pl(ot)
jcli> pl
```

In the `jiveplot` repository exists [a the colourful PDF](https://github.com/haavee/jiveplot/blob/master/doc/jplotter-cookbook-draft-v2.pdf) that explains the high-level ideas behind `jiveplot`.
Use the built-in `help <command>` to have `<command>` explained in (too?) much detail, e.g. about 'mini languages' that help easing the data selection.


### Weights
One of the simplest diagnostics to check is checking the _weights_ that the correlator has assigned to each _complex spectrum_. The weight is a floating point number $0 \leq \text{weight} \leq 1$, where $\text{weight} = 0$ implies no valid samples at all went into the resulting spectrum, and $\text{weight} = 1$ meaning _perfect data_ - not a sample was lost computing that spectrum.

```python
# assumes a MeasurementSet was already opened.
# select p(lot) t(ype) w(eight-versus-)t(ime)
jcli> pt wt

# just give it a go; pl(ot) all data and see what happens
jcli> pl
```

Most likely this plots _way_ too much information. It helps realising that the _weight_ on a cross-baseline "XY" is computed from the weights of the individual antennae forming the baseline. Those weights are taken from the antennae's auto-correlation spectra, the '0-baseline' "XX" and "YY". In fact, that extends to the cross-polarization products too: the RL/LR weights are formed by combining the baseline input's individual "X/R", "X/L", "Y/R", "Y/L" polarization weights.

This knowledge, together with `jplotter`'s data set agnostic data selection mechanisms allows plotting only the _relevant_ weights:
```python
# only select the auto baselines
jcli> bl auto

# for each subband/spectral window ('*'), select only the p(arallel) polarizations
# (the 'fq' command is for selecting the 'frequencies', or "channels")
jcli> fq */p

# and regenerate the pl(ot)
jcli> pl
```

### Auto-correlation spectra, a.k.a. "bandpass"
It is very insightful to inspect the amplitude of the complex spectra versus frequency response of the individual antennas. This is also called the "bandpass". It shows (local) RFI signals, polarization- or subband related issues, or e.g. receiver gain fall off when observing near the edge of the receiver's usable frequency range.

For these plots again only the auto baselines are used, but the cross-polarization plots have a good use case here. They'll show e.g. if a polarization is swapped, or if a station is using a linearly polarized receiver whilst others employ circular polarized receivers.

```python
# again assumes a measurement set is opened
# make sure auto baselines are selected
jcli> bl auto

# select p(lot) t(ype) amp(litude-versus-)freq(uency)
pt ampfreq

# select all polarization products (=the default) of all subbands ('*')
jcli> fq *

# and regenerate the pl(ot)
jcli> pl
```

### Phase across the band
If the system(s) are working

### Phase versus time

# Export to FITS-IDI

The `jive-casa` toolbox comes with two programs:
- `j2ms2` for correlator output $\rightarrow$ MeasurementSet format
- `tConvert` for MeasurementSet $\rightarrow$ FITS-IDI format

If the correlated data cannot usefully be processed using [`CASA`](https://casa.nra.edu/), or other tools that operate on MeasurmentSet, or inspected using [`jiveplot`](#data-inspection-using-jiveplot), then maybe exporting to FITS-IDI format is a last resort.

The basic use should be literally as simple as this, provide the input MS name and a desired FITS-IDI output file name:
```bash
$> tConvert <input-ms> <output-fits-file>
```

Unless the `<input-ms>` was created in a bizarre way - e.g. mixing data from different experiments, or from different correlator setups - the process should Just Work&trade;. The key issue to be aware of is that the MeasurementSet format allows much more than what can be represented in FITS-IDI format. `tConvert` does all kinds of checks on `<input-ms>` to verify that in principle the translation can be done.

See the full [`tConvert`](https://code.jive.eu/verkout/jive-casa/tConvert.md) documentation for all intricate details and slightly more advanced use cases that `tConvert` supports.



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
