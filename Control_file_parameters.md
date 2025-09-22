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
# Appendix
## Control parameters

### channels\<List of strings\> *(default=All channels from setup station)*

List of channels to correlate, where the the channel names are specified in the $FREQ block of the *setup_station* in the VEX file.

This parameter is optional in the latest development version of SFXC, where if it's not specified all channels from the *setup_station* will be correlated.
In older versions such as stable-5.1, this parameter should always be specified.

Example $FREQ block (from a VEX file):
```
$FREQ;
def 1610.49MHz16x16MHz;
   sample_rate = 32 Ms/sec;
   chan_def =   : 1610.4900 MHz : L : 16 MHz : &CH01 : &BBC02 : &NoCal;
   chan_def =   : 1610.4900 MHz : L : 16 MHz : &CH02 : &BBC01 : &NoCal;
   chan_def =   : 1610.4900 MHz : U : 16 MHz : &CH03 : &BBC02 : &NoCal;
   chan_def =   : 1610.4900 MHz : U : 16 MHz : &CH04 : &BBC01 : &NoCal;
   chan_def =   : 1642.4900 MHz : L : 16 MHz : &CH05 : &BBC04 : &NoCal;
   chan_def =   : 1642.4900 MHz : L : 16 MHz : &CH06 : &BBC03 : &NoCal;
   chan_def =   : 1642.4900 MHz : U : 16 MHz : &CH07 : &BBC04 : &NoCal;
   chan_def =   : 1642.4900 MHz : U : 16 MHz : &CH08 : &BBC03 : &NoCal;
   chan_def =   : 1674.4900 MHz : L : 16 MHz : &CH09 : &BBC06 : &NoCal;
   chan_def =   : 1674.4900 MHz : L : 16 MHz : &CH10 : &BBC05 : &NoCal;
   chan_def =   : 1674.4900 MHz : U : 16 MHz : &CH11 : &BBC06 : &NoCal;
   chan_def =   : 1674.4900 MHz : U : 16 MHz : &CH12 : &BBC05 : &NoCal;
   chan_def =   : 1706.4900 MHz : L : 16 MHz : &CH13 : &BBC08 : &NoCal;
   chan_def =   : 1706.4900 MHz : L : 16 MHz : &CH14 : &BBC07 : &NoCal;
   chan_def =   : 1706.4900 MHz : U : 16 MHz : &CH15 : &BBC08 : &NoCal;
   chan_def =   : 1706.4900 MHz : U : 16 MHz : &CH16 : &BBC07 : &NoCal;
enddef;
```

To correlate the CH15 and CH16 from this $FREQ block:

```
    "channels": ["CH15", "CH16"]
```

### cross_polarize \<boolean\> *(Optional, default=false)*

When enabled, cross-polarisations are computed, meaning that LCP and RCP channels from the same sub-band will be correlated agains each other.

Example:
```
"cross_polarize": true
```

### data_sources \<Dictionary\>

A dict of URLs where for each station there is an entry that indicates where the input data to correlate is located, this can be either files, network sockets, or Mark5 diskpacks.

A basic example *data_sources* for two stations stations Ef, and On, correlating from files would be:
```
"data_sources": {
    "Ef: ["file:///data/exp/file1.vdif", "file:///data/exp/file2.vdif"],
    "On": ["file:///media/data/other_file1.vdif"]
}
```

When VEX2 $DATASTREAMS are used the syntax is slighly different.
Below is an example $DATASTREAMS containing two DATASTREAMS $DS1, and $DS2.
```
$DATASTREAMS;
def DSxx;
   datastream = &DS1 : VDIF;
   datastream = &DS2 : VDIF;
   thread = &DS1 : &thread0 : 0 : 4 : 64 Ms/sec : 2 : real : 8000;
   thread = &DS2 : &thread1 : 0 : 4 : 64 Ms/sec : 2 : real : 8000;
   channel = &DS1 : &thread0 : &CH01 : 0;
   channel = &DS1 : &thread0 : &CH03 : 1;
   channel = &DS1 : &thread0 : &CH05 : 2;
   channel = &DS1 : &thread0 : &CH07 : 3;
   channel = &DS2 : &thread1 : &CH02 : 0;
   channel = &DS2 : &thread1 : &CH04 : 1;
   channel = &DS2 : &thread1 : &CH06 : 2;
   channel = &DS2 : &thread1 : &CH08 : 3;
enddef;
```
A possible *data_sources* to correlate these two datastreams is:
```
"data_sources": {
    "On": {"DS1": ["file:///media/data/file_ds1.00001", "file:///media/data/file_ds1.00002"],
           "DS2": ["file:///media/data/file_ds2.00001", "file:///media/data/file_ds2.00002"]}
}
```
So in the example without $DATASTREAMS we simply specify a single list of inputs for each station, but
in the case $DATASTREAMS are used then for each station we specify a dict where each entry corresponds to a list of inputs
belonging to a particular datastream.

