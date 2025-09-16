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
# SFXC workshop 2025 • Timetable

**Dates:** 21–23 September 2025  
**Timezone:** Europe/Amsterdam

> Based on the Indico timetable’s detailed view (numbered + minutes). See the [official Indico page](https://indico.astron.nl/event/410/timetable/) for any late changes.

---

## Sunday, 21 September 2025
- **18:00–20:00** — **Ice‑breaker reception**  
  _At the venue - ASTRON's premises_

## Monday, 22 September 2025

### Theory 1 — 09:00–10:30
- **09:00–09:45** — **(VLBI) Correlation 101** — *Bob Campbell (JIVE)*  
  _Introduction to correlation and correlator basics, history._
- **09:45–10:30** — **Correlation preparation** — *Aard Keimpema (JIVE), Mark Kettenis (JIVE)*  
  _Knowing what (meta)data is needed for correlation, where to get it and how to store/format it._

**10:30–11:00** — Coffee

### Hybrid session 1 / Theory 2 — 11:00–12:30
- **11:00–12:30** — **Setting up the correlator** — *Aard Keimpema (JIVE), Mark Kettenis (JIVE)*  
  _After an introduction to setting up a cluster for running the SFXC distributed correlator, participants will set up and verify the operation of their correlation environment._

**12:30–13:30** — Lunch

### Hands‑on — 13:30–16:15
- **13:30–15:00** — **Hands‑on 1: Getting everyone's correlator to run** — *Aard Keimpema (JIVE), Mark Kettenis (JIVE)*  
  _On a simple VLBI observation._
- **15:00–15:30** — Tea
- **15:30–16:15** — **Hands‑on 2: Post‑correlation processing** — *Marjolein Verkouter (JIVE)*  
  _Translating the SFXC output into more standard data format(s) for inspection and export/archiving._

**16:15–17:00** — Q+A

**18:25–20:55** — Workshop dinner


## Tuesday, 23 September 2025

### Hybrid 3: Pulsar processing — 09:00–10:30
- **09:00–09:30** — **Pulsar processing** — *Aard Keimpema (JIVE), Franz Kirsten (Onsala)*  
  _Introduction._
- **09:30–10:00** — **Pulsar gating** — *Franz Kirsten (Onsala)*  
  _Handling a FITS file with pulses in._
- **10:00–10:30** — **Producing filterbank format output & running PSR tools** — *Franz Kirsten (Onsala)*

**10:30–11:00** — Coffee

### Hybrid 4: FRB Processing — 11:00–12:30
- **11:00–11:30** — **PRECISE processing** — *Franz Kirsten (Onsala)*  
  _How the PRECISE EVN‑lite mode actually works and what it does._
- **11:30–12:00** — **Semi‑automated pulse processing** — *Aard Keimpema (JIVE)*  
  _How to use and run the semi‑automatic gating system._
- **12:00–12:30** — **SFXC and duct‑tape = single dish FRB workflow** — *Omar Ould‑Boukattine (UvA, ASTRON)*  
  _How several components can be duct‑taped to form a workflow for unique science._

**12:30–13:30** — Lunch

### Hybrid 5: Wide‑field processing — 13:30–15:00
- **13:30–15:00** — **Wide‑field processing** — *Jack Radcliffe (University of Manchester)*  
  _Bright source, two images, check fringes (same dataset as Day 1)._

**15:00–15:30** — Tea

### Hybrid 6: Geodesy — 15:30–16:15
- **15:30–16:15** — **Geodesy** — *Mark Kettenis (JIVE)*  
  _Same dataset as Day 1; check geodesy data on flexbuffs @ JIVE. Running `sfxc2mark4`, pulse‑cal, and `fourfit`._

### Future bells & whistles — 16:15–17:00
- **16:15–16:25** — **SFXC — the GPU version** — *Aard Keimpema (JIVE)*  
  _Recent results from porting SFXC to GPU‑accelerated platforms._
- **16:25–16:35** — **SKA‑VLBI** — *Jack Radcliffe (University of Manchester), Mark Kettenis (JIVE)*  
  _Multi‑beam processing; plan to implement as multiple field‑center correlation with extra bookkeeping._
- **16:35–17:00** — **User/community input discussion**  
  _Knee‑jerk feedback and community needs/wishes._

_This page is a human‑readable export for GitHub Pages; always cross‑check the official [Indico page](https://indico.astron.nl/event/410/timetable/) for the latest schedule._

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
