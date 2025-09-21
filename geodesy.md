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
3. [Building HOPS (optional)](#building-hops-optional)
4. [Basic Correlation](#basic-correlation)
5. [Geodesy Post-processing](#geodesy-post-processing)
6. [Phasecal](#phasecal)

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
mpirun `which sfxc` N24L2_No0005.ctrl n24l2.vix
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

## Phasecal

If you look at the plots generated by fourfit, you'll see that it
isn't entirely happy and reports an `H` error code.  One of the
reasons for this is that we didn't provide any phase cal information.

To enable phasecal extraction in the correlator you will need to add a
`phasecal_file` parameter to the control file.  This parameter should
name the file where the extracted phasecal information should be
written.  Our convention is to give these file a `.pc` extension.

Unfortunately `sfxc2mark4.py` does not support phasecal processing for
mixed-bandwidth correlation yet.  Therefore you need to remove `Cm`
and `De` from the `stations` parameter in the control file.  This
isn't a major issue since the e-MERLIN out-stations don't support
phasecal anyway.  But a fix is being worked on.

Now recorrelate the date, re-run `sfxc2mark4.py` and re-run fourfit.
Do you see any difference?


_Content built by Mark Kettenis._ <i><span id="lastModified"></span></i>

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
