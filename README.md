# Running Stata on the HPC Clusters

This guide presents an overview of running Stata on the HPC clusters at Princeton.

## Run Stata in Your Web Browser

If you are new to high-performance computing then you will find that the simplest way to use Stata on the HPC clusters is through the Open OnDemand web interface. If you have an account on Adroit or Della
then browse to [https://myadroit.princeton.edu](https://myadroit.princeton.edu) or [https://mydella.princeton.edu](https://mydella.princeton.edu). If you need an account on Adroit then complete [this form](https://forms.rc.princeton.edu/registration/?q=adroit).

To begin a session, click on "Interactive Apps" and then "XStata". You will need to choose the "Stata version", "Number of hours" and "Number of cores". Set "Number of cores" to 1 unless you are sure that your script has been explicitly
parallelized using, for example, the Parallel Computing Toolbox (see below). Click "Launch" and then when your session is ready click "Launch MATLAB". Note that the more resources you request, the more you will have to wait for your session to become available.

![jupyter](https://tigress-web.princeton.edu/~jdh4/stata_two_frames.png)

## Submitting Batch Jobs to the Slurm Scheduler

Intermediate and advanced MATLAB users prefer submitting jobs to the Slurm scheduler over using the web interface (described above). A job consists of a MATLAB script and a Slurm script that specifies the needed resources and the commands to be run.

### Running a Serial Stata Job

A serial Stata job is one that requires only a single CPU-core. Here is an example of a trivial, one-line serial MATLAB script (`hello_world.do`):

```stata
disp 21+21
```

The Slurm script (`job.slurm`) below can be used for serial jobs:

```bash
#!/bin/bash
#SBATCH --job-name=stata         # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G per cpu-core is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)

module purge
module load stata/16.0

stata -b hello_world.do
```

To run the Stata script, simply submit the job to the cluster with the following command:

```
$ sbatch job.slurm
```

After the job completes, view the output with `cat hello_world.log`:

```
 /__    /   ____/   /   ____/
___/   /   /___/   /   /___/   16.0   Copyright 1985-2019 StataCorp LLC
  Statistics/Data Analysis            StataCorp
                                      4905 Lakeway Drive
                                      College Station, Texas 77845 USA
                                      800-STATA-PC        http://www.stata.com
                                      979-696-4600        stata@stata.com
                                      979-696-4601 (fax)

100-user Stata network perpetual license:
       Serial number:  401606267559
         Licensed to:  Stata/SE 16
                       100-user Network

Notes:
      1.  Stata is running in batch mode.
      2.  Unicode is supported; see help unicode_advice.

. do "hello_world.do" 

. display 21+21
42

. 
end of do-file
```

Use `squeue -u $USER` to monitor the progress of queued jobs.

### Choosing a Stata version

Run the command below to see the available Stata versions. For example, on Della:

```
$ module avail stata
----------------------------- /usr/licensed/Module s/modulefiles -----------------------------
matlab/R2010a      matlab/R2012a      matlab/R2014b      matlab/R2016b            matlab/R2018b
matlab/R2010b      matlab/R2013a      matlab/R2015a      matlab/R2017a            matlab/R2019a
matlab/R2011a      matlab/R2013b      matlab/R2015b      matlab/R2017b(default)   matlab/R2019b
matlab/R2011b      matlab/R2014a      matlab/R2016a      matlab/R2018a
```

In your Slurm script, you can either take the default version with `module load stata` or choose a specific version, for example: `module load stata/16.0`.

### Running a Multi-threaded Stata Job with the Parallel Computing Toolbox

Most of the time, running MATLAB in single-threaded mode (as described above) will meet your needs. However, if your code makes use of the Parallel Computing Toolbox (e.g., `parfor`) or you have intense computations that can benefit from the built-in multi-threading provided by MATLAB's BLAS implementation, then you can run in multi-threaded mode. One can use up to all the CPU-cores on a single node in this mode. **Multi-node jobs are not possible with the version of MATLAB that we have so your Slurm script should always use `#SBATCH --nodes=1`.**

Here is an [example](https://www.mathworks.com/help/parallel-computing/interactively-run-a-loop-in-parallel.html) from MathWorks of using multiple cores (`for_loop.m`):

```stata
# stata-mp source code
```

The Slurm script (`job.slurm`) below can be used for this case:

```bash
#!/bin/bash
#SBATCH --job-name=parfor        # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=4        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G is default)
#SBATCH --time=00:00:30          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=all          # send email on job start, end and fault
#SBATCH --mail-user=<YourNetID>@princeton.edu

module purge
module load stata/16.0

stata-mp -b hello_world.do
```

One must tune the value of `--cpus-per-task` for optimum performance. Use the smallest value that gives you a significant performance boost because the more resources you request the longer your queue time will be.

## Running Stata on Nobel

The Nobel cluster is a shared system without a job scheduler. Because of this, users are not allowed to run MATLAB in multi-threaded mode. The first step in using MATLAB on Nobel is choosing the version. Run `module avail matlab` to see the choices. Load a module with, e.g., `module load matlab/R2019b`.

After loading a MATLAB module, to run MATLAB interactively with its GUI on the script `myscript.m`:

```
$ matlab -singleCompThread -r myscript
```

To run MATLAB without its GUI in command-line mode:

```
$ matlab -singleCompThread -nodisplay -nosplash -r myscript
```

## Using the MATLAB GUI on Tigressdata

In addition to the web interfaces on MyAdroit and MyDella, one can also launch MATLAB with its GUI on Tigressdata. Tigressdata is ideal for data post-processing and visualization. You can access your files on the different filesystems using these paths: `/tiger/scratch/gpfs/<YourNetID>`, `/della/scratch/gpfs/<YourNetID>`, `/perseus/scratch/gpfs/<YourNetID>`, `/tigress` and `/projects`.

Mac users will need to have [XQuartz](https://www.xquartz.org/) installed while Windows users should install [MobaXterm](https://mobaxterm.mobatek.net/download.html) (Home Edition). Visit the the [OIT Tech Clinic](https://princeton.service-now.com/snap/?id=kb_article&sys_id=ea2a27064f9ca20018ddd48e5210c771) for assistance with installing, configuring and using these tools.

To run MATLAB interactively with its graphical user interface:

```
$ ssh -X <YourNetID>@tigressdata.princeton.edu
$ module load matlab/R2019a
$ matlab
```

It can take a minute or more for the GUI to appear and for initialization to complete.

To work interactively without the GUI:

```
$ ssh <YourNetID>@tigressdata.princeton.edu
$ module load matlab/R2019a
$ matlab
>>
```

Note that one can use the procedures above on the HPC clusters (e.g., Della) but only for non-intensive work since the head node is shared by all users of the cluster.

## Getting Help from Data & Statistical Services (DSS)

For help on using Stata with data analysis please see the [DSS website](https://dss.princeton.edu). DSS offers online [tutorials](https://dss.princeton.edu/online_help/stats_packages/r/) and [training](https://dss.princeton.edu/training/) for performing data analysis with stata as well as one-on-one appointments.

## Getting Help from Research Computing

If you encounter any difficulties while working with Stata on the HPC clusters then please
send an email to <a href="mailto:cses@princeton.edu">cses@princeton.edu</a> or attend a <a href="https://researchcomputing.princeton.edu/education/help-sessions">help session</a>.
