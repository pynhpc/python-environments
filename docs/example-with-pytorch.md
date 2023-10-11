# Running Pytorch Jobs on Hopper

Runs requiring Pytorch can be run directly using the system installed python or a python virtual 
environment or any one of the available containers.
 
## Running with the System Python in Batch Mode 
To run with the system python, log in to the cluster AMD head node which has a gpu card that 
allows for testing gpu codes.

```
ssh NetID@hopper-amd.orc.gmu.edu
```
On the hopper-amd headnode, load the GNU 10 and default python - version 3.9.9

```
module load gnu10
module load python
```
You can test and work on short running scripts directly on the headnode:

```
python main.py
```
Once you're ready, use a SLURM script to run your code in batch mode. When setting the SLURM 
parameters in your script, pay attention to the the following:

1 - The partition and QOS:
```
#SBATCH --partition=gpuq
#SBATCH --qos=gpuq
```
Groups that have access to the gpuq can use the `contrib-gpuq` partition with the 
corresponding group QOS. The time limit on the gpuq partiton defaults to 3 days:

```
#SBATCH --time=3-00:00:00
```
but can be set to a maximum of 5 days.

2 - GPU node options:

|Type of GPU  | SLURM setting               | No. of GPUs on Node |No. of CPUs |RAM  |
|------------ |-----------------------------|-------------------- |--------    |-----|
|A100 80GB    | --gres=gpu:A100.80gb:nGPUS  | 4                   | 64         |500GB|
|DGX A100 40GB| --gres=gpu:A100.40gb:nGPUs  | 8                   | 128        |1TB  |


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

## Running interactively on a GPU node

To work directly on a gpu node, start interactive session with

```
salloc -p gpuq -q gpu --nodes=1 --ntasks-per-node=4 --constraint=amd --gres=gpu:A100.80gb:1 --mem=50GB --pty $SHELL
```

The gres and other parameters can be adjusted to match the required resources. Once on the gpu node, the same steps 
as outlined above can be followed to set up your environment and run your code. If no time limit had been set 
the session will continue until it is ended with

```
exit
```
This will take you back to the head node.

## Managing pytorch and other packages with python virtual environments

To use additional libraries or different versions of pytorch and other libraries than what is available in the 
python modules, use [Python Virtual Environments](https://wiki.orc.gmu.edu/mkdocs/Python_Virtual_Environment/). The following steps summarize the process.

**NOTE**: To make sure your Python Virtual Environment runs across all nodes, it is important to use the 
versions of modules built for this. Before creating the Python Virtual Environment

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
python -m vevn pytorch-env

source pytorch-env/bin/activate

python -m pip install --upgrade pip

```

5 - Remove system python module and install torch modules (Refer to [this page](https://pytorch.org/get-started/locally/) for updated instructions on installing Pytorch)

``` 
module unload python
pip3 install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html

```
6 - Install additional modules as needed, test your scripts and deactivate the python virtual environment once you're done.

``` deactivate
```
In your SLURM script, to run with the Python Virtual Environment, you activate the Python Virtual Environment instead of loading the python module.

Sample script, `run.slurm`, for python virtual environments:
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
source activate ~/pytorch-env/bin/activate

## Execute your script
python main.py

```




###  Running Pytorch with Singularity Containers

Containers and examples for running Pytorch are available for all users can be found on the cluster at 

```
/containers/dgx/Containers/pytorch
```
 and 
```
/containers/dgx/Examples/Pytorch
```
You can copy the set up to your working directory on Hopper directly by

```
cp -r /containers/dgx/Examples/Pytorch/misc/* .
```

You can then modify the scripts to run in your directories. In the available example scripts the environmental variable $SINGULARITY_BASE points to `/containers/dgx/Containers`. Check [these pages](https://wiki.orc.gmu.edu/mkdocs/DGX_A100_User_Guide/#running-containerized-applications) 
for examples on runnig containerized version of Pytorch. 



## Common Pytorch Issues

1. GPU not recognized - In your Python Virtual Environment, you probably need to use an earlier version of Pytorch. Replace your pytorch install by re-installing an earlier verison e.g.:
```
pip3 install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html

```
2. CUDA Memory Error - Refer to this page for documentation on dealing with this error: [Pytorch FAQs](https://pytorch.org/docs/stable/notes/faq.html#my-gpu-memory-isn-t-freed-properly) 
