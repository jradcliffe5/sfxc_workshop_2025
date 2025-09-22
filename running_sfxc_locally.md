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

# Running SFXC

## Example Data

Some example data is
[available](https://archive.jive.nl/sfxc-workshop/n24l2) on the EVN
archive, taken from the Network Monitoring Experiment N24L2. 

See the [README](https://archive.jive.nl/sfxc-workshop/n24l2/README)
file for details.

Note that in this case the data for each station has been merged into
regular files; there is no need to use `vbs_fs` to assemble them.

You can download the contents with the following incantation:

```
wget -t45 -l1 -r -nd https://archive.jive.eu/sfxc-workshop/n24l2 -A "*"
```

Note that although this is just snippets of data, by VLBI standards, it
is still several gigabytes of downloads; it took me 15 minutes to
download the data to a machine inside the JIVE network. So you may want
to have a cup of coffee while this download runs.

## Using `vbs_fs`

On JIVE's flexbuff and mark6 machines, `vdif` files are commonly stored
in small pieces; we use a user-level filesystem `vbs_fs` implemented
via Linux's `FUSE` interface to hide this level from other parts of the
software, and so can you! 

`vbs_fs` has strong opinions about the organisation of the chunks on
disk; a virtual file like `n25l1_cm_no0013.vdif` must be a directory in
which the corresponding chunks
`n25l1_cm_no0013.vdif/n25l1_cm_no0013.vdif.00000000` etc. are stored.

For the example in this section we assume a data layout of the form:

```
Data/Cm/n25l1_cm_no0013.vdif/n25l1_cm_no0013.vdif.00000000
Data/Cm/n25l1_cm_no0013.vdif/n25l1_cm_no0013.vdif.00000001
[...]
Data/Ef/n25l1_ef_no0013/n25l1_ef_no0013.00000000
Data/Ef/n25l1_ef_no0013/n25l1_ef_no0013.00000001
[...]
Data/Hh/n25l1_hh_no0013/n25l1_hh_no0013.00000000
Data/Hh/n25l1_hh_no0013/n25l1_hh_no0013.00000001
[...]
Data/Tr/n25l1_tr_no0013/n25l1_tr_no0013.00000000
Data/Tr/n25l1_tr_no0013/n25l1_tr_no0013.00000001
[...]
```
With that in place, we can run `vbs_fs` to mount all those chunks as
virtual files on a (pre-existing) mountpoint vmount:

`vbs_fs -m 3 -R ${PWD}/Data/\* vmount/`

And then we can look in that directory and see the aggreggated virtual files:

```
$ ls -lt vmount/
total 0
-rw-r--r-- 0 small small 2815954944 May  8 11:35 n25l1_tr_no0013
-rw-r--r-- 0 small small 2815954944 May  8 11:24 n25l1_hh_no0013
-rw-r--r-- 0 small small 1342147200 May  8 11:17 n25l1_kn_no0013.vdif
-rw-r--r-- 0 small small 1342147200 May  8 11:17 n25l1_cm_no0013.vdif
-rw-r--r-- 0 small small 2815954944 May  8 11:07 n25l1_ef_no0013
```

## Not using `vbs_fs`

On the other hand if, as in the archive example above, the datafiles have been provided to you in
aggregated form, you can simply use them as they are.

## SFXC Control files

If you have a correct vex file for the experiment, you can generate
correlator control files for it with the `generate_jobs.py` utility
including in the SFXC distribution.

`python $SFXC_Dir/sfxc/sfxc/utils/generate_jobs.py -s Cm,Ef,HH,De -S Ef n24l2.vix`
    
Note, though, that this will take the vex file's word for where the
data is stored; if you have a local copy then you will need to adjust
the vex file or the control file accordingly.

Note also that you will need to edit the start times to match the time
range of the snippets of data provided for the tutorials. The program
`vdif_print_headers`, included in the SFXC distribution, will can print
the timestamp of the first frame of `vdif` data:


```
$ vdif_print_headers -n1 Data/n24l2_cm_no0004.vdif

, pos = 0, nbytes = 0, frames = 0
2024y144d12h41m41.000s ,frame_nr = 3118, thread_id = 0, nchan = 1, invalid = 0, legacy = 0, station = Cm, bps-1 = 1, data_size = 8032
```

So we need to set a start time of `2024y144d12h41m41` in our control file.

## MPI wrangling

Don't forget to set your `$CALC_DIR` environment variable and to make a
delay directory to put the delay files in. (If you forget the former,
delete any incomplete delay files that get created; SFXC will trip over
them if you don't.)

```
export CALC_DIR=/home/small/LocalProjects/WorkingCopies/SFXC-Workshop/SFXC-installation/sfxc/sfxc/lib/calc10/data/
```

SFXC is an MPI program and needs to be run via `mpirun`.  If you are
running on your own laptop and you are just want something to *happen*
as soon as possible you can use a control file, as generated above, and
a vexfile and run without specifying much of anything, although you do
have to tell it you don't mind if there aren't as many mpi slots as it
expects with the `--oversubscribe` option.

Note also that sfxc doesn't generate its own logfiles; if you want to
know what it said while running you need to redirect its output yourself.

```
mpirun -np 16 --oversubscribe sfxc Control/n24l2_no0004.ctrl n24l2.vix
2>&1 n24l2_no0004.log
```

The output, in this case `n24l2_no0004.cor` is written where the
control file specifies, but some transient files associated with
channel-unpacking are also written in the directory where the `mpirun`
command is executed. In this case we get:

```
chex.7OdoAq
'dynamic_channel_extractor[4-10000][2bit][jiAy][nmED][barq][fevu][lkCB][poGF][dcts][hgxw].so'
```
These can safely be ignored or deleted after the run.

## MPI Wrangling Properly

(Descriptions of machine and slot files go here)

## Checking output

The SFXC distribution also includes the utility 
`print_corfile.py` which we can use to quickly check the
output. Running it on the output file `n24l2_no0004.cor` that we just
generated, it ends with 

```

Baseline: station1 = Cm, station2 = De
freq = 0, sb = 1, pol = 0, fringe ampl = 0.001611, SNR = 23.605510, offset = 1, weight = 63983855.000000
freq = 0, sb = 1, pol = 3, fringe ampl = 0.001242, SNR = 19.140606, offset = 0, weight = 63998976.000000
freq = 1, sb = 0, pol = 0, fringe ampl = 0.002678, SNR = 45.299266, offset = 0, weight = 63983855.000000
freq = 1, sb = 0, pol = 3, fringe ampl = 0.002160, SNR = 36.408778, offset = -1, weight = 63998976.000000
Baseline: station1 = Cm, station2 = Ef
freq = 0, sb = 1, pol = 0, fringe ampl = 0.006921, SNR = 132.656400, offset = 0, weight = 63983856.000000
freq = 0, sb = 1, pol = 3, fringe ampl = 0.011368, SNR = 196.009518, offset = 0, weight = 63998976.000000
freq = 1, sb = 0, pol = 0, fringe ampl = 0.008154, SNR = 139.960749, offset = 1, weight = 63983856.000000
freq = 1, sb = 0, pol = 3, fringe ampl = 0.012091, SNR = 225.846521, offset = -1, weight = 63998976.000000
Baseline: station1 = De, station2 = Ef
freq = 0, sb = 1, pol = 0, fringe ampl = 0.005670, SNR = 103.679537, offset = 0, weight = 63985142.000000
freq = 0, sb = 1, pol = 3, fringe ampl = 0.004634, SNR = 80.771258, offset = 0, weight = 63998976.000000
freq = 1, sb = 0, pol = 0, fringe ampl = 0.006006, SNR = 105.001002, offset = 0, weight = 63985142.000000
freq = 1, sb = 0, pol = 3, fringe ampl = 0.007483, SNR = 128.739404, offset = 0, weight = 63998976.000000
Reached end of file
```

Since we are after all correlating an NME full of nice bright targets,
we expect a pleasantly high SNR and that's exactly what we get. 

## Post-correlation

### Installing j2ms2 (and tConvert)
We need j2ms2 to turn the SFXC correlation files into the wildly
popular and universally loved Measurement Set
format so other tools can work with them. And to install j2ms2 we need
to install the `myvex` package first.

I want them both installed in `/usr/local` because I am a simple person
and that's where I think local sofware goes; the documentation for both
packages describes the procedure for alternative locations.

```
git clone http://code.jive.eu/verkout/myvex.git
cd myvex/
cmake .; make
sudo make install
cd ..
```

```
git clone https://code.jive.eu/verkout/jive-casa.git j2ms2_etc
cd ../j2ms2_etc/
cmake -DVEX_ROOT_DIR=/usr/local; make
sudo make install
cd ..
```

And if you installed them somewhere bespoke, do make sure they are in
your shell's path.

`tConvert` can turn Measurement Set data into IDI or UV FITs, if that's
what you want. It will be installed by the above incantation but I
don't plan to elaborate on it.

### Running `j2ms2`

The control files you generated above will each specify a "reference_station"
```
j2ms2 eo:setup_ref_station=Ef -o n24l2.ms no0004/*.cor
```

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
