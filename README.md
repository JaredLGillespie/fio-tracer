# FIO Tracer
An I/O tracing utility based on [iobs](https://github.com/UOFL-CSL/iobs) which utilizes `fio` and `blktrace` to track the 
latencies of an IO across the Linux Kernel. This requires an additional 'S' tracepoint to be added, similar to what's 
done [here](https://github.com/uofl-csl/linux-nvme-s-tracepoint). The following Linux tools are utilized 
internally: `blktrace` and `fio`.

## Installation
The latest version can be obtained via `wget`:
```bash
$ wget https://raw.githubusercontent.com/uofl-csl/fio-tracer/master/fio-tracer.py
```
## Getting Started
Executing the script without any arguments shows the different arguments that can be used:
```bash
$ sudo python3 fio-tracer.py
fio-tracer.py 0.1.0
Usage: fio-tracer.py <file> [-c] [-l] [-o <output>] [-r <retry>] [-v] [-x]
Command Line Arguments:
<file>            : The configuration file to use.
-o <file          : (OPTIONAL) Output metrics to a csv file.
```

### Input
The main input to the tool should be a file in [INI](https://en.wikipedia.org/wiki/INI_file) format which has the configurations for how to run each of the workloads. The possible configuration settings for each workload is the following:
* fio_command [str]- The workload generation command to run (e.x. fio ...).
* fio_file [str] - The file fio will operate on.
* fio_lat_file_pref [str] - The latency file prefix that will hold the latencies generated by FIO.
* fio_ramp_time [str] - If ramp time is used, indicate here to skip IOs prior to this.
* blktrace_delay [int] - The amount of delay between running the workload and starting the trace. Defaults to 0.
* device [str] - The device to run the trace on (e.x. /dev/sdd).
* schedulers [str] - The io schedulers to use (e.x. noop, cfq, deadline, none, mq-deadline, bfq, kyber).
* runtime [int] - The amount of time in seconds to run the trace (should match the workload runtime).
* filter_stddev [float] - If given, filters IOs with latencies > stddev * filter_stddev.
* filter_abnormal [float] - If given, filters IOs which have a negative latency in the file system layer.


Each of these commands should be under a header [...] indicating a specific job. The global header [global] can be used for configuring settings that are the same for all jobs.

### Output
Upon completion of each job, the following metrics are output to STDOUT:
* CLAT (Completion Latency) [µs]
* SLAT (Submission Latency) [µs]
* Q2C [µs]
* Q2D (Block Layer Latency) [µs]
* D2S (Driver Latency) [µs]
* S2C (Device Latency) [µs]
* SLAT - Q2C (File System Latency) [µs]

The following is a sample output the is given for a job:
```bash
Results of workload:
clat [µs]: 30.80
slat [µs]: 8.02
q2c [µs]: 28.97
q2d [µs]: 10.35
d2c [µs]: 18.62
d2s [µs]: 2.30
s2c [µs]: 16.32
fs [µs]: 1.83
```

In addition to STDOUT, an output file can be given via the `[-o] < output>` command-line argument. This file will be written to as a csv with the first row being a header of the following columns:
* workload
* scheduler
* rw
* clat
* user
* block layer
* driver 
* device

## License
Copyright (c) 2018 UofL Computer Systems Lab. See [LICENSE](https://github.com/uofl-csl/fio-tracer/blob/master/LICENSE) for details.