The above examples show how to correlate from files, in this case the absulute path is simply prependended by *file://*

When correlating from a diskpack the URL format is *mk5://<IP ADDRESS>/<VSN>:<BYTE OFFSET>*
- <IP ADDRESS> is the IP address of the Mark5 playback unit. Note that correlating from diskpack requires the mk5read daemon to be running on the Mark5 unit.
- <VSN> is the name of the diskpack, this is usually specified in the $TAPELOG_OBS section of the VEX file
- <BYTE OFFSET> is the byte position in the Mark5 diskpack. The byte offsets for each scan are usually given in the $SCHED section of the VEX file.
If the byte offset points to data that is from before the *start_time* then SFXC will simply ignore all data before *start_time*. However, if the byte offset
points to data that is after *start_time* then SFXC will flag all data up that point (the frame which <BYTE OFFSET> points to) as as invalid for that station.

For example, it we wish to correlate the scan shown below (snippet from a $SCHED section) from a Mark5 unit with IP address 192.168.2.1, and the
data is stored on a diskpack with VSN *XAO+1020*.
```
scan No0013;
   start = 2023y154d18h21m57s;
   mode = K2048;
   station = Ur : 0 sec : 60 sec : 5524.088422400 GB :   : &n : 1;
   source = TARGET;
endscan;
```
then the *data_sources* would become
```
"data_sources": ["mk5://192.168.2.1/XAO+1020:5524088422400"]
```

Correlating data directly from the network requires jive5ab, which communicates with SFXC through a unix socket.
The URL format in *data_sources* is given by *mk5://<UNIX SOCKET>/<STREAM NAME>:0*, where
- UNIX SOCKET: The absolute file name of the unix socket
- STEAM NAME: Arbitrary string, the name is not used within SFXC
Note the ":0" at the end which is needed because the *mk5://* protocol specifier is used.

An example *data_sources* for a network correlation using jive5ab where the filename of the unix socket is "/tmp/mk5read-Wb":
```
data_sources": {
   "Wb": [
       "mk5:///tmp/mk5read-Wb/Wb-eVLBI:0"
   ]
```

## delay_directory \< String \> *(Optional, default = "file:///tmp/")*

SFXC stores its delay model in so-called delay files. There will a different delay file for each station in a experiment. This option controlls the directory where SFXC will store these delay files. Note that the directory name should be pre-pend with "file://".

The delays are only read by the SFXC manager node (MPI RANK 0), therefore the directory only needs to present on the physical machine where the manager node is run. If delay files don't already exist then SFXC will create delay files for all stations in the current correlation in this directory. Note that if the vexfile is updated after the delay files have been created, it may be nessecary to manually generate new delay files, e.g. using the gen_all_delay_tables.py script, for these changes to also be applied by SFXC.

Example where delays are stored in the directory /data/EXP/delays:
```
"delay_directory": "file:///data/EXP/delays",
```

### exit_on_empty_datastream \<boolean\> *(Optional, default=True*)

When set to true the correlation will abort if one of the paritipating station contains has no valid input data. Otherwise the correlation will continue
with this station flagged as invalid for the entire correlation.

## extra_delay \<dictionary\>

While, it is common that there is a slight difference in instumental delay between the various sub-bands, sometimes this difference is so large that it causes de-correlation. This option allows user to set an additional delay for some of the sub-bands. Because often this extra delay is between the two polarisations, it is also possible to specify an additional delay for a specific polarisation. Note that the unit of extra delay is seconds.

Example: add 2µs delay to RCP channels of station Kn, and -1µs to channels "CH02", and "CH03" for station Qq.
```
"extra_delay": {
     "Kn": {
         "R": 2e-06
     },
     "Qq": {
         "CH02": -1e-06,
         "CH03": -1e-06
     }
}
```

### fft_size_delaycor \<int\>,  *(Optional, default=256)*
The number of spectral channels used during delay correction, must be a power-of-two, and fft_size_delaycor <= fft_size_correlation.

### fft_size_correlation \<int\>,  *(Optional, default=max(fft_size_delaycor, nchan))*

The number of spectral channels used in cross-correlation, this must be a power-of-two, and fft_size_correlation >= fft_size_delaycor.
In multiple phase centre correlation, this option sets the spectral resolution at which the UV-shifting is performed, and therefore controls loss in S/R due to frequency smearing.

Example:
```
    "fft_size_delaycor" : 128,
    "fft_size_correlation": 256,
    "nchan": 32
```

### integr_time\<float\>

Integration time in seconds.

Example:
```
  "integr_time": 1.25
```

### sub_integr_time\<float\> *(Optional, default=min(20480, 1e6 x integr_time))*

