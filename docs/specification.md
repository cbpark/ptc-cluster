# Specification and SSH connection

The PTC cluster consists of 1 **master node** and 21 **compute nodes** now. There is no login server and the SSH connection to `ptc.ibs.re.kr` directs to the master node. All the machines are in the data center located on the ground floor of the experiment building in the IBS main site.

| ![xkcd - Datacenter scale](https://imgs.xkcd.com/comics/datacenter_scale.png) |
| :--: |
| [xkcd](https://xkcd.com/), [*Datacenter Scale*](https://xkcd.com/1737/) |

## Hardware

### CPU and memory

The master node has 1.6 GHz 6-core Intel Xeon CPU and 16 GB memory with 32 GB swap space. Note that the master node is not intended for hard computations or tasks, but for distributing and logging computing jobs to compute nodes. In most cases, you will run commands at the master node and the real computations will be performed by the compute node.

Each compute node has quite a number of CPU cores, 24 to 72 cores with clock speeds of 2.2 GHz to 2.6 GHz. The total accumulated number of CPU cores of all the compute node is 952. The hyper-threading of CPU is disabled for all nodes. Each node has more than 128 GB memory.

To see the detailed information of the compute nodes, run

``` no-highlight
scontrol show nodes
```

### Storage

The size of disk partition for the home directory is 22 TB. Moreover, 18 TB and 34 TB storage disks are attached and shared via the local network. They are mounted as `/data` and `/bigdata`, respectively. The disk space will be available upon request of the user. Due to a technical reason, `/bigdata` is not mounted on the compute nodes.

Currently, the disk quota has not been set. Use `/data` or `/bigdata` if you need a space for storing data for a long time.

## Software

The operating system of all the nodes is [CentOS](https://www.centos.org/) 7.4, and the installed softwares include

* [coreutils](https://www.gnu.org/software/coreutils/coreutils.html) 8.22,
* [binutils](http://sources.redhat.com/binutils) 2.25.1,
* [GCC](https://gcc.gnu.org/) 4.8.5,
* [GNU C Library](https://www.gnu.org/software/libc/) 2.17,
* [Python](http://python.org/) 2.7.5,
* [Perl](http://www.perl.org/) 5.16.3,
* [Gnu Emacs](https://www.gnu.org/software/emacs/) 24.3.1.

You can list the installed packages by running:

``` no-highlight
yum list installed
```

The physics softwares such as [ROOT](http://root.cern.ch/) and [Pythia](http://home.thep.lu.se/Pythia/) are provided through [Environment Modules](http://modules.sourceforge.net/). See the page of [Environment modules](modules.md). The more recent GCC's are also available using modules.

Note that Mathematica is not installed and it will never be since we have a dedicated workstation server for that. And, the Fortran 77 compilers such as `g77` are not supported any longer. Update your codes or contact the developer so that they can be compiled by modern Fortran compilers. The Intel C++ and Fortran compilers are not installed as they are not free software and our budget is limited.

## SSH connection

Users can connect to the master node via SSH with port 22:

``` no-highlight
ssh userid@ptc.ibs.re.kr
```

If a graphical user interface is necessary, it can be achieved using X11 forwarding.

``` no-highlight
ssh -X userid@ptc.ibs.re.kr
```

The machines assigned an IP address of the external network of the IBS building, starting with 10.10.24, can directly connect to the server. Meanwhile, the laptops or PCs using the wireless network (eduroam) in the IBS building as well as user's home network cannot connect directly without SSL VPN. The IT team of the IBS headquarter will provide the SSL VPN client for Linux, macOS, and Windows OS.

The SSH connection from the master node to compute nodes are not allowed unless there are active jobs on the nodes. See the `srun --pty bash` command in the [job scheduler](job-scheduler.md).

Files can be transferred to the server by using `sftp`, `scp`, or `rsync`.

If you see a message like

``` no-highlight
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

when connecting to the server, remove the line beginning with `ptc.ibs.re.kr` in `.ssh/known_hosts` in your home directory.
