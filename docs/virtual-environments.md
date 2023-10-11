Python Virtual Environments On Hopper
--------------------------

There are different approaches in the Python ecosystem for creating virtual environments (VE): `pyenv`, `venv`, `virtualenv`, `virtualenvwrapper`, and `pipenv`. Among these, the methods `venv` and `virtualenv` are quite similar, and either one can be chosen. The key distinction between these two lies in how they handle Python executables within the virtual environment folder:

- `venv` creates a virtual environment without copying Python executables.
- `virtualenv` creates a virtual environment by copying Python executables.

Before utilizing Python Virtual Environments on Hopper, please take note of the following:

- **Python environments need to be established separately on Hopper. A Python Virtual environment built on Hopper should not be expected to function on any other system.**
- **For Hopper, it is recommended to employ `venv` for creating virtual environments, following the process detailed below.**


## Starting an interactive Session on Hopper GPUs

If you intend to run the Python Virtual Environments on the Hopper DGX-A100 GPU nodes, the first step is to get directly on the
GPU node by starting an interactive session with the `salloc` command

```
salloc -p gpuq -q gpu -n 1 --ntasks-per-node=1 --gres=gpu:A100.40gb:1 --mem=50GB

```
Currently available GPU node options

|Type of GPU  | SLURM setting               | No. of GPUs on Node |No. of CPUs |RAM  |
|------------ |-----------------------------|-------------------- |--------    |-----|
|A100 80GB    | --gres=gpu:A100.80gb:nGPUS  | 4                   | 64         |500GB|
|DGX A100 40GB| --gres=gpu:A100.40gb:nGPUs  | 8                   | 128        |1TB  |

If you don't want to use DGX nodes you can also use the below command to start an interactive session on the contrib-gpuq partition
```
salloc -p contrib-gpuq -q gpu -n 1 --ntasks-per-node=1 --gres=gpu:1 --mem=5GB
```
The way the GPU nodes are partitioned will likely change over time to optimize utilization. Use the `sinfo` command to view the list of nodes available, the time restriction for each node, and the available partitions.

Once you have the interactive session started, you should load the necessary modules. For Python on the DGX nodes,
 use the module `python/3.8.6-ff` which has been built to run across both the CPU nodes and GPU nodes. Currently,
 other Python modules will not work across both the GPU and CPU nodes because of the differences in architecture.

## Creating a Python Virtual Environment

`venv` is a Python module available in the standard library and compatible with most Python versions. It simplifies the creation of isolated environments for Python projects. Unlike other environment management tools, `venv` ensures that libraries are not shared with other virtual environments by default. You also have the option to prevent access to globally installed libraries if desired.

When you utilize `venv` to create a virtual environment, it establishes an isolated space where you can install your project's dependencies independently. Unlike tools like Virtualenv, `venv` does not rely on copying the Python interpreter binary into the virtual environment directory. Instead, it often employs symbolic links to the existing Python interpreter on your system. As a result, the virtual environment created by `venv` contains symbolic links to the Python executable, optimizing disk space usage.

To summarize, `venv` is a built-in module within Python's standard library that offers a simple method for establishing isolated environments tailored to Python projects. It avoids library sharing between virtual environments and enables efficient management of project-specific dependencies.

# Python Virtual Environment Setup for Hopper Cluster

To ensure that your Python Virtual Environment runs consistently across all nodes, it's important to use modules built for this purpose. Before creating the Python Virtual Environment, follow these steps:


1 - First switch modules to GNU 10 compilations:
```
module load gnu10/10.3.0-ya
```

2 - Check and load python module
```
module avail python
module load python/3.8.6-pi
```
3 - Now you can create the python virtual environment. After it is created, activate it and upgrade pip
```
python -m venv py-env

source py-env/bin/activate

python -m pip install --upgrade pip
```

4 - Remove system python module and install  modules 

``` 
module unload python
pip install sklearn

```
5 - Install additional modules as needed, test your scripts and deactivate the python virtual environment once you're done.
``` 
deactivate

```


### Example:
================================================================== 

