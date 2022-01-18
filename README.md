# About 

The Section of Biostatistics at the University of Copenhagen has
access to a small linux computing cluster which we call
*biostat-ucph*. The cluster consists of two servers called **cox**
(official name: ```biostatcomp01fl.sund.root.ku.dk```) and **rao**
(official name: ```biostatcomp02fl.sund.root.ku.dk```). The cluster is
suitable for simulations and other parallel computing tasks.  In order
to avoid that multiple users disturb each others computations, all
computing tasks have to be started using the [Slurm Workload Manager](https://slurm.schedmd.com/documentation.html).

Since the documentation is quite rich, we below explain roughly how to
compute@biostat-ucph and provide examples.

Questions and help requests should be posed to those who have tried
before. You can also get write access to this github repository and
share your experience and examples.

# Important

It is *not* allowed to run R interactively on the servers *expect*
when you need to install R packages for subsequent use by jobs
submitted to the scheduler. You can do that by starting up R
interactively in a terminal on either cox and rao and installing the
packages as usual.

# Usage: compute@biostat-ucph

## Connect to server

Start *vpn* unless you are at work (CSS) and use a wired connection.
On linux (and Mac?) connect from terminal using **ssh**, e.g.,

```
ssh -X abc123@cox
```

where abc123 should be your KU id. On Windows, use **putty**.

## Getting comfortable

### Modules

The software on the servers needs to be enabled using the module terminal command.  
E.g., in order to use **R** you need to load the following 2 modules:

```
module load gcc/11.2.0
module load R/4.1.2
```

### R packages

Ensure that all R packages that your program needs are installed on
the server. To install a package you can start **R** interactively and
use =install.packages()= as usual. 

## Setup

You should have your computation task prepared:

1. A file with code that should be run repeatedly (in parallel) and everytime it is run does the following:
   * controls the random seed if necessary
   * reads in data, or simulates data
   * runs the analysis (important: each analysis should only use a single compuation core and not parallelize!) 
   * saves the results in a result file which should have a task specific filename
2. A terminal command-line command starting with =sbatch= or a bash script file which is then run from the terminal using =sbatch=.

## Running

The following commands are used to communicate with the Slurm scheduler and are probably the only ones that you will need.

* ```squeue``` - view information about jobs located in the Slurm scheduler
* ```scancel``` - cancel a job running on Slurm
* ```sbatch``` - submit a command to Slurm


## Examples

### An example of R
Consider a R program like the following where we generate some random
data and estimate a parameter

```{
x <- rnorm(1000)
mean(x)
}
```

In order to run this program 100 times in parallel with different random seeds we use
an R code file which we call =myScript.R= and which has the following
contents which consists of a header that controls the randomness and the R program:

```{
number_of_tasks <- as.numeric(Sys.getenv("SLURM_ARRAY_TASK_COUNT"))
task_id <- as.numeric(Sys.getenv("SLURM_ARRAY_TASK_ID"))

# Set a reproducible random seed for your simulations
set.seed(123457890)
# Generate a large number of random seeds to be assigned to each task
seeds <- sample(1:10^6, size = number_of_tasks, replace = FALSE)
# Assign the randomly generated seed to the current task
set.seed(seeds[task_id])

# ---------- Your R code goes between these lines ----------
x <- rnorm(1000)
result <- mean(x)
# -------------------------------------------------------------

# Save the results for this task as an individual file in the output folder
save(result, file = paste('output/result-', task_id, '.RData', sep = ""))
}
```
Then, run this from the command line with job name 'mySimulation':

```sbatch -a 1-100 -J 'mySimulation' R CMD BATCH myScript.R```

or use a bash script called for example =run-simulation.sh= which
specifies more options, see e.g., [slurm
examples](https://computing.sas.upenn.edu/gpc/job/slurm) for examples.

The bash script is then run from the command line as follows:

```sbatch run-simulation.sh```

# Misc

The file configuration.txt contains information on what has been
installed on the servers by Andreas KJ. 
