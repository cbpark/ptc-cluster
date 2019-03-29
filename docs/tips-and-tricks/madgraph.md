# MadGraph

Since there are a lot of questions and requests in regard to run [MadGraph 5](https://launchpad.net/mg5amcnlo) (MadGraph5_aMC@NLO) in the cluster, the instruction is given here. The version of MadGraph used here is 2.6.1.

There are two ways to run MadGraph, _interactive execution_ and _batch processing_. Both are possible to run it in the cluster.

## Interactive execution

Recall that running hard computations on the master node is not allowed. Still, We can get an interactive command interface in the compute node by `srun`. Before going there, we need to load some [modules](../modules.md) to use MadGraph.

``` no-highlight
module load gnu7 MG5aMC_PY8_interface collier delphes
```

Then, check the list of currently loaded modules as

``` no-highlight
$ module list

Currently Loaded Modules:
  1) autotools        6) fastjet/3.3.0             11) collier/1.2
  2) prun/1.2         7) hepmc/2.06.09             12) python/2.7.14
  3) ohpc             8) lhapdf/6.2.1              13) root/6.12.06
  4) gnu7/7.2.0       9) pythia/8.2.35             14) delphes/3.4.1
  5) openmpi/1.10.7  10) MG5aMC_PY8_interface/1.0
```

We now jump to a compute node. The compute node will automatically be chosen by Slurm.

``` no-highlight
$ srun -p longlunch --pty bash
[cbpark@compute-0-0 MG5_aMC_v2_6_1]$
```

Here, `longlunch` is a name of the Slurm [partition](../job-scheduler.md).

``` no-highlight
$ scontrol show partition longlunch
PartitionName=longlunch
   AllowGroups=usercl1 AllowAccounts=ALL AllowQos=ALL
   AllocNodes=ALL Default=NO QoS=N/A
   DefaultTime=NONE DisableRootJobs=NO ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=20 MaxTime=03:00:00 MinNodes=1 LLN=NO MaxCPUsPerNode=40
   Nodes=compute-0-[0-13,16-24]
   PriorityJobFactor=1 PriorityTier=1 RootOnly=NO ReqResv=NO OverSubscribe=NO
   OverTimeLimit=NONE PreemptMode=OFF
   State=UP TotalCPUs=1088 TotalNodes=23 SelectTypeParameters=NONE
   DefMemPerCPU=2000 MaxMemPerNode=UNLIMITED
```

As we can see from the above, the `longlunch` partition is allowed only for users in the `usercl1` group (`AllowGroups=usercl1`). A user can become the member of the group by the system administrator. It can have 12 nodes for a job and the time limit is 3 hours (`MaxNodes=12 MaxTime=03:00:00`). We can choose any other partition that we like to use. By running the `srun` command with `--pty bash`, we are in an interactive command line in a compute node.

Now we modify some fields of `input/mg5_configuration.txt`.

``` no-highlight
pythia8_path = /opt/ohpc/pub/libs/gnu7/pythia/8.2.35

mg5amc_py8_interface_path = /opt/ohpc/pub/libs/gnu7/MG5aMC_PY8_interface/1.0

hepmc_path = /opt/ohpc/pub/libs/gnu7/hepmc/2.06.09

#! Default Running mode
#!  0: single machine/ 1: cluster / 2: multicore
run_mode = 1

#! Cluster Type [pbs|sge|condor|lsf|ge|slurm|htcaas|htcaas2] Use for cluster run only
#!  And cluster queue (or partition for slurm)
#!  And size of the cluster (some part of the code can adapt splitting accordingly)
cluster_type = slurm
cluster_queue = longlunch
cluster_size = 100

delphes_path = /opt/ohpc/pub/libs/gnu7/delphes/3.4.1/bin

lhapdf = lhapdf-config

fastjet = fastjet-config

collier = /opt/ohpc/pub/libs/gnu7/collier/1.2/lib
```

The path of each package can be found by `module show`. For example,

``` no-highlight
$ module show hepmc
---------------------------------------------------------------------------------
   /opt/ohpc/pub/moduledeps/gnu7/hepmc/2.06.09:
---------------------------------------------------------------------------------
whatis("Name: HepMC built with gnu7 toolchain ")
whatis("Version: 2.06.09 ")
whatis("Category: runtime library ")
whatis("Description: HepMC is a C++ Event Record for Monte Carlo Generators. ")
whatis("URL http://lcgapp.cern.ch/project/simu/HepMC/ ")
prepend_path("INCLUDE","/opt/ohpc/pub/libs/gnu7/hepmc/2.06.09/include")
prepend_path("LD_LIBRARY_PATH","/opt/ohpc/pub/libs/gnu7/hepmc/2.06.09/lib")
help([[
This module loads the HepMC library
built with the gnu7 compiler toolchain.

Version 2.06.09

]])
```

The path of HepMC is `/opt/ohpc/pub/libs/gnu7/hepmc/2.06.09` as can be deduced from the above. `lhapdf-config` and `fastjet-config` are already listed in our `PATH`.

``` no-highlight
$ which lhapdf-config
/opt/ohpc/pub/libs/gnu7/lhapdf/6.2.1/bin/lhapdf-config
$ which fastjet-config
/opt/ohpc/pub/libs/gnu7/fastjet/3.3.0/bin/fastjet-config
```

The most important part of the configuration file for running in the cluster is

``` no-highlight
run_mode = 1

cluster_type = slurm
cluster_queue = longlunch
cluster_size = 100
```

Here, we choose the cluster running mode (`run_mode = 1`) in the Slurm workload manager (`cluster_type = slurm`). The job will be submitted to the `longlunch` partition (`cluster_queue = longlunch`) using up to 100 cores (`cluster_size = 100`). Recall that the maximum allowed number of nodes is 12 and the maximum number of CPUs per node is 40 for a job submitted to the `longlunch` partition. In practice, MadGraph will choose the number of cores to use within the specified `cluster_size`.

After modifying the other fields if necessary, we run `mg5_aMC`.

``` no-highlight
[cbpark@compute-0-0 MG5_aMC_v2_6_1]$ ./bin/mg5_aMC
************************************************************
*                                                          *
*                     W E L C O M E to                     *
*              M A D G R A P H 5 _ a M C @ N L O           *
*                                                          *
*                                                          *
*                 *                       *                *
*                   *        * *        *                  *
*                     * * * * 5 * * * *                    *
*                   *        * *        *                  *
*                 *                       *                *
*                                                          *
*         VERSION 2.6.1                 2017-12-12         *
*                                                          *
*    The MadGraph5_aMC@NLO Development Team - Find us at   *
*    https://server06.fynu.ucl.ac.be/projects/madgraph     *
*                            and                           *
*            http://amcatnlo.web.cern.ch/amcatnlo/         *
*                                                          *
*               Type 'help' for in-line help.              *
*           Type 'tutorial' to learn how MG5 works         *
*    Type 'tutorial aMCatNLO' to learn how aMC@NLO works   *
*    Type 'tutorial MadLoop' to learn how MadLoop works    *
*                                                          *
************************************************************
load MG5 configuration from input/mg5_configuration.txt
set collier to /opt/ohpc/pub/libs/gnu7/collier/1.2/lib
set fastjet to fastjet-config
set lhapdf to lhapdf-config
No valid eps viewer found. Please set in ./input/mg5_configuration.txt
No valid web browser found. Please set in ./input/mg5_configuration.txt
Loading default model: sm
INFO: load particles
INFO: load vertices
INFO: Restrict model sm with file models/sm/restrict_default.dat .
INFO: Run "set stdout_level DEBUG" before import for more information.
INFO: Change particles name to pass to MG5 convention
Defined multiparticle p = g u c d s u~ c~ d~ s~
Defined multiparticle j = g u c d s u~ c~ d~ s~
Defined multiparticle l+ = e+ mu+
Defined multiparticle l- = e- mu-
Defined multiparticle vl = ve vm vt
Defined multiparticle vl~ = ve~ vm~ vt~
Defined multiparticle all = g u c d s u~ c~ d~ s~ a ve vm vt e- mu- ve~ vm~ vt~ e+ mu+ t b t~ b~ z w+ h w- ta- ta+
MG5_aMC>
```

We generate the SM top-pair process at the LHC beam condition and save the output to the `ttbar` directory, then launch the event generation. The event samples will be showered and hadronized by Pythia. The detector simulation will be performed by [Delphes](https://cp3.irmp.ucl.ac.be/projects/delphes) using the default card.

``` no-highlight
MG5_aMC> generate p p > t t~
MG5_aMC> output ttbar
MG5_aMC> launch
> shower=Pythia8
> detector=Delphes
> 0
> set nevents 100
> 0
```

Sometimes, we would see the harmless warning and error messages like

``` no-highlight
WARNING: cluster.get_job_identifier runs unexpectedly. This should be fine but report this message if you have problem.
** [################################################################] (100.00%)(0.00%)
Error in <TList::Clear>: A list is accessing an object (0x3df8be0) already deleted (list name = TList)
```

After successful running, we can return back to the master node.

``` no-highlight
[cbpark@compute-0-0 MG5_aMC_v2_6_1]$ exit
```

If we do not exit from the compute node, it will automatically be killed when the time limit has been reached.

## Batch processing

The interactive execution would be convenient when one wants to test a model or do a pilot research. But if we're going to produce a large number of event samples, the batch processing will be much easier and efficient.

At first, we modify `input/mg5_configuration.txt` as in the above. Then, we move to the MadGraph directory and create a script named `run_mg5_slurm.sh` (the name is arbitrary, of course) for batch processing as follows.

``` bash
#! /bin/bash -l
#
#SBATCH --job-name=ttbar_mg5_batch
#SBATCH --output=ttbar_mg5_batch_output.log
#SBATCH --partition=longlunch

module load gnu7 MG5aMC_PY8_interface collier delphes

./bin/mg5_aMC <<EOF
generate p p > t t~
output ttbar_batch
launch
  shower=Pythia8
  detector=Delphes
  0
  set nevents 100
  0
EOF
```

See [How to script MG5 run?](https://answers.launchpad.net/mg5amcnlo/+faq/2186) in the official FAQ of Madgrpah 5.

We submit the script to the job queue as

``` no-highlight
sbatch run_mg5_slurm.sh
```

and it's done. `squeue` shows that our job is running.

``` no-highlight
$ squeue
JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
  698 longlunch ttbar_mg   cbpark  RUNNING       5:21   3:00:00      1 compute-0-21
```

Since the standard output is written to `ttbar_mg5_batch_output.log` (`#SBATCH --output=ttbar_mg5_batch_output.log`), we can check the messages from the Madgraph while running.

``` no-highlight
tail -f ttbar_mg5_batch_output.log
```
