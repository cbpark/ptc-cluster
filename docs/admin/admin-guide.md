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
sudo pdsh -w 'compute-0-[0-22]' grep alice /etc/passwd
```

A useful command to force the synchronization is

``` no-highlight
sudo pdsh -w 'compute-0-[0-22]' 'rm /tmp/.wwgetfiles_timestamp; SLEEPTIME=1 /warewulf/bin/wwgetfiles'
```

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
