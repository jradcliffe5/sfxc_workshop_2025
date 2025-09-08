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
# SFXC workshop 2025 • Pulsar processing



## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)
3. [Singularity Images](#singularity-images)
4. [Producing filterbank format output + run PSR tools](#filterbank-psr-tools)
5. [Pulsar Gating](#pulsar-gating)

## Introduction
Pulsar processing is special in the sense that 
- the signal is **dispersed** and that
- the pulsar is "off" most of the time.

Thus, in order to achieve the highest signal-to-noise possible, the data need to be
de-dispersed at the correct dispersion measure and we can apply a technique called
**gating** to only use data when the pulsar is "on".

### Dispersion
- add a nice plot of dispersion
- and an illustration of what dedispersion does
- ideally both with a dynamic spectrum and the frequency-collapsed time series on top
- let's ignore scattering for now.
- include definition of DM, maybe even a fancy movie?

### Pulsar timing
- intro to pulsar timing and how to generate and use an ephemeris file

[//]: # (Only in-line math works on the Github-pages site:)
[//]: # ($\gamma = \lim_{n\to\infty}\left(\sum_{k=1}^n \frac{1}{k} - \ln(n)\right)$)

## Data download
_Add links and instructions for obtaining the relevant datasets here._

## Singularity Images
_Need to find where to upload singularity containers that container pulsar software_

## Producing filterbank format output + run PSR tools
What we'll do in this tutorial is to use SFXC to generate a filterbank file from the
baseband data of one of the stations in our observations. We chose to use the data from
Effelsberg as it is the most sensitive dish in the array.

The aim of this exercise is to a) introduce the pulsar-related capabilities of SFXC and b)
show some basics of pulsar analysis software. 
### Prepare the ctrl file
```yaml
    "number_channels": 128, 
    "cross_polarize": true, 
    "integr_time": 2.048, 
    "sub_integr_time": 128.0,
    "start": "2021y66d20h43m15s", 
    "stop": "2021y66d20h45m09s",
    "output_file": "file:///data1/franz/pr143a/sfxc/pr143a_corr_no0082_b0355_128us_125kHz_FullPol_FullDedisp.cor", 
    "filterbank": true, 
```

## Pulsar Gating
- pic of a pulse profile chopped into gates
### Prepare the ctrl file
```yaml
    "number_channels": 128, 
    "cross_polarize": true, 
    "integr_time": 2.048, 
    "sub_integr_time": 128.0,
    "start": "2021y66d20h43m15s", 
    "stop": "2021y66d20h45m09s",
    "output_file": "file:///data1/franz/pr143a/sfxc/pr143a_corr_no0082_b0355_128us_125kHz_FullPol_FullDedisp.cor", 
    "pulsar_binning": true, 
    "pulsars": {
        "B1933+16_D": {
            "nbins": 4, 
            "polyco_file": "file:///scratch/m/mhvk/viswesh/GP052D/B1957+20/polycob1957+20_gpfit.dat", 
            "no_intra_channel_dedispersion": true, 
            "coherent_dedispersion": false,
            "interval": [
                0.8, 
                0.9
            ], 
        },
    } 
```



## Current & future developments

## Resources
### Technical papers/memos on wide-field correlation
1. Deller, A. T., et al., “DiFX-2: A More Flexible, Efficient, Robust, and Powerful Software Correlator”, *PASP*, 123(901), 275 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011PASP..123..275D/abstract>
2. Morgan, J. S., et al., “VLBI imaging throughout the primary beam using accurate UV shifting”, *A&A*, 526, A140 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011A%26A...526A.140M/abstract>
3. Keimpema, A., et al., “The SFXC software correlator for very long baseline interferometry: algorithms and implementation”, *Experimental Astronomy*, 39(2), 259–279 (2015). doi: <https://ui.adsabs.harvard.edu/abs/2015ExA....39..259K/abstract>

---
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
