# Environment modules

The physics softwares and common libraries such as [GNU Scientific Library](https://www.gnu.org/software/gsl/) and [Boost](https://www.boost.org/) are provided through the [Environment Modules](http://modules.sourceforge.net/) system, or modules for short. The modules are regularly updated and maintained by the system administrator.

| ![xkcd - Compiling](https://imgs.xkcd.com/comics/compiling.png) |
| :--: |
| [xkcd](https://xkcd.com/), [*Compiling*](https://xkcd.com/303/) |

The command `module avail` shows all the available modules.

``` no-highlight
$ module avail

----------------------- /opt/ohpc/pub/moduledeps/gnu7-openmpi -----------------------
   boost/1.63.0    fftw/3.3.6    herwig/7.1.2    scipy/0.19.1    whizard/2.6.3

--------------------------- /opt/ohpc/pub/moduledeps/gnu7 ---------------------------
   ExRootAnalysis/1.1.4        lhapdf/6.2.1           pdtoolkit/3.24
   MG5aMC_PY8_interface/1.0    looptools/2.14         pythia/8.2.35
   ccfits/2.5                  metis/5.1.0            python/2.7.14  (D)
   cfitsio/3.430               mpich/3.2              python/3.4.6
   collier/1.2                 mvapich2/2.2           python/3.5.4
   delphes/3.4.1               numpy/1.13.1           python/3.6.4
   fastjet/3.3.0               ocaml/4.06.0           rivet/2.6.0
   golem/1.3.3                 ocr/1.0.1              root/6.12.06
   gsl/2.4                     openblas/0.2.20        sherpa/2.2.4
   hdf5/1.10.1                 openmpi/1.10.7  (L)    superlu/5.2.1
   hepmc/2.06.09               openmpi3/3.0.0         yoda/1.7.0b1

----------------------------- /opt/ohpc/pub/modulefiles -----------------------------
   autotools   (L)    gnu7/7.2.0   (L)    ohpc       (L)    prun/1.2        (L)
   cmake/3.9.2        hwloc/1.11.8        papi/5.5.1        singularity/2.4
   gnu/5.4.0          llvm5/5.0.0         pmix/1.2.3        valgrind/3.13.0

  Where:
   L:  Module is loaded
   D:  Default Module

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching any
of the "keys".
```

For example, if you want to use [ROOT](http://root.cern.ch/), run

``` no-highlight
module load root/6.12.06
```

The version number could be omitted if only one version is available or the default module is good enough to use. The list of loaded modules can be seen by

``` no-highlight
$ module list

Currently Loaded Modules:
  1) autotools   3) gnu7/7.2.0       5) ohpc            7) root/6.12.06
  2) prun/1.2    4) openmpi/1.10.7   6) python/2.7.14
```

The list in the above tells you that ROOT 6.12.06 and Python 2.7.14 are loaded in the current environment. Indeed, they are added to your `$PATH`.

``` no-highlight
$ which root
/opt/ohpc/pub/libs/gnu7/root/6.12.06/bin/root
$ which python
/opt/ohpc/pub/libs/gnu7/python-2.7.14/bin/python
```

`module show` prints out the commands in the module file. For example,

``` no-highlight
$ module show root
---------------------------------------------------------------------------------
   /opt/ohpc/pub/moduledeps/gnu7/root/6.12.06:
---------------------------------------------------------------------------------
whatis("Name: ROOT built with gnu7 toolchain ")
whatis("Version: 6.12.06 ")
whatis("Category: runtime library ")
whatis("Description: ROOT is a modular scientific software framework. ")
whatis("URL https://root.cern.ch ")
depends_on("python/2.7.14")
prepend_path("PATH","/opt/ohpc/pub/libs/gnu7/root/6.12.06/bin")
prepend_path("MANPATH","/opt/ohpc/pub/libs/gnu7/root/6.12.06/man")
prepend_path("INCLUDE","/opt/ohpc/pub/libs/gnu7/root/6.12.06/include")
prepend_path("LD_LIBRARY_PATH","/opt/ohpc/pub/libs/gnu7/root/6.12.06/lib")
prepend_path("PYTHONPATH","/opt/ohpc/pub/libs/gnu7/root/6.12.06/lib")
setenv("ROOTSYS","/opt/ohpc/pub/libs/gnu7/root/6.12.06")
help([[
This module loads the ROOT program and library
built with the gnu7 compiler toolchain.

Version 6.12.06

]])
```

This shows that `ROOTSYS` environment variable has been added. For a script to use ROOT given as below,

``` bash
#! /bin/bash -l
#
#SBATCH --job-name=test
#SBATCH --output=root_output.txt
#
#SBATCH --partition=espresso
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=100
#SBATCH --time=10:00

module load root
root -b -q $ROOTSYS/tutorials/math/binomial.C
```

running `sbatch run.sh` (`run.sh` is the name of the script given above) gives us

``` no-highlight
$ cat root_output.txt
   ------------------------------------------------------------
  | Welcome to ROOT 6.12/06                http://root.cern.ch |
  |                               (c) 1995-2017, The ROOT Team |
  | Built for linuxx8664gcc                                    |
  | From tag v6-12-06, 9 February 2018                         |
  | Try '.help', '.demo', '.license', '.credits', '.quit'/'.q' |
   ------------------------------------------------------------


Processing /opt/ohpc/pub/libs/gnu7/root/6.12.06/tutorials/math/binomial.C...

TMath::Binomial simple test
Build the Tartaglia triangle
============================
n= 0                             1
n= 1                           1   1
n= 2                         1   2   1
n= 3                       1   3   3   1
n= 4                     1   4   6   4   1
n= 5                   1   5  10  10   5   1
n= 6                 1   6  15  20  15   6   1
n= 7               1   7  21  35  35  21   7   1
n= 8             1   8  28  56  70  56  28   8   1
n= 9           1   9  36  84 126 126  84  36   9   1
n=10         1  10  45 120 210 252 210 120  45  10   1
n=11       1  11  55 165 330 462 462 330 165  55  11   1
n=12     1  12  66 220 495 792 924 792 495 220  66  12   1

TMath::Binomial fancy test
Verify Newton formula for (x+y)^n
x,y in [-2,2] and n from 0 to 9
=================================
Average Error = 1.064988e-13
```

Users of another shell like Zsh must initialize the module function before running `module`. See files in `$MODULESHOME/init`. For example, if you're using Z shell, the script in the above should be

``` bash
#! /bin/zsh -l
#
#SBATCH --job-name=test
#SBATCH --output=root_output.txt
#
#SBATCH --partition=espresso
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=100
#SBATCH --time=10:00

. $MODULESHOME/init/zsh
module load root
root -b -q $ROOTSYS/tutorials/math/binomial.C
```

Notice `/bin/zsh` in the first line and `. $MODULESHOME/init/zsh`.

`module unload` will unload the module.

``` no-highlight
$ module unload root
$ module list

Currently Loaded Modules:
  1) autotools   2) prun/1.2   3) gnu7/7.2.0   4) openmpi/1.10.7   5) ohpc
```

Note that Python 2.7.14 has been unloaded as well since it was loaded as a requirement to load ROOT. If what you want is just Python 2.7.14,

``` no-highlight
$ module load python/2.7.14
$ module list

Currently Loaded Modules:
  1) autotools   3) gnu7/7.2.0       5) ohpc
  2) prun/1.2    4) openmpi/1.10.7   6) python/2.7.14

$ which python
/opt/ohpc/pub/libs/gnu7/python-2.7.14/bin/python
```

`module avail` shows that you could use GCC 5.4.0 (`gnu/5.4.0`), while GCC 7.2.0 (`gnu7/7.2.0`) is currently loaded.

``` no-highlight
$ gcc --version
gcc (GCC) 7.2.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Since GCC 7.2.0 conflicts with GCC 5.4.0 (they cannot coexist in an environment), you cannot load directly GCC 5, but switch to it by running

``` no-highlight
$ module switch gnu7 gnu/5.4.0

Due to MODULEPATH changes, the following have been reloaded:
  1) python/2.7.14

The following have been reloaded with a version change:
  1) openmpi/1.10.7 => openmpi/1.10.6

$ module list

Currently Loaded Modules:
  1) autotools   3) ohpc        5) openmpi/1.10.6
  2) prun/1.2    4) gnu/5.4.0   6) python/2.7.14

$ gcc --version
gcc (GCC) 5.4.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

The list of loaded modules will be reset once you re-login the master server. All the modules can be removed from your environment by purging them.

``` no-highlight
$ module purge
$ module list
No modules loaded
```

Sometimes, it's necessary to remove `.lmod.d` in your home directory to refresh the list of available modules.

``` no-highlight
rm -rf ~/.lmod.d
module avail
```

`module --help` will show other useful commands.
