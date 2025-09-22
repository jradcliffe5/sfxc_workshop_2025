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
# SFXC workshop 2025 • MPI & Slurm



## On this page
1. [Introduction](#introduction)
2. [Data download](#data-download)
3. [MPI machine file](#mpi-machine-file)
4. [MPI rank file](#mpi-rank-file)
5. [SLURM](#slurm)

## Introduction

SFXC uses the Message Passing Interface (MPI) to distribute the
correlation over multiple machines.  This tutorial will help you to
get familliar with how SFXC uses MPI and how you can use SFXC on a
system that uses the SLURM workload manager.  There are several MPI
implementations, but the most popular one is OpenMPI.  That is the
implementation we will use for this tutorial.  If your cluster uses a
different MPI implemntation, the commands may be different, but the
concepts will be the same.


## Data download

This tutorial uses the same N24L2 dataset as some of the previous tutorials.
If you did do so already, you can download this dataset from:

<https://archive.jive.nl/sfxc-workshop/n24l2>

On the workshop cluster the data is available available in the
`/data/n24l2` directory.


## MPI machine file

The simplest situation is when your cluster has a cluster filesystem
and all your input data lives on that cluster filesystem.  In this
case you don't have to worry where the various SFXC processes run and
all you need is a machine file that lists the machines you want to run
your job on.  You also should specify the number of slots for each
machine.  Set this to the number of CPU cores of the machines.  Here
is an example machine file:

```text
# sfxc.machines
node001 slots=8
node002 slots=8
```

Log in to the head node of your cluster.  If you are using the cluster
we set up for this workshop, the head node is the one mentioned on the
note with your login credentials.  Now create a working directory for
this exercise:

```text
mkdir n24l2-mpi
cd n24l2-mpi
```

Pick two random cluster nodes, and create a machine file for them and
save this as `sfxc.machines` in the current directory.  Also create
a correlator control file.  Here is an example:

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
    "output_file": "file:///home/workshop99/n24l2-mpi/N24L2_No0005.cor",
    "integr_time": 2.0,
    "slices_per_integration": 4,
    "delay_directory": "file:///home/workshop99/n24l2-mpi/delays",
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
gen_all_delay_tables.py n24l2.vix N24L2_No0005.ctrl
mpirun --machinefile sfxc.machines --np 16 `which sfxc` N24L2_No0005.ctrl n24l2.vix
```

Make sure the number of SFXC processed specified by the `-n` option
isn't larger than the total number of slots in your machine file.
After the job runs successfully, check that it did indeed create an
output file.  Then remove it.

Now change the `output_file` parameter in the control file such that
it writes the output in a file in the `/tmp` directory and run the
correlation again.  Can you still find the output file?

It is worth playing around with the `-n` parameter a bit.  What
happens when you set it to a number that is larger than the number of
slots?  What happens if you set it to a very small number?


## MPI rank file

In the last excercise, the correlator output perhaps didn't land where
you expected it to be.  We can use a rank file to have better control
over where we read the input data and where we write the output.  Here
is an annotated example:

```text
# sfxc.ranks
rank 0 = node001 slot=0  # manager
rank 1 = node001 slot=1  # log
rank 2 = node002 slot=0  # output
rank 3 = node002 slot=1  # input 'Cm'
rank 4 = node001 slot=2  # input 'De'
rank 5 = node002 slot=2  # input 'Ef'
rank 6 = node001 slot=3  # input 'Hh'
rank 7 = node002 slot=3  # correlator node 0
rank 8 = node001 slot=4
rank 9 = node002 slot=4
rank 10 = node001 slot=5
rank 11 = node002 slot=5
rank 12 = node001 slot=6
rank 13 = node002 slot=6
rank 14 = node001 slot=7
rank 15 = node002 slot=7
```

Save this file as `sfxc.ranks` and remove any output files created
during the previous exercises in this tutorial.  Then run the
correlator again but now specify the ranks file:


```text
mpirun --machinefile sfxc.machines --rankfile sfxc.ranks --np 16 `which sfxc` N24L2_No0005.ctrl n24l2.vix
```

Looking at the ranks file, on which node do you expect the output?
Can you find it there?


## SLURM

Most clusters are shared resources.  Picking some random cluster nodes
to run SFXC may not work.  These clusters typically use a job control
system.  A popular job control system is SLURM.  Most SLURM
installations provide MPI integration.  On these systems you don't
even have to create a machine file to run SFXC.  But before you run
SFXC you will have to allocate some resources.  To allocate resources
to run SFXC with 16 processes you can do:

```text
salloc -n16
```

If there are enough resources available this will start a new shell,
where you can run commands interactively.  This shell "knows" about
the resources that were reserved for you (if you are curious how, use
the `env` command in your shell and inspect its output).  We can now
use `mpirun` without specifying a machine file to run SFXC:

```text
mpirun `which sfxc` N24L2_No0005.ctrl n24l2.vix
```

After this command finished exit the shell to give up your resource
reservation:

```text
exit
```

A better way to use SLURM is to make use of batch jobs.  This way you
queue multiple correlation jobs that will run as soon as resources are
available to run them on the cluster.  And resources will be
automatically released when jobs finish.  To do this, create a file
named `sfxc.batch` with the command you want to run:

```text
#!/bin/sh
#SBATCH -n 16
mpirun $HOME/opt/bin/sfxc N24L2_No0005.ctrl n24l2.vix
```

Here we assume you have installed SFXC in `$HOME/opt`.  If you
installed it somewhere else you'll need to move it.

You can now submit your job with the `sbatch` command:

```text
sbatch slurm.batch
```

You can keep an eye on your running jobs using the `squeue` command.
Also keep an eye on the correlator output.  The terminal output of
your job will go to a file name `slurm-###.out` where `###` is the
number of your job.  The `tail` command is useful for keeping track of
the correlator progress:

```text
tail -f slurm-99.out
```


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
