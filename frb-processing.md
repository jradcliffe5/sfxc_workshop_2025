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
2. [PRECISE processing](#precise-processing)

## Introduction
FRBs -- Fast Radio Bursts -- are millisecond-duration, extremely luminous flashes that
have as of yet only been found at radio frequencies ($\sim100\,\mathrm{MHz}- \sim
9\,\mathrm{GHz}$). There exist two classes: FRBs that burst repeated and those that have
only ever been seen once. The exact emission mechanisms or the progenitor source are as of yet
unknown, as well as it is unclear if repeaters and apparent non-repeaters share a common
origin. For a recent review check out [**Petroff et al. 2021??**](#ref_petroff??) and also
take a look [**Franz' presentation**](link to indico I guess).

One aspect that can help us understand FRBs better is the region they are generated
in. To that end we don't only need to know their host galaxy but also the characteristics
of their local environment -- e.g. do they reside in a star forming or not; can they be
assiciated with a persistent radio radio source or not? VLBI is the only technique that
provides sufficient angular resolution to pinpoint FRBs to a location within a host
galaxy. In conjunction with high resolution optical observations, the emission region can
be characterised.

FRB processing is almost like [pulsar processing](pulsar-processing.md) -- but you only
have a single pulse to work with at a time. Thus, in terms of SFXC-processing it's a matter of
coherently dedispersing the data and defining an appropriate on-gate. Check out Aard's
presentation on Semi-automated pulse processing for more details. 

Before the data can be correlated, we need to find the bursts in data first, of
course. This was the subject of Franz' live demo, the content of which is briefly
described below.

## PRECISE processing
### Burst finding
After an observation, the data from Eff are transmitted to a dedicated processing server
at Onsala Space Observatory. We built a pipeline that 
- converts the raw baseband data to filterbank format
- searches those filterbanks for bursts
- classifies the bursts candidates as either RFI or astrophysical in origin
- sends the astrophysical candidates to a dedicated Slack channel for further inspection

The pipeline itself can be found in [this git
repo](https://github.com/pharaofranz/frb-baseband). It depends on the following packages:
- [jive5ab](https://github.com/jive-vlbi/jive5ab) and `digifil` (part of
  [DSPSR](https://dspsr.sourceforge.net/)) for the conversion of VDIF/Mark5B data to
  filterbank format
- [Heimdall](https://sourceforge.net/p/heimdall-astro/wiki/Home/) for the burst searching
- [FETCH](https://github.com/devanshkv/fetch) as the burst candidate classifier

### Correlation
Once we found a burst in a particular observation, the data from all participating
stations are transferred to JIVE for correlation. Besides the regular fringe finding and
clock searching, gates are set around the bursts to boost S/N -- see Aard's talk on single
pulse processing. We typically have three
correlator passes:
1. One at high time and frequency resolution aimed at finding the crude burst location in
   the field of view via delay mapping. The correlation is performed at the pointing center.
2. In the second run we use the first, crude localisation as the new phase center and
   correlate all gated bursts at this location. This
   run will provide the data for burst localisation.
3. The third run uses the entire track on the source and uses the same phase center as the
   second run. The aim here is to obtain a deep image to assess whether or not the FRB is
   colocated with a persistent radio source.

### Calibration & Imaging
Calibration itself is the same for FRB observations as it is for any other continuum VLBI
experiments. We correct for the ionosphere, the bandpass, electronic delays and run a
regular fringe fit. In principle, we would only need to run the fringe fitting on the
phase calibrator scans that were taken before and after the target scan that contains the
FRB. However, for the deep image we of course need to calibrate the entire experiment
which also helps at finding issues that might have affected the entire run.

As a FRB signal only lasts of order a millisecond, the UV-coverage per burst in the second
correlator run is very sparse. RFI and residual gain errors can easily shift power from one side
lobe to another, making localisation of a single burst difficult and potentially ambiguous
on the side lobe level. In case multiple bursts were detected in one experiment (or
across several experiments), the UV-plane can be filled a little more by combining the
visibilities and effectively using earth rotation synthesis. In all cases, unless many
bright bursts were detected yielding one unambiguous "peak" in the dirty maps, it is
advisable to rather fit the cross pattern in a dirty image with a 2D-Gaussian than running
CLEAN in the regular fashion. 

<img src="figures/frb-processing/fig1-bhardwaj2025.png" alt="drawing" style="width: 60%;height: auto;" class="center"/>

<a name="fig-1">**Figure 1**</a> - *Dirty maps of individual bursts from FRB
20240114A. Gold ellipses indicate the $2\sigma-\mathrm{ellipse}$ of the 2D Gaussian fit to
the fringe pattern; white ellipses indicate the best fit $2\sigma-\mathrm{localisation
region}$. From [Bhardwaj et al. 2025](#ref_bhardwaj25).*

## References
<a name="ref_petroff22">1.</a> Petroff, E. et al., “Fast radio bursts at the dawn of the
2020s”, *Astronomy and Astrophysics Review*, 30 (1), 2 (2022). DOI: [10.1007/s00159-022-00139-w](https://ui.adsabs.harvard.edu/abs/2022A&ARv..30....2P){:target="_blank"}  

<a name="ref_bhardwaj25">2.</a> Bhardwaj, M. et al., “A Hyperactive FRB Pinpointed in an SMC-Like
Satellite Host Galaxy”, *arXiv*, arXiv:2506.11915 (2025). DOI: [10.48550/arXiv.2506.11915](https://ui.adsabs.harvard.edu/abs/2025arXiv250611915B){:target="_blank"}  


_Content built by Franz Kirsten._ <i><span id="lastModified"></span></i>

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
