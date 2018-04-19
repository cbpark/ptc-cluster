# Job scheduler

A job scheduler or workload automation is necessary for efficient handling of the computing resources. It is essential and the most important application in order to run your jobs on the compute nodes, so every user must know how to use it. Tasks without passing through the job scheduler will be killed without notice.

As a job scheduler of the PTC cluster, [Slurm](https://slurm.schedmd.com/) is employed. The installed version is 17.02.9. If you are new to Slurm, the official [Quick Start User Guide](https://slurm.schedmd.com/quickstart.html) is the best place to start learning about it. Here is a basic instruction for using Slurm in the PTC cluster.

The users of the old PTC cluster system might be familiar with Sun Grid Engine (SGE). Please refer to [SGE to SLURM conversion](https://srcc.stanford.edu/sge-slurm-conversion) and modify the scripts.

A summary of commands and options of Slurm is given [here](https://slurm.schedmd.com/pdfs/summary.pdf).

## Partition

The partitions group compute nodes into logical sets. Each partition has its own job size limit, job time limit, and user group permitted to use it. It is sometimes called _job queues_. The `sinfo` command reports the state of partitions.

``` no-highlight
$ sinfo
PARTITION    AVAIL  TIMELIMIT JOB_SIZE MAX_CPUS_PER_NODE NODES(A/I/O/T)    CPUS(A/I/O/T)
espresso*       up      20:00      1-2                10      0/21/0/21      0/952/0/952
```

The name of the partition shown in the above is `espresso`, which is up and running. `*` denotes that it is the default partition. The `espresso` partition will be used if you do not specify a partition to use. The job time limit (`TIMELIMIT`) is set to be 20 minutes. Jobs running beyond the time limit will be automatically killed. A job submitted to the `espresso` partition can use two nodes at most (`JOB_SIZE`), and the maximum number of CPUs per node (`MAX_CPUS_PER_NODE`) is 10. Thus, the job in the `espresso` partition can use concurrently up to 2 * 10 = 20 CPUs. The last two fields in the above show the number of nodes by a state in the format "allocated/idle/other/total" (A/I/O/T) and the number of CPUs in the same format.

Another useful command is `scontrol show partition`.

``` no-highlight
$ scontrol show partition espresso
PartitionName=espresso
   AllowGroups=ALL AllowAccounts=ALL AllowQos=ALL
   AllocNodes=ALL Default=YES QoS=N/A
   DefaultTime=NONE DisableRootJobs=NO ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=2 MaxTime=00:20:00 MinNodes=1 LLN=NO MaxCPUsPerNode=10
   Nodes=compute-0-[0-20]
   PriorityJobFactor=1 PriorityTier=1 RootOnly=NO ReqResv=NO OverSubscribe=FORCE:4
   OverTimeLimit=NONE PreemptMode=OFF
   State=UP TotalCPUs=952 TotalNodes=21 SelectTypeParameters=NONE
   DefMemPerCPU=2000 MaxMemPerNode=20000
```

Note again `MaxNodes=2 MaxTime=00:20:00`. `AllowGroups=ALL` means that any user can run jobs in the partition. `scontrol show partitions` shows the detailed information of all the partitions where you can submit jobs.

A new user can use only the `espresso` partition. The other partitions with a longer time limit and larger job size will be available when the user is qualified to have enough knowledge and skill to use the cluster. It recommended that doing test and exercise in the `espresso` partition repeatedly before using the other partitions.

The `squeue` command shows the status of running jobs.

``` no-highlight
$ squeue
JOBID PARTITION     NAME     USER    STATE       TIME   TIME_LIMIT  NODES
  317 microcent my.scrip   cbpark COMPLETI       0:50         1:00      1
  315 fortnight      zsh   cbpark  RUNNING       5:26  14-00:00:00      1
  318 microcent my2.scri   cbpark  RUNNING       0:39         1:00      3
```

`scontrol show job` can be used to see the detail of the submitted job.

``` no-highlight
$ scontrol show job 315
JobId=315 JobName=zsh
   UserId=cbpark(1001) GroupId=cbpark(1001) MCS_label=N/A
   Priority=4294901729 Nice=0 Account=(null) QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:10:32 TimeLimit=14-00:00:00 TimeMin=N/A
   SubmitTime=2018-04-09T15:11:12 EligibleTime=2018-04-09T15:11:12
   StartTime=2018-04-09T15:11:12 EndTime=2018-04-23T15:11:12 Deadline=N/A
   PreemptTime=None SuspendTime=None SecsPreSuspend=0
   Partition=fortnight AllocNode:Sid=ptc:35960
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=compute-0-11
   BatchHost=compute-0-11
   NumNodes=1 NumCPUs=40 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   TRES=cpu=40,mem=80000M,node=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=1 MinMemoryCPU=2000M MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   Gres=(null) Reservation=(null)
   OverSubscribe=YES Contiguous=0 Licenses=(null) Network=(null)
   Command=zsh
   WorkDir=/home/cbpark
   Power=
```

## Job submission

The simplest command to run tasks on the compute nodes is `srun`.

``` no-highlight
$ srun -N 2 -n 5 -p espresso /bin/hostname
compute-0-1
compute-0-0
compute-0-1
compute-0-0
compute-0-0
```

The above task print the hostname of the compute nodes in the `espresso` partition (`-p espresso`) using 5 CPUs (`-n 5`) of 2 nodes (`-N 2`). Note that the maximum number of nodes per task in the `espresso` partition is set to be 2. If you set a larger number, the job will not run and await more resources.

``` no-highlight
$ srun -N 5 -n 10 -p espresso /bin/hostname
srun: Requested partition configuration not available now
srun: job 330 queued and waiting for resources
```

``` no-highlight
$ squeue
JOBID PARTITION     NAME     USER    STATE       TIME   TIME_LIMIT  NODES  NODELIST(REASON)
  330  espresso hostname   cbpark  PENDING       0:00        20:00      5  (PartitionNodeLimit)
```

The job can be deleted by running `scancel 330` (see `JOBID`). To cancel all the jobs of yours,

``` no-highlight
scancel -u <username>
```

where `<username>` is your user ID.

Complicated operations can be done by submitting a script, which is reusable for later execution. Let's say the script is `run.sh`.

``` bash
#! /bin/bash -l
#
#SBATCH --job-name=test
#SBATCH --output=output.txt
#
#SBATCH --partition=espresso
#SBATCH --ntasks=10
#SBATCH --nodes=2
#SBATCH --mem-per-cpu=100
#SBATCH --time=10:00
#
#SBATCH --mail-type=END,FAIL,TIME_LIMIT
#SBATCH --mail-user=cbpark@ibs.re.kr

srun echo 'Greetings from' $(/bin/hostname)
srun sleep 60
```

The above script prints `Hello, world! from hostname` 10 times (`--ntaskes=10`) to `output.txt` (`--output=output.txt`) using two nodes (`--nodes=2`) and 100 MB per CPU (`--mem-per-cpu=100`). The time limit has been set to be 10 minutes (`--time=10:00`). The user will receive an email notification when the job has been ended. Note that lines beginning with `#SBATCH` are not comments. `#SBATCH` is a prefix to set options. If you want to comment out the line, attach one more `#`, i.e., `##SBATCH`. `-l` option in the first line means that the shell acts as if it had been invoked as a login shell.

You'll get an email when the job is finished if you specified `--mail-type` and `--mail-user`.

Before submitting the script to the job scheduler, it's advisable to validate the script.

``` no-highlight
$ sbatch --test-only run.sh
sbatch: Job 564 to start at 2018-04-12 00:20:54 using 80 processors on compute-0-[0-1]
```

To actually submit the script to the job scheduler, run

``` no-highlight
sbatch run.sh
```

See `man sbatch` or the [sbatch](https://slurm.schedmd.com/sbatch.html) page on the official website for available options. The options that might be useful are `--deadline`, `--workdir`, `--mem`, `--ntasks-per-node`, and so on.

If you want to cancel a running job and resubmit it, memorize the job id and run

``` no-highlight
scontrol requeue <jobid>
```

Sometimes, you want interactive operations for a job.

``` no-highlight
srun -n 10 -p espresso --pty bash
```

will put your command prompt to a compute node. This is similar to the `qrsh` command of SGE. You can return back to the master node by the `exit` command. If you're using another shell such as [Zsh](https://www.zsh.org/), add `--pty zsh` instead of `--pty bash`.

### OpenMP

[OpenMP](http://www.openmp.org/) supports multiprocessing programming by implementing multithreading which runs concurrently with the runtime environment allocating threads to different processors. (Do not confuse it with [Open MPI](https://www.open-mpi.org/).) Note that you should make sure that all the CPU cores you request are on the same node. Below is an example script using OpenMP.

``` bash
#! /bin/bash -l
#
#SBATCH --job-name=omp_test
#SBATCH --output=omp_test.log
#
#SBATCH --partition=espresso
#SBATCH --nodes=1            # number of nodes
#SBATCH --cpus-per-task=8    # number of threads
#SBATCH --mem-per-cpu=100    # memory per cpu

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun -c $SLURM_CPUS_PER_TASK ./MYPROGRAM
```

Here `MYPROGRAM` is the executable you will run on the compute node. The most important part is `--nodes=1` and `--cpus-per-task=8`. The latter tells the job scheduler how many threads you intend to run with. Unless the number of nodes is 1, the job could be distributed over many nodes, leading to poor performance.

## Deadly commands

| ![xkcd - Admin Mourning](https://imgs.xkcd.com/comics/admin_mourning.png) |
| :--: |
| [xkcd](https://xkcd.com/), [*Admin Mourning*](https://xkcd.com/686/) |

* `nohup`

You might have heard of the `nohup` command, and moreover, might love to use it. It makes a command ignoring hangup signals, for instance, when you want to have the command running even after you close the terminal. It would never be harmful if you're using it on your standalone machine. However, all the tasks running in the cluster should be recognized by the job scheduler since the job scheduler will allocate resources and gracefully close the tasks. The `nohup` command sometimes deceives the job scheduler and keeps the processes running in the background even after the submitted job had been completed. The node can eventually be frozen due to fulling up of resources such as memory and disk space. Do not use `nohup` in the cluster.
