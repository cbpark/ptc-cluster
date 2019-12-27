# Jupyter Notebook

[Jupyter](https://jupyter.org/) Notebook (formerly known as IPython Notebook) is a web-based interactive computational environment for creating Jupyter notebook documents. Due to the web-based environment, it would be tricky to run it in the cluster. Here is the instruction to run Jupyter Notebook through [ssh tunneling](https://www.ssh.com/ssh/tunneling).

## Virtual environment

At first, it is recommended to use an isolated environments in which you can install python packages without interfering with the system. Here [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is used, although there are several other ways to manage virtual environments. If you're already using a virtual environment, skip to the next step.

Add the lines in the below to your shell configuration file such as `.bash_profile`:

``` bash
export WORKON_HOME=$HOME/.virtualenvs
source /usr/bin/virtualenvwrapper_lazy.sh
```

By running `source ~/.bash_profile`, you are ready to use the virtualenvwrapper. Now, load a python [module](../modules.md) into your environment. For example,

``` no-highlight
$ module load python/3.6.4
$ which python3
/opt/ohpc/pub/libs/gnu7/python-3.6.4/bin/python3
```

It's time to create a virtual environment. Suppose that the name of the environment is `jupyter`.

``` no-highlight
$ mkvirtualenv -p $(which python3) jupyter
mkdir: created directory ‘/home/cbpark/.virtualenvs’
(...)
Installing setuptools, pip, wheel...done.
```

You can _activate_ the virtual environment:

``` no-highlight
$ workon jupyter
```

To deactivate the environment, run

``` no-highlight
$ deactivate
```

## Installing and running jupyter

After activating the virtual environment, make sure that `.virtualenv` has been added to the `PATH`:

``` no-highlight
$ workon jupyter
$ echo $PATH
/home/cbpark/.virtualenvs/jupyter/bin:/opt/ohpc/pub/libs/gnu7/python-3.6.4/bin:(...)
```

Install jupyter using [pip](https://pypi.org/project/pip/). The packages will be installed in `~/.virtualenvs/jupyter`. The whole installation may take several minutes.

``` no-highlight
$ pip install --upgrade jupyter
Collecting jupyter
(...)
Successfully installed MarkupSafe-1.1.1 Send2Trash-1.5.0 (...)

$ which jupyter
~/.virtualenvs/jupyter/bin/jupyter
```

Before running the jupyter notebook, set a password for the notebook:

``` no-highlight
$ mkdir -p ~/.jupyter
$ jupyter notebook password
Enter password:
Verify password:
[NotebookPasswordApp] Wrote hashed password to /home/cbpark/.jupyter/jupyter_notebook_config.json
```

Before running the jupyter notebook, we open an interactive job session of Slurm:

``` no-highlight
$ srun -p workday -J "jupyter" -c 10 --pty /bin/bash
$ hostname
compute-0-25
```

Here, the job name is `jupyter` and we will use 10 CPU cores for the job. The hostname of the allocated compute node is `compute-0-25`. Memorize the hostname for the next step!

Now you can run the jupyter notebook:

``` no-highlight
$ workon jupyter
$ jupyter notebook --no-browser --port=8888
```

Do not forget to add the `--no-browser` option, because otherwise, it will automatically open the web browser in the server. You may choose a port other than `8888`.

## SSH tunneling

To open the jupyter notebook in the web browser of your desktop or laptop, we use ssh tunneling.

``` no-highlight
$ ssh -t cbpark@ptc.ibs.re.kr -L 8888:localhost:8888 ssh compute-0-25 -L 8888:localhost:8888
cbpark@ptc.ibs.re.kr's password:
cbpark@compute-0-25's password:
```

Customize the user ID and hostname to yours in the above. (Note that you can connect to a compute node via ssh as long as you are running a job in the node.) If you have successfully logged in the compute node, you can now open the jupyter notebook in the web browser. Open your favorite web browser and go to `http://localhost:8888`.