Sub integration time in microseconds. When multiple phase centre correlation is active this option specifies the time to integrate before performing a uv shift.
This determines the amount of S/R loss due to time-averaging smearing. Note that we require that there are an integer number of FFTs of size *fft_size_correlation*
in one *sub_integr_time*.

If e.g. fft_size_correlation = 2048, and the sample rate for the *setup_station* is 32 MSamples / s, then one FFT is 2048 / 3.2e7 = 64 µs long. Which means that sub_integr_time needs
to be a multiple of 64, and therefore sub_integr_time = 12000 µs is **not** permissable, but sub_integr_time = 12032 µs is. NB: We do **not** require an integer number of sub-intergrations
in an integration.

Example, 2s integration time and 12.032ms sub-integrations.
```
    "integr_time": 2.0,
    "sub_integr_time: 12032,
```

### LO_offset \< Dict \>

Set *LO_offset* in Hz.

An error which sometimes happens at a station is that the local oscilator (LO) is tuned to the wrong frequency, which causes the sub-bands from that station to be offset in frequency relative to the other stations in the experiment. Because the sub-bands from a station with an LO offset will no longer perfectly overlap with the sub-bands from the other stations in the experiment, this will lead to some loss in data. However, the frequency ranges that do overlap can be recovered by applying a frequency shift equal to the LO offset.

Example, apply a 1MHz LO offset to station Qq:
```
"LO_offset": {
    "Qq": 1000000.0
}
```


### message_level \<int\> *(Optional, default = 1)*

Controls the amount of status information SFXC prints to the console. The current messege levels are:
- Message level 0: Suppresses most output.
- Message level 1, and 2: Print which correlation slices are currently scheduled on the correlator nodes. *(default)*
- Message level >= 3: Also print which MPI messages are send between the various nodes, only useful for debugging purposes.

Example:
```
"message_level": 1
```

### multi_phase_center \<Boolean\> *(Optional, default = True)*

Enable correlation of multiple phase centres. NB: If set to false, only the first source in a scan is correlated.
The feature is enabled by default if there is a scan containing multiple sources *in the schedule*.
Note that when the feature is enable then the output file names will have the source name appended to them, even for scans where there was only a single scan.

Example:
```
"multi_phase_center": true
```

### nchan \<int\>
Number of spectral channels per sub-band in the visibilities. The number of channels must be a power-of-two, and nchan <= fft_size_correlation. If nchan < fft_size_correlation, then the results are averaged down in frequency after correlation.

Example with 32 spectral channels per sub-band:
```
"nchan": 32
```

### normalize \< Boolean\> *(Optional, default=true)*

By default both auto- and cross-correlations are normalised using the total power in a band such that average amplitude of the auto-correlations are normalised to 1.0.
This option allows users to disable normalization.

Example:
```
"normalize": false
```

### scans \<List of strings\> *(Optional, default=All scans in the time range between start_time and stop_time)*

A list of scans to correlate, the names of the scans are defined in the $SCHED block in the VEX file.

For astronomical experiments it is usually not nessesary to supply a list of scans, the list of scans be uniquely deduced
from the schedule using the start- and stop times of the correlation. However, in geodetic VLBI it is common for different
scans to overlap in time, while also including some of the same stations. In these cases the start- and
stop times of the correlation don't always provide enough information to uniquely deduce which scans to correlate.
For these cases a list scans to correlate must be supplied.

NB: Providing a list of scans doesn't mean that they will be correlated in their entire, the start_time and stop_time options
still control the time range to correlate.

Example snippet from a $SCHED section of a VEX file
```
$SCHED;
scan No0001;
   start = 2024y056d12h00m00s;
   mode = C4096;
   station = Ef : 0 sec : 120 sec : 0.000000000 GB :   : &ccw : 1;
   station = Hh : 0 sec : 120 sec : 0.000000000 GB :   :   : 1;
   station = Wb : 0 sec : 120 sec : 0.000000000 GB :   :   : 1;
   source = 3C84;
endscan;
scan No0002;
   start = 2024y056d12h07m00s;
   mode = C4096;
   station = Ef : 0 sec : 600 sec : 0.000000000 GB :   : &ccw : 1;
   station = Hh : 0 sec : 600 sec : 0.000000000 GB :   :   : 1;
   station = Wb : 0 sec : 600 sec : 0.000000000 GB :   :   : 1;
   source = 3C84;
endscan;
scan No0003;
   start = 2024y056d12h22m00s;
   mode = C4096;
   station = Ef : 0 sec : 600 sec : 0.000000000 GB :   : &ccw : 1;
   station = Hh : 0 sec : 600 sec : 0.000000000 GB :   :   : 1;
   station = Wb : 0 sec : 600 sec : 0.000000000 GB :   :   : 1;
   source = 3C84;
endscan;
```

