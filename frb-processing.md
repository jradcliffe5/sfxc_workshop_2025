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
# SFXC workshop 2025 • FRB Processing



## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)
3. [Singularity Images](#singularity-images)
4. [PRECISE processing](#precise-processing)
5. [Semi-automated pulse processing](#semi-automated-pulse-processing)
6. [Single dish FRB workflow](#single-dish-processing)

## Introduction
FRB processing is almost like pulsar processing -- but you only have a single pulse to
work with.

## Data download
_Add links and instructions for obtaining the relevant datasets here._

## Singularity Images
_Need to find where to upload singularity containers that container pulsar software_

## PRECISE processing
PRECISE stands for "Pinpointing REpeating ChIme Sources with the Evn". 

### Observing mode
- we are an EVN-Lite project
- stations join on a best-effort basis
- weekly runs of 10 hours each -- times defined by Eff availability
- ...

### Data processing
- after an observation, the data from Eff are transmitted to a dedicated processing server
  at OSO
- the data are processed with `jive5ab`, `digifil` (part of `DSPSR`), `Heimdall` and
  `FETCH`.
  - `jive5ab` unpacks the VDIF data and splits them into individual files per IF that contain
    both pols 
  - `digifil` converts the split VDIF files into 'detected' filterbanks; i.e. Stokes I
    (can also create full polarisation filterbanks with I, Q, U, V)
  - `Heimdall` runs on a GPU and searches the filterbanks for bursts
  - `FETCH` is an ML-classifier that 'inspects' the candidates found by Heimdall and
    classifies them as either terrestrial or celestial
- supposedly real candidates are sent to Dropbox and links to files are sent to a Slack
  channel where they can be inspected further
- in case of a detection we contact the EVN/JIVE and request correlation

### Correlation procedure
- what are our separate correlation passes

### Single pulse analysis
- maybe talk about some fine structure, coherent dedispersion and such? Bolometer branch?


## Semi-automated pulse processing

## Single dish FRB workflow

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
