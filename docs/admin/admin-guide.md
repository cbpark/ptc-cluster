# Administrator Guide

This document is a guide only for the administrator. Normal users without the administrator's right have nothing to see here.

## Adding user

Suppose that the new user's ID is `alice` and its real name is _Alice Liddell_.

``` no-highlight
sudo wwuseradd alice
sudo passwd alice                      # set the password.
sudo usermod -c 'Alice Liddell' alice  # this is not mandatory,
                                       # but would be useful to identify the user.
```

If `alice` would be added to the group `usercl1`,

``` no-highlight
sudo usermod -aG usercl1 alice
```

Now check `/etc/passwd` and `/etc/group`:

``` no-highlight
grep alice /etc/passwd /etc/group
```

If everything is fine, synchronize the files to compute nodes.

``` no-highlight
sudo wwsh file resync
```

The synchronization might take several minutes. To check the status, run

``` no-highlight
sudo pdsh -w 'compute-0-[0-31] compute-1-[1-16]' grep alice /etc/passwd
```

A useful command to force the synchronization is

``` no-highlight
sudo pdsh -w 'compute-0-[0-31] compute-1-[1-16]' 'rm /tmp/.wwgetfiles_timestamp; SLEEPTIME=1 /warewulf/bin/wwgetfiles'
```

If the user reports some error messages concerning `nix` when logging in the master server, run

``` no-highlight
sudo su -
add-user-nix alice
exit
```

And, check whether the user directory has been created with correct permission. For example,

``` no-highlight
$ ls -ld /nix/var/nix/gcroots/per-user/alice
drwxr-xr-x 2 alice alice 6 Nov 21  2018 /nix/var/nix/gcroots/per-user/alice
```

To register the user to the `sacct`, run `sacctmgr`.

``` no-highlight
$ sudo sacctmgr show account
$ sudo sacctmgr show user -s
$ sudo sacctmgr add user alice Account=ctpu
```

## System-wide environment variables

See `/etc/profile` and files in `/etc/profile.d`.

## Slurm configurations

