---
title: "OPTIONAL PRACTICAL: Benchmarking Molecular Dynamics Performance Using GROMACS"
teaching: 20
exercises: 60
questions:
- "How does a small, 80k-atom system performance scale as more cores are used?"
- "What about a larger, 12M-atom system?"
objectives:
- "Gain a basic understanding of key aspects of running molecular dynamics
simulations in parallel."
keypoints:
- "Larger systems scale better to large core-/node-counts than smaller systems."
---

## Aims

In this exercise, you will be running molecular dynamics simulations using
[GROMACS](https://manual.gromacs.org/). You will begin by benchmarking the
strong-scaling performance of an 80k-atom GROMACS simulation. You will also be
looking at the effects on performance that increasing the number of OMP thread
count has when running GROMACS on a single node. Finally, you can see how
multithreading and dynamic load balancing can impact performance.

## Measuring strong scaling

You  will be running a benchmark that has been used to explore the
performance/price behaviour  of  GROMACS on various  generations of CPUs and
GPUs. A number of publications have used this benchmark to report on, and
compare, the performance of GROMACS on different systems (*e.g* see:
[https://doi.org/10.1002/jcc.24030](https://doi.org/10.1002/jcc.24030)
).

The  benchmark system in question is “benchMEM”, which is available from the
list of Standard MD benchmarks at https://www.mpibpc.mpg.de/grubmueller/bench.
This benchmark simulates a membrane channel protein embedded in a lipid
bilayer surrounded by water and ions. With its size of ~80 000 atoms, it
serves as a prototypical example for a large class of setups used to study all
kinds of membrane-embedded proteins.For some more information see
[here](https://www.mpibpc.mpg.de/16460085/bench.pdf).

To get a copy of "benchMEM", run the following from your work directory:

```
wget https://www.mpibpc.mpg.de/15101317/benchMEM.zip
unzip benchMEM.zip
```
{: .language-bash}

Once the file is unzipped, you will need to create a Slurm submission script.
Below is a copy of a script that will run on a single node, using a single
processor. You can either copy the script from here, or from ARCHER2 in
directory `/work/ta017/shared/GMX_sub.slurm`.

```
#!/bin/bash

#SBATCH --job-name=GMX_test
#SBATCH --account=ta022
#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --reservation=shortqos
#SBATCH --time=0:5:0

#SBATCH --nodes=1
#SBATCH --tasks-per-node=1
#SBATCH --cpus-per-task=1

#SBATCH --distribution=block:block
#SBATCH --hint=nomultithread

module restore /etc/cray-pe.d/PrgEnv-gnu
module load gromacs

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun gmx_mpi mdrun -ntomp $SLURM_CPUS_PER_TASK -s benchMEM.tpr
```
{: .language-bash}

Run this script to make sure that it works -- how quickly does the job
complete? You can see the walltime and performance of your code by running
`tail md.log`. This simulation is meant to perform 10,000 steps, each of which
should simulate 2 ps of "real-life" time. The "Walltime" tells you how quickly
the job ran on ARCHER2, and the "Performance data will let you know how many
nanoseconds you can run in a day, and how many hours it will take to run a
nanosecond of simulation.

How do the "Walltime" and "Performance" data change as you increase the number
of cores being used? You can vary this by changing
`#SBATCH --tasks-per-node=1` to a higher number. Try filling out the table
 below:

 |Number of cores| Walltime | Performance (ns/day) | Performance (hours/ns) |
 |---------------|----------|----------------------|------------------------|
 |   1  | | | |
 |   2  | | | |
 |   4  | | | |
 |   8  | | | |
 |  16  | | | |
 |  32  | | | |
 |  64  | | | |
 | 128  | | | |
 | 256* | | | |
 | 512* | | | |

 ---
 **NOTE**

 Jobs run on more than one node will need to be run with constant
 `#SBATCH --tasks-per-node=128` but varying `#SBATCH --nodes=1`

 ---

## Measuring hybrid OpenMP + MPI performance on a single node

GROMACS can  run  in  parallel using  MPI  and  simultaneously  also  using
OpenMP  threads (see [here](http://manual.gromacs.org/current/user-guide/mdrun-performance.html#multi-level-parallelization-mpi-and-openmp)).
In this part of the tutorial, you will learn how to run hybrid MPI and OpenMP
jobs on ARCHER2, and you will benchmark the performance of the `benchMEM`
system to see whether performance improves when using OpenMP threads.

For this tutorial, you will start by comparing the performance of a simulation
that uses all of the cores on a node. Using following Slurm submission script
template, try running simulations that use varying levels of MPI tasks and
OpenMP threads. You can do this by changing the `#SBATCH --tasks-per-node` and
`#SBATCH --cpus-per-task` lines (making sure that the number of MPI ranks and
OpenMP threads always multiply to 128 or less).

```
#!/bin/bash

#SBATCH --job-name=GMX_test
#SBATCH --account=ta022
#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --reservation=shortqos
#SBATCH --time=0:5:0

#SBATCH --nodes=1
#SBATCH --tasks-per-node=128
#SBATCH --cpus-per-task=1

#SBATCH --distribution=block:block
#SBATCH --hint=nomultithread

module restore /etc/cray-pe.d/PrgEnv-gnu
module load gromacs

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun gmx_mpi mdrun -ntomp $SLURM_CPUS_PER_TASK -s benchMEM.tpr
```

How do the simulation times change as you increase the numbers change?
How do these times change if you do not spread the threads over the NUMA
regions as suggested?

You may find it helpful to fill out this table

| MPI Ranks | OpenMP threads | Walltime (s) | performance (ns/day) |
|-----------|----------------|--------------|----------------------|
|       128 |              1 | | |
|        64 |              2 | | |
|        42 |              3 | | |
|        32 |              4 | | |
|        25 |              5 | | |
|        16 |              8 | | |
|         8 |             16 | | |

## Multithreading and performance

The `--hint=nomultithread` asks SLURM to ignore the possibility of running
two threads per core. If we remove this option, this makes available 256
"cpus" per node (2 threads per core in hardware). To run 8 MPI tasks with
1 task per NUMA region running 32 OpenMP threads, the script would look like:

```
#!/usr/bin/env bash

#SBATCH --job-name=GMX_test
#SBATCH --account=ta022
#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --reservation=shortqos
#SBATCH --time=0:5:0

#SBATCH --nodes=1
#SBATCH --tasks-per-node=128
#SBATCH --cpus-per-task=1

#SBATCH --hint=multithread
#SBATCH --distribution=block:block

module load epcc-job-env
module load xthi/1.0

export OMP_PLACES=cores
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun gmx_mpi mdrun -ntomp $SLURM_CPUS_PER_TASK -s benchMEM.tpr
```
Note: physical cores appear as affinity 0-127, while the extra "logical"
cores are numbered 128-255. Logical cores 0 and 128 occupy the same physical
core etc.

> ## Multithreading and GROMACS?
>
> Staring with the MPI-only case first, how does enabling multithreading
> affect GROMACS performance?
>
> What about the performance of hybrid MPI+OpenMP jobs?
{: .challenge}

## Load balancing

GROMACS performs dynamic load balancing when it deems necessary. Can you tell
from your md.log files so far whether it has been doing so, and what it
calculated the load imbalance was before deciding to do so?

To demonstrate the effect of the load imbalance counteracted by GROMACS’s
dynamic load balancing scheme, investigate what happens when this is turned
off by including the `-dlb no` option to `gmx_mpi mdrun`.

{% include links.md %}