Example control file entry:
```
    "start_time": "2024y056d12h12m00s",
    "start_time": "2024y056d12h25m00s",
    "scans": ["No0002", "No0003"]
```
### output_file \<string\>

File name of output file, note that the filename should be preceded by a "file://" protocol specifier.
It is permissible to use a relative path for the filename, but note that this can lead to unexpected results if the current working directory doesn't exist on the machine running
the SFXC output node (MPI rank 2).

Example:
```
"output_file": "file:///data/out/output_file.cor"
```
### phasecal_file \<string\> *(Optional)*

File name where raw phasecal information is written to. The actual phase calibration tones can be extracted from this file using the phasecal.py script.

Example:
```
"phasecal_file": "file:///data/out/filename.pc"
```

### phasecal_integr_time \<float\> *(Optional)*

Time to integrat phase calibration tones before writing to disk in seconds, note that this time is independend of the *integr_time*

Example for a 1.0 second integration time:
```
"phasecal_integr_time": 1.0
```

### pulsar_binning \<Boolean\> *(Optional, default=False)*

Enables pulsar binning, note that only sources given in the "pulsars" parameter specified below are binned.

Example:
```
"pulsar_binning": true
```

### pulsars \<Dictionary\> *(Optional)*

Dictionary of pulsar to perform pulsar binning on when *pulsar_binning* is enabled.

An example *pulsars* block is shown below
```
"pulsars": {
    "PSR0329": {
        "nbins": 10,
        "polyco_file": "file://polyco_PSR0329.dat",
        "interval": [
            0.52,
            0.72
        ],
        "coherent_dedispersion": false
    }
}
```
Where "PSR0329" is the name of the source as given in the VEX file, the various parameters are
- nbins: The number of pulsar bins
- polyco_file: A geocentric tempo polyco file which is used to predict the pulsar phase
- interval: The binning interval in units of pulsar phase, e.g. a pulse phase of 0.5 is exactly half way in a rotation period. The pulse phase can also be > 1.0, e.g. "interval": [0.9, 1.1].
- "coherent_dedispersion": Enables coherent de-dispersion

### reference_station \<String\> *(Optional)*

Only correlate baselines to this station, can be useful for reducing data volume in tests.

Example where only baselines to station "Ef" are computed
```
"stations": ["Ef", "Jb", "On", "Wb"],
"reference_station": "Ef"
```

### slices_per_integration \<int\> *(Optional, default = 1)*

SFXC parallelizes the correlation in both time and frequency. Along the time axis the data is split into time slices such that each correlator node
will one such time slice at a time for a single sub-band. By default the length of each time slice is the same as the integration time.

The option slices_per_integration sets into how many time slices each integration should be split. These slices are correlated in parrallel with each other on different correlator nodes, reducing the latency before correlator output is produced is reduced. Furthermore, while this feature does not increase maximum throughput, it does improve the CPU utilization for shorter jobs.In cases where a large number of short jobs are run sequentially sliced integrations can significantly reduce total run time.

Note that all timeslices belonging to the same integration are summed on the output node, so setting this option does not alter the correlator output.

Example, divide each integration into 4 slices.
```
"slices_per_integration": 4
```

### start_time \<string\>

Start time of correlation. This time should be within one of the scans defined in the $SCHED block.

Times in SFXC are indicated using the vex date format which is given by YYYY**y**DDD**d**HH**h**MM**m**SS**s**, where,
  - YYYY = year
  - DDD = Day of year, NB: this is one-based so January 1 is day 001.
  - HH = Hour is 24h format, so 1pm = 13h
  - MM = Minute
  - SS = Second

Example, correlating data starting from 13:30 on the February 1 2025:
```
  "start_time": "2025y032d13h30m00s"
```

### stations \<List of strings\>

Two letter codes of stations to correlate, note that this is case sensitive.

Example:
```
  "stations": [
      "Cm", "Ef", "Hh", "Jb", "Ys"
    ]
```

### stop_time \<string\>

Time to stop correlating, same date format as is used for *start_time*.

Example:
```
  "stop_time": "2025y032d13h34m00s"
```

### tsys_file \<string\> *(Optional)*

If there is an 80HZ switch power amplitude calibration signal present in the data, then SFXC can extract this signal.
This parameter is the location where a raw output is generated which can then be further processed using the *tsys.py* script.

Example:
```
"tsys_file": "file:///data/out/filename.tsys"
```

### window_function \<String\> *(Optional, default=HANN)*

Select window to use, possible values are RECTANGULAR, COSINE, HAMMING, HANN, PFB, and NONE.

Before cross-correlation SFXC applies a window function to the data, whith consecutive window being half-overlapped.
Note that if NONE is selected, then the data is zero-padded.

---
_Content built by Aard Keimpema._ <i><span id="lastModified"></span></i>

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