The main configuration file for the Slurm workload manager is `/etc/slurm/slurm.conf`. The official website of Slurm provides the generator of the configuration file. See [Configuration Tool](https://slurm.schedmd.com/configurator.html). To see the available options, see the man page:

``` no-highlight
man slurm.conf
```

## Rebooting a node

Suppose that the name of the node to reboot is `compute-node`. Before rebooting the node, we must tell Slurm that the node will be down.

``` no-highlight
sudo scontrol update nodename='compute-node' state=down reason='reboot'
```

`sinfo` shows that the status of the node has been set to be `down`.

``` no-highlight
$ \sinfo -l
PARTITION    AVAIL  TIMELIMIT   JOB_SIZE ROOT OVERSUBS     GROUPS  NODES       STATE NODELIST
espresso*       up      20:00        1-2   no       NO        all      1        down compute-node
```

But, the node has not been rebooted yet.

``` no-highlight
$ sudo pdsh -w 'compute-node' uptime
compute-node:  15:41:27 up 134 days,  4:26,  0 users,  load average: 0.00, 0.01, 0.05
```

We now reboot the node:

``` no-highlight
$ sudo pdsh -w 'compute-node' reboot
compute-node: Connection to compute-node closed by remote host.
$ sudo pdsh -w 'compute-node' uptime
compute-node: ssh: connect to host compute-node: No route to host
```

If the node has been successfully rebooted, `uptime` will show an output like:

``` no-highlight
$ sudo pdsh -w 'compute-node' uptime
compute-node:  15:45:15 up 1 min,  0 users,  load average: 0.22, 0.07, 0.03
```

The final step is to attach the node back to Slurm.

``` no-highlight
sudo scontrol update nodename='compute-node' state=resume
```

## Adding a compute node to Slurm

Suppose that the name of the compute node to add is `compute-node-new`. At first, check whether the node is listed in [Warewulf](http://warewulf.lbl.gov/) and running well.

``` no-highlight
$ sudo wwsh node list 'compute-node-new'
NAME                    GROUPS              IPADDR              HWADDR
=================================================================================
compute-node-new        UNDEF               10.1.1.nnn          aa:bb:cc:dd:ee:ff

$ sudo pdsh -w 'compute-node-new' uptime
compute-node-new:  11:51:06 up 29 min,  0 users,  load average: 0.00, 0.01, 0.05
```

Then, see the list of files in both the Warewulf database and the node.

``` no-highlight
$ sudo wwsh file list
cgroup.conf             :  rw-r--r-- 1   root root              236 /etc/slurm/cgroup.conf
dynamic_hosts           :  rw-r--r-- 0   root root             5124 /etc/hosts
group                   :  rw-r--r-- 1   root root             1526 /etc/group
ifcfg-ib0.ww            :  rw-r--r-- 1   root root              133 /etc/sysconfig/network-scripts/ifcfg-ib0
munge.key               :  r-------- 1   munge munge           1024 /etc/munge/munge.key
network                 :  rw-r--r-- 1   root root               20 /etc/sysconfig/network
passwd                  :  rw-r--r-- 1   root root             3782 /etc/passwd
shadow                  :  rw-r----- 1   root root             2732 /etc/shadow
slurm.conf              :  rw-r--r-- 1   root root             5842 /etc/slurm/slurm.conf

$ sudo wwsh provision print 'compute-node-new' | grep FILES
   compute-node-new: FILES            = cgroup.conf,dynamic_hosts,group,ifcfg-ib0.ww,munge.key,passwd,shadow,slurm.conf
```

We find in the above that `network` is missing in the node. (The missing file might be different case by case. If every file is already listed, we don't have to add the files.) Add the file to the new node and print the list again.

``` no-highlight
$ sudo wwsh provision set 'compute-new-node' --fileadd network
Are you sure you want to make the following changes to 1 node(s):

     ADD: FILES                = network

Yes/No> Yes

$ sudo wwsh provision print 'compute-new-node' | grep FILES
   compute-new-node: FILES            = cgroup.conf,dynamic_hosts,group,ifcfg-ib0.ww,munge.key,network,passwd,shadow,slurm.conf
```

We now add the node to Slurm. `slurmd` will show the hardware configuration.

``` no-highlight
$ sudo pdsh -w 'compute-new-node' slurmd -C
compute-new-node: NodeName=compute-new-node CPUs=48 Boards=1 SocketsPerBoard=2 CoresPerSocket=24 ThreadsPerCore=1 RealMemory=128444 TmpDisk=64222
compute-new-node: UpTime=0-01:55:51
```

Memorize the first line somewhere and add a line to `/etc/slurm/slurm.conf` as follows:

```
(...)

NodeName=compute-new-node      Procs=48 Sockets=2 CoresPerSocket=24 ThreadsPerCore=1 RealMemory=128444 Weight=1041 MemSpecLimit=4000 State=UNKNOWN

(...)
```

And, update the partitions:

```
(...)

PartitionName=espresso     Nodes=compute-new-node           MaxNodes=1   MaxCPUsPerNode=10 DefMemPerCPU=2000 MaxTime=00:20:00 State=UP Default=YES

(...)
```

Since `slurm.conf` has been updated, the file must be re-synced.

``` no-highlight
sudo wwsh file resync
```

The re-synchronization process will take a few or several minutes. To see whether it's been done in the compute nodes, run

``` no-highlight
sudo pdsh -w 'compute-0-[0-31] compute-1-[1-16]' grep NodeName=compute-new-node /etc/slurm/slurm.conf
```

When everything is fine, we restart the slurm daemons. In the master node, run

``` no-highlight
sudo systemctl restart slurmctld.service
sudo pdsh -w 'compute-0-[0-31] compute-1-[1-16]' systemctl restart slurmd
sudo systemctl restart slurmctld.service
```

If there is an error, check the systemd status and see the log message:

``` no-highlight
systemctl status slurmctld.service
sudo tail -n50 /var/log/slurmctld.log
```

## Viewing data for jobs

The `sacct` command displays accounting data for all jobs in the Slurm database. See `man sacct`. For example,

``` no-highlight
sudo sacct -n -X -o User%12,Start,End,Elapsed,AllocNodes,AllocCPUs,State -S 2018-11-20T12:00 -E 2018-11-20T18:00
```

will show all the jobs submitted from 12:00 to 18:00 on November 20, 2018. The command will display the username, the initialization time, the termination time, the elapsed time, the count of allocated CPUs, the number of nodes allocated to the job, and the job status. See `Job Accounting Fields` in `man sacct` or run `sacct --helpformat`.

To create a log file to upload into the IRIS system,

``` no-highlight
sudo SLURM_TIME_FORMAT="%F%t%R" sacct -n -X -o User%12,Start,End -S 2019-01-01T00:00 -E 2019-01-15T00:00:00 > ptc_log.csv
```

Note that `SLURM_TIME_FORMAT` follows the formats of the `strftime` function. See `man strftime`. In the above, `%R` yields the time in 24-hour notation (`%H-%M`).

## Printing out the reason of the DOWN state

``` no-highlight
$ scontrol show node $(sinfo | grep down | awk '{print $6}') | awk '/NodeName|Reason/{print}'
```

## Disable Hyperthreading using racadm

``` no-highlight
racadm --nocertwarn -r compute-node.ipmi -u USER -p PASSWORD get BIOS.ProcSettings.LogicalProc
racadm --nocertwarn -r compute-node.ipmi -u USER -p PASSWORD set BIOS.ProcSettings.LogicalProc Disabled
racadm --nocertwarn -r compute-node.ipmi -u USER -p PASSWORD get BIOS.ProcSettings.LogicalProc
```

Check the Key name. If it's `BIOS.Setup.1-1`, run

```
racadm --nocertwarn -r compute-node.ipmi -u USER -p PASSWORD jobqueue create BIOS.Setup.1-1 -r pwrcycle -s TIME_NOW -e TIME_NA
```

Then, reboot the node.
