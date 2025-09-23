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
# Installing SFXC

## Prerequisite packages

Life is hard, and operating systems are even worse. The current
prerequisites for SFXC on Debian 12 are given in the [Apptainer recipe]
in the source repository as:

```
apt-get install -y python3 python3-pip build-essential git flex bison \
                   openmpi-bin libopenmpi-dev python-is-python3 \
                   libfftw3-dev libgsl-dev gfortran-multilib g++-multilib
```

As there are some doubts whether the CALC10 software used by SFXC is
64-bit safe, the model generation tools are compiled in 32-bit mode.
This means that if you are using a 64-bit (x86_64) Linux system, you
will have to install 32-bit versions of the C++ and Fortran compilers
and their support libraries; these packages are included in the above
list.

The Apptainer recipe is actively maintained and should be considered
definitive. Finding equivalent packages for other flavours of Linux and
Unix (including Apple systems) is left as an exercise for the interested reader.


## SFXC software installation 

The SFXC software correlator can be distributed under the terms of the General Public License (GPL).
The SFXC Git repository can be found at [[https://code.jive.eu/JIVE/sfxc]]

The repository can be downloaded using:

```
git clone https://code.jive.eu/JIVE/sfxc.git
```

The current production release is taken from the stable-5.1 branch which can be checked out using:

```
git checkout stable-5.1
```

In principle this branch will only receive bug fixes. Development of
new features happens on the master branch, which can be checked out
using:

```
git checkout master
```

Note that using that version is not recommended unless you feel like
being a guinea pig for testing the bugs we introduce while adding all
the new cool stuff.

Building SFXC should be as easy as:

```
cd sfxc
./compile.sh
./configure CXX=mpicxx
make
make install
```

This will install SFXC in `/usr/local`, which typically requires
super-user privileges and may not be what you want.  You can specify a
different location by using the `--prefix` option when running
`configure`.

SFXC can optionally use the Intel Performance Primitives (IPP) library.
See the `--enable-ipp` and `--with-ipp-path` options to `configure`.

##  GUI tools 

SFXC comes with a couple of GUI tools to visualize the correlation
results. This needs some additional Python packages, known to `apt` as
`python3-ply` and `python3-qwt5`, along with the Python VEX parser
module that can be found in the `vex/` top-level subdirectory.  This
module uses a standard Python distutils `setup.py`, so can be installed
by:

```
  cd vex
  python setup.py build
  python setup.py install
```

The last command will probably require root priviliges; `setup.py`
offers a couple of alternative installation methods that avoid this.
More information on the VEX parser is provided in `vex/README`.

## Post-processing software 

To convert the SFXC correlator output into FITS-IDI, additional tools
are needed.  Information on how to obtain and build these tools is
available at
[https://code.jive.eu/verkout/jive-casa](https://code.jive.eu/verkout/jive-casa).

## Usage

A very basic User's Manual for SFXC can be found
[here](https://www.jive.eu/jivewiki/doku.php?id=sfxc-guide).

We kindly request users of SFXC to reference our paper that describes
its algorithm and implementation [The SFXC software correlator for very
long baseline interferometry: algorithms and implementation,
A. Keimpema et. al., Experimental Astronomy, Volume 39, Issue 2,
pp.259-279](http://adsabs.harvard.edu/abs/2015ExA....39..259K).

---

_Content built by Des Small (JIVE)._ <i><span id="lastModified"></span></i>

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