```
[user@hopper2 ~]$ python -m venv py-env
[user@hopper2 ~]$ source py-env/bin/activate
(py-env) [user@hopper2 ~]$ python -m pip install --upgrade pip
Collecting pip
  Downloading pip-23.2.1-py3-none-any.whl (2.1 MB)
     |████████████████████████████████| 2.1 MB 13.6 MB/s
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 20.2.1
    Uninstalling pip-20.2.1:
      Successfully uninstalled pip-20.2.1
Successfully installed pip-23.2.1
(py-env) [user@hopper2 ~]$ module unload python
(py-env) [user@hopper2 ~]$ pip install sklearn
Collecting sklearn
  Downloading sklearn-0.0.post7.tar.gz (3.6 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Building wheels for collected packages: sklearn
  Building wheel for sklearn (pyproject.toml) ... done
  Created wheel for sklearn: filename=sklearn-0.0.post7-py3-none-any.whl size=2951 sha256=c62d78ad5864da7dddeea4f58c96b35c6bab2584dec8748ce5687b788491f2eb
  Stored in directory: /home/smudund/.cache/pip/wheels/bc/86/46/dd4e366dc5e1303b4d6927d2a603a1ae7f979d488a5d202330
Successfully built sklearn
Installing collected packages: sklearn
Successfully installed sklearn-0.0.post7
(py-env) [user@hopper2 ~]$ deactivate
[user@hopper2 ~]$
```
## Things to remember before creating a virtual environment
1. To create a Python Virtual Environment from a login node, **a Python module must first be
loaded using the ‘module load’ command**. 

**On Hopper**, you need to make sure you
have loaded one of the correct Python modules compiled under GNU 10 to 
run across all nodes. Do this by first loading the gnu10 module and then the Python 
modules under it:

```
module load gnu10
module load python/<version>
```

2. Using “pip install <package_name>” within an activated Virtual Environment installs
the package in the Virtual Environment’s home directory and is available only to that Virtual Environment.
However, if you use “pip install --user <package_name>” within an
activated Virtual Environment, then the package is installed both in the Virtual Environment’s home
directory and in /home/$USER/.local directory. By being installed in the
/home/$USER/.local directory that package is then available to all Virtual Environments
you create. So, the use of “--user” option with *pip install* is not
recommended.

## Using a Python Virtual Environment in a SLURM submission script

Below is a sample SLURM submission script. Save the information into `run.slurm`,  
update the timing information, the `<N_CPU_CORES>`, `<MEMORY>` and `<N_GPUs>` to reflect the 
number of CPU cores and GPUs you need (referring to the table above) and submit it by entering 

```
sbatch run.slurm
```

Sample script, `run.slurm`:
```
#!/bin/bash 
#SBATCH --partition=gpuq 
#SBATCH --qos=gpu 
#SBATCH --job-name=gpu_basics 
#SBATCH --output=gpu_basics.%j.out 
#SBATCH --error=gpu_basics.%j.out 
#SBATCH --nodes=1 
#SBATCH --ntasks-per-node=<N_CPU_CORES> 
#SBATCH --gres=gpu:A100.80gb:<N_GPUs> 
#SBATCH --mem=<MEMORY>  
#SBATCH --export=ALL 
#SBATCH -time=0-01:00:00 
#
set echo 
umask 0022 
# to see ID and state of GPUs assigned
nvidia-smi 

## Load the necessary modules
module load gnu10
module load python

## Execute your script
python main.py

```


## Adding your Python Virtual Environment as a Kernel in JupyterLabs

Python Virtual Environments created on Hopper can be added as kernels to the 
[JupyterLab](Running_Jupyter_Notebooks.md) sessions started under 
[Open OnDemand](open_on_demand_on_Hopper.md). 
To see your Python Virtual Environment as a kernel,
first, activate the virtual environment from the command line:

```
source test-env/bin/activate

```
Next pip install ipykernel
 
```
pip install ipykernel
```
and then add the kernel for the virtual environment

```
Python -m ipykernel install --user --name=env-name
```
Finally, deactivate your environment. After this, the next time you start a JupyterLab session
on [Open OnDemand](open_ondemand_on_Hopper.md) you should see your added kernel as an option.


