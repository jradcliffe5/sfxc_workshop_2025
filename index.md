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

# SFXC workshop 2025

Welcome! This page hosts the **tutorial materials** and **guides** from the first **SFXC Workshop**, held at **JIVE** (Joint Institute for VLBI in Europe; [JIVE](https://jive.eu){:target="_blank"}) on **21–23 September 2025**.


## About the workshop

The SFXC Workshop is designed for **VLBI users, correlator operators, and developers** interested in **software correlation** with SFXC. It combines **theory sessions** with **hands-on exercises**, covering:

- Fundamentals of **SFXC**.
- Advanced modes such as **wide-field correlation** and **FRB workflows**.
- Technical developments such as **GPU-based correlation** and **future roadmap**.
- Open discussions on community needs and pipeline automation.

This site serves as the central hub for **tutorial content** and **step-by-step guides** prepared for the workshop. For logistical information please consult the [Indico page](https://indico.astron.nl/event/410/){:target="_blank"}.


## Timetable

The timetable can be found on the [timetables page](timetable.md). The slides for the theory sessions are available on this page, while the hands-on sessions are listed below.

## Lectures
- [(VLBI) Correlation 101]() - Bob Campbell (JIVE)
- [Correlation preparation]() - Aard Keimpema (JIVE), Mark Kettenis (JIVE)
- [Setting up the correlator]() - Aard Keimpema (JIVE), Mark Kettenis (JIVE)
- [Pulsar processing]() - Aard Keimpema (JIVE), Franz Kirsten (Onsala)
- [ADD MORE HERE]() - Some person

## Tutorials

- [Installing SFXC](installing_sfxc_locally.md)
- [Running SFXC](running_sfxc_locally.md)
- [Basic SFXC tutorial](sfxc-basic-tutorial.md)
- [Post-correlation processing](correlation_post.md)
- [Pulsar processing](pulsar-processing.md)
- [FRB processing](frb-processing.md)
- [Wide-field processing](wide-field.md)
- [Geodesy](geodesy.md)

## Appendices
- [Control file parameters](control_file_parameters.md)

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
