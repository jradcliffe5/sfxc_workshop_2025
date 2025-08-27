
<!-- Prism CSS -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism.min.css" />
<link id="prism-dark" rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css" disabled />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/plugins/line-numbers/prism-line-numbers.min.css" />

<!-- Prism JS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/plugins/line-numbers/prism-line-numbers.min.js"></script>


<!-- MathJax -->
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    jax: ["input/TeX", "output/HTML-CSS"],
    extensions: ["tex2jax.js"],
    "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] },
    tex2jax: { inlineMath: [ ["$","$"], ["\\(","\\)"] ], displayMath: [ ["$$","$$"], ["\\[","\\]"] ], processEscapes: true },
    TeX: { noUndefined: { attributes: { mathcolor: "red", mathbackground: "#FFEEEE", mathsize: "90%" } } },
    messageStyle: "none"
  });
</script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js"></script>


<style>
    *, *::before, *::after { box-sizing: border-box; }
    html, body { height: 100%; }
    :root {
      --bg: #0b1220;
      --bg-soft: #0f172a;
      --card: #0f172a;
      --text: #e5e7eb;
      --muted: #94a3b8;
      --border: #1f2937;
      --accent: #60a5fa;
      --accent-ink: #0b1220;
      --radius: 16px;
      --shadow: 0 1px 2px rgba(2,6,23,.25), 0 10px 35px rgba(2,6,23,.35);
      --maxw: 1000px;
    }
    @media (prefers-color-scheme: light) {
      :root {
        --bg: #fbfdff;
        --bg-soft: #f8fafc;
        --card: #ffffff;
        --text: #0f172a;
        --muted: #475569;
        --border: #e2e8f0;
        --accent: #2563eb;
        --accent-ink: #ffffff;
        --shadow: 0 1px 2px rgba(15,23,42,.06), 0 10px 35px rgba(15,23,42,.08);
      }
    }

    html { scroll-behavior: smooth; }
    body {
      margin: 0;
      color: var(--text);
      background:
        radial-gradient(1200px 600px at 5% -10%, rgba(99,102,241,.18), transparent 45%),
        radial-gradient(1000px 500px at 95% -10%, rgba(59,130,246,.18), transparent 45%),
        var(--bg);
      font: 16px/1.6 ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      min-height: 100dvh;
      display: grid;
      grid-template-rows: auto 1fr;
    }
    ul {text-align: justify;}
    ol {text-align: justify;}

    ::selection { background: color-mix(in oklab, var(--accent) 25%, transparent); }
    a { color: var(--accent); text-underline-offset: 3px; }

    .container { max-width: var(--maxw); margin: 0 auto; padding: 2.2rem clamp(1rem, 2vw, 2rem); }

    header.site {
      position: sticky; top: 0; z-index: 50;
      backdrop-filter: blur(10px) saturate(140%);
      background: color-mix(in oklab, var(--bg) 65%, transparent);
      border-bottom: 1px solid var(--border);
    }
    .site-inner { display: grid; grid-template-columns: 1fr auto; gap: 1rem; align-items: center; }
    .brand {
      display: flex;
      flex-direction: column;
      font-weight: 900;
      letter-spacing: .3px;
    }
    .brand-main {
      font-size: clamp(1.6rem, 4vw, 2.4rem);
      background: linear-gradient(90deg, var(--accent), #a855f7);
      -webkit-background-clip: text; background-clip: text;
      -webkit-text-fill-color: transparent;
    }
    .brand-sub {
      font-size: 0.9rem;
      font-weight: 600;
      color: var(--muted);
    }

    nav ul { display: flex; gap: .6rem; list-style: none; margin: 0; padding: 0; }
    nav a { display: inline-block; padding: .5rem .8rem; border-radius: 999px; border: 1px solid var(--border); background: color-mix(in oklab, var(--bg-soft) 85%, transparent); }
    nav a:hover { border-color: color-mix(in oklab, var(--accent) 50%, var(--border)); }

    .viewport-area { min-height: 0; }
    .grid { display: grid; grid-template-columns: 1fr; gap: 2rem; height: 100%; }
    @media (min-width: 1000px) {
      .grid { grid-template-columns: 270px 1fr; }
      aside.toc { position: sticky; top: 5.5rem; max-height: calc(100dvh - 6rem); overflow: auto; }
    }
    .container.grid { height: 100%; padding-block: 1.6rem; overflow: hidden; }
    main { height: 100%; overflow: auto; padding-right: .2rem; }

    .card { background: var(--card); border: 1px solid var(--border); border-radius: var(--radius); box-shadow: var(--shadow); }
    .toc { padding: 1rem; }
    .toc h2 { margin: .2rem 0 .6rem; font-size: .9rem; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; }
    .toc ol { margin: 0; padding: 0 0 0 1.1rem; }
    .toc a { color: inherit; text-decoration: none; }
    .toc a:hover { color: var(--accent); text-decoration: underline; }

    main h1 { font-size: clamp(2rem, 4vw, 2.8rem); line-height: 1.12; margin: .2rem 0 1rem; }
    main h2 { font-size: clamp(1.35rem, 2.2vw, 1.7rem); margin: 2rem 0 .7rem; display: flex; align-items: center; gap: .5rem; }
    main h3 { font-size: clamp(1.08rem, 1.8vw, 1.2rem); margin: 1.3rem 0 .4rem; }
    p { margin: .5rem 0 1rem; text-align: justify}
    .lede { color: var(--muted); font-size: 1.06rem; }

     .steps { counter-reset: step; }
    .step { position: relative; padding: 1.2rem 1.2rem 1.2rem 3.6rem; border: 1px solid var(--border); border-radius: var(--radius); background: linear-gradient(180deg, color-mix(in oklab, var(--card) 88%, transparent), color-mix(in oklab, var(--bg-soft) 75%, transparent)); box-shadow: var(--shadow); }
    .step + .step { margin-top: 1rem; }
    .step::before {
      counter-increment: step;
      content: var(--step-prefix) counter(step);
      position: absolute; left: .8rem; top: 1rem;
      width: 2.2rem; height: 2.2rem; border-radius: 999px; display: grid; place-items: center; font-weight: 800; color: var(--accent-ink); background: var(--accent); box-shadow: var(--shadow);
    }

    .callout { border: 1px solid var(--border); border-radius: var(--radius); padding: .9rem 1rem; background: color-mix(in oklab, var(--bg-soft) 90%, transparent); }
    .callout strong { color: var(--muted); text-transform: uppercase; font-size: .78rem; letter-spacing: .06em; }

    pre[class*="language-"] { border-radius: var(--radius); border: 1px solid var(--border); box-shadow: var(--shadow); margin: .7rem 0; }
    pre[class*="language-"] code { white-space: pre-wrap; word-break: break-word; overflow-wrap: anywhere; }
    .code-header { display: flex; align-items: center; justify-content: space-between; gap: .6rem; font-size: .9rem; color: var(--muted); padding: .6rem .9rem; border: 1px solid var(--border); border-bottom: 0; border-top-left-radius: var(--radius); border-top-right-radius: var(--radius); background: color-mix(in oklab, var(--bg-soft) 88%, transparent); }
    .badge { font-size: .72rem; border: 1px solid var(--border); padding: .15rem .5rem; border-radius: 999px; }
    .copy-btn { appearance: none; border: 1px solid var(--border); background: color-mix(in oklab, var(--bg-soft) 85%, transparent); padding: .35rem .6rem; border-radius: 10px; color: var(--text); cursor: pointer; }
    .copy-btn:hover { border-color: color-mix(in oklab, var(--accent) 50%, var(--border)); }

    pre.line-numbers { padding-left: 3.2em; }
    .line-numbers .line-numbers-rows { border-right: 1px solid var(--border) !important; }

    @media (prefers-color-scheme: dark) { #prism-dark[disabled] { display: none; } }

    footer { color: var(--muted); font-size: .95rem; border-top: 1px solid var(--border); margin-top: 3rem; padding-top: 1rem; }
    @media print { header.site, .toc, .copy-btn { display: none !important; } body { background: #fff; } .container { padding: 0; } }

/* Extra rules so the theme applies nicely to plain Markdown renderers */
body { max-width: 1000px; margin: 0 auto; padding: 2rem 1rem; }
main, article, section { all: unset; }
h1, h2, h3 { scroll-margin-top: 5rem; }
code, pre { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
pre { padding: 1rem; border-radius: var(--radius); border: 1px solid var(--border); background: color-mix(in oklab, var(--bg-soft) 90%, transparent); }
blockquote { border-left: 4px solid var(--accent); padding-left: 1rem; color: var(--muted); }
hr { border: none; border-top: 1px solid var(--border); margin: 2rem 0; }
a { text-decoration: underline; }
ul, ol { padding-left: 1.3rem; }

</style>

# SFXC workshop 2025 • Wide-field VLBI

_Built with ♥ — HTML + CSS + Prism.js + a bit of AI. © Jack Radcliffe (2025)_

This page outlines the wide-field VLBI correlation that was presented as part of the first **SFXC workshop**, held on **21–23 September 2025** at the Joint Institute for VLBI in Europe ([JIVE](https://jive.eu)). For more information and resources regarding the workshop, see the [workshop webpage](https://indico.astron.nl/event/410).

The definition of the Euler–Mascheroni constant is:

$$
\gamma = \lim_{n\to\infty}\left(\sum_{k=1}^n \frac{1}{k} - \ln(n)\right)
$$

## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)
3. [Project setup](#project-setup)
4. [Correlator preparation](#correlator-preparation)
5. [Running the correlator](#running-the-correlator)
6. [Post processing](#post-processing)
7. [Current & future developments](#current--future-developments)
8. [Resources](#resources)

## Introduction
Wide-field VLBI is a specialised correlation mode that …

**Folder structure**
```text
tutorial/
├─ index.html
└─ (images, assets, etc.)
```

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

## Post processing
The correlator produces a custom `.cor` format that you can convert to standard interferometric formats.

Conversion generally uses helper software from the **jive-casa** repository: <https://code.jive.eu/verkout/jive-casa>.
A Singularity image may be available at `/SOFTWARE/jive-casa/jive-casa.img`.

**Convert to Measurement Set with `j2ms2`:**
```bash
singularity run --app j2ms2 /SOFTWARE/jive-casa/jive-casa.img <correlator_outputs>.corr
```

This produces `<vix_prefix>.ms`. If multiple correlator outputs exist, pass them all as arguments to `j2ms2`.

**Convert MS to FITS-IDI with `tConvert`:**
```bash
singularity run --app tConvert /SOFTWARE/jive-casa/jive-casa.img <measurement_set.ms> <output_idi>.IDI
```

If IDI files exceed 2 GB, they may be split into ~1.9 GB chunks (as on the EVN archive).

## Current & future developments
- Containerised software
- Automated wide-field correlation software
- End-to-end correlation & calibration

## Resources
### Technical papers/memos on wide-field correlation
1. Deller, A. T., et al., “DiFX-2: A More Flexible, Efficient, Robust, and Powerful Software Correlator”, *PASP*, 123(901), 275 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011PASP..123..275D/abstract>
2. Morgan, J. S., et al., “VLBI imaging throughout the primary beam using accurate UV shifting”, *A&A*, 526, A140 (2011). doi: <https://ui.adsabs.harvard.edu/abs/2011A%26A...526A.140M/abstract>
3. Keimpema, A., et al., “The SFXC software correlator for very long baseline interferometry: algorithms and implementation”, *Experimental Astronomy*, 39(2), 259–279 (2015). doi: <https://ui.adsabs.harvard.edu/abs/2015ExA....39..259K/abstract>


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

    // Auto-enable dark Prism theme when user prefers dark
    const darkLink = document.getElementById('prism-dark');
    const mq = window.matchMedia('(prefers-color-scheme: dark)');
    if (mq.matches) darkLink.disabled = false;
    mq.addEventListener?.('change', (ev) => { darkLink.disabled = !ev.matches; });
</script>
