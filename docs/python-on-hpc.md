# Running Python on the ORC Clusters


<!--
>  <span style="color: red;">**NOTE**</span> 
> 
>       
>     To run Python jobs or build Python Virtual Environments that will run across all the nodes on Hopper, use the Python 
>     modules under the GCC/10 compiler:
>
>     ```
>      module load gnu10
>      module load python/<version> ## the default python is version 3.10
>
>      ```  
    
Different versions of Python are available on both Argo and Hopper clusters. Since the two clusters are 
set up in very similar ways, the information below on running Python jobs will be applicable 
to both clusters. 

The main differences between Argo and Hopper are outlined on this page: [Difference between Argo and Hopper](ARGO_vs_HOPPER.md)

-->

> [!NOTE]
> To run Python jobs or build Python Virtual Environments that will run across all the nodes on Hopper, use the Python 
> modules under the GCC/10 compiler:
>
>     ```
>      module load gnu10
>      module load python/<version> ## the default python is version 3.10
>
>      ```  
 
<span style="color: red;">The examples below will be based on the Hopper cluster</span>. Slight modifications in the SLURM scripts will be necessary 
to run them on Argo.

## Python Versions

To see the available version of Python, run the command

```
ml spider python
```

This will list all the available versions of Python that are installed on the cluster and include all the different builds.

```
------------------------------------------------------------------------------------------------
  python:
------------------------------------------------------------------------------------------------
     Versions:
        python/2.7.18-z2
        python/2.7.18-z4
        python/3.7.4-rg
        python/3.7.6-iu
        python/3.7.6-ks
        python/3.7.7-intel
        python/3.8.6-ff
        python/3.8.6-pi
        python/3.8.6-kg
        python/3.8.6-rp
        python/3.8.6-vw
        python/3.8.6-ye
        python/3.8.6-p2
        python/3.8.6-4q
        python/3.9.7-intel
        python/3.9.9-jh
     Other possible modules matches:
        intel-python  intelpython

------------------------------------------------------------------------------------------------
  To find other possible module matches execute:

      $ module -r spider '.*python.*'

------------------------------------------------------------------------------------------------
 
```
You can also run

```
module load gnu10
module avail python

```
Which will show you only the GCC builds or the Intel builds, depending on which compiler you're working with. 
Going with the recommended option (GNU-10.3.0 build), you should see

```
-------------------------------------------- GNU-10.3.0 ---------------------------------------------
 python/3.8.6-pi    python/3.9.9-jh (D)

  Where:
   D:  Default Module

```

Running 
```
module load python
```
will load the default version. You can also load a specific version with

```
module load python/<version>

```
With the Python module now in your path, you should be able to execute Python commands
and run Python scripts.

## Running a Python Job

### Interactively on a CPU 

<span style="color: red;">Python jobs should not be run directly on the head nodes</span>. The preferred method, even if you're testing 
a small job, is to start a debug session directly on a compute node using the debug partition and then test your script or, for short jobs, 
run it directly from the node. The default time limit on the debug partition is 1 hour. To get more information on the available partitions, resources, and limits on the node use the `sinfo` command.

To connect directly to a compute node and use the debug partition, use the `salloc` command together with additional SLURM parameters

```
salloc -p debug  -n 1  --cpus-per-task=12 --mem=15GB

```
However, if you want to run an interactive job that may require a time limit of more than 1 hour use the command shown below :
```
salloc -p normal  -n 1  --cpus-per-task=12 --mem=15GB -t 0-02:00:00

```
This command will allocate you a single node with 12 cores and 15GB of memory for 2 hours on the normal partition.
Once the resources become available, your prompt should now show that you're on one of the Hopper nodes.

```
salloc: Granted job allocation 
salloc: Waiting for resource configuration
salloc: Nodes hop065 are ready for job
[user@hop065 ~]$

```
Modules you loaded while on the head nodes are exported onto the node as well. If you had not already 
loaded any modules, you should be able to load them now as well. To check the currently loaded modules on the node use the command shown below :
```
[user@hop065 ~]$ module list

Currently Loaded Modules:
  1) use.own     3) prun/2.0       5) gnu10/10.3.0-ya   7) sqlite/3.37.1-6s   9) openmpi/4.1.2-4a
  2) autotools   4) hosts/hopper   6) zlib/1.2.11-2y    8) tcl/8.6.11-d4     10) python/3.9.9-jh

Inactive Modules:
  1) openmpi4
```
 You can now start Python to debug your code or use it interactively.
```
[user@hop065 ~]$ python
Python 3.9.9 (main, Mar 25 2022, 16:08:31)
[GCC 10.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
You could also run your Python script directly

```
$ python myscript.py
```

The interactive session will persist until you type the 'exit' command as shown below:

```
$ exit

exit
salloc: Relinquishing job allocation 

```

### Interactively on a GPU 

In a similar manner, you can start an interactive session on a GPU node with

```
salloc -p gpuq -q gpu --ntasks-per-node=1 --gres=gpu:A100.40gb:1 -t 0-01:00:00 

```

### Using a SLURM Submission Script

Once your tests are done and you're ready to run longer Python jobs, you should now switch to using 
the batch submission with SLURM. To do this, you write a SLURM script setting the different parameters 
for your job, loading the necessary modules, and executing your Python script which is then submitted to the 
selected queue from where it will run your job. Below is an example SLURM script (`run.slurm`):

```

#!/bin/bash
#SBATCH --partition=normal                 # will run on any cpus in the 'normal' partition
#SBATCH --job-name=python-cpu
## Separate output and error messages into 2 files.
## NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID
#SBATCH --output=/scratch/%u/%x-%N-%j.out  # Output file
#SBATCH --error=/scratch/%u/%x-%N-%j.err   # Error file
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1                   # up to 48 per node
#SBATCH --mem-per-cpu=3500M                 # memory per CORE; maximum is 180GB per node
#SBATCH --export=ALL
#SBATCH --time=0-01:00:00                   # set to 1hr; please choose carefully

set echo
umask 0027

module load gnu10
module load python                          # load the recommended Python version

python myscript.py                          # execute your Python script

```

If you need GPU nodes for your Python job, you would change the partition information to the `gpuq` and set the number of GPU nodes needed.

``` 
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                           # need to select 'gpu' QoS
#SBATCH --job-name=python-gpu
## Separate output and error messages into 2 files.
## NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID
#SBATCH --output=/scratch/%u/%x-%N-%j.out  # Output file
#SBATCH --error=/scratch/%u/%x-%N-%j.err   # Error file
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1                 # up to 128; 
#SBATCH --gres=gpu:A100.80gb:1              # up to 8; only request what you need
#SBATCH --mem-per-cpu=3500M                 # memory per CORE; total memory is 1 TB (1,000,000 MB)
#SBATCH --export=ALL 
#SBATCH --time=0-01:00:00                   # set to 1hr; please choose carefully

set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi

module load gnu10                           
module load python

python myscript.py

```

Please use the scratch space to submit your job's SLURM script with

```
sbatch run.slurm

```
To access scratch space use `cd /scratch/UserID` command to change directories(replace 'UserId' with  your GMU GMUnetID). Please note that scratch directories have no space limit and data in /scratch gets purged 90 days from the date of creation, so make sure to move your files to a safe place before the purge.

To copy files directly from scratch to your project space you can use the `cp` command to create a copy of the contents of the file or directory specified by the SourceFile or SourceDirectory parameters into the file or directory specified by the TargetFile or TargetDirectory parameters. The `cp` command also copies entire directories into other directories if you specify the -r or -R flags.
The command below copies entire files from the scratch space to your project space (" /projects/orctest" as shown in the example below, where " /projects/orctest" is a project space)
```
[UserId@hopper2 ~]$ cd /scratch/UserId
[UserId@hopper2 UserId]$ cp -p -r *  /projects/orctest

```

### Optimizing your GPU runs
Current available GPU node options

|Type of GPU  | SLURM setting               | No. of GPUs on Node |No. of CPUs |RAM  |
|------------ |-----------------------------|-------------------- |--------    |-----|
|A100 80GB    | --gres=gpu:A100.80gb:nGPUS  | 4                   | 64         |500GB|
|DGX A100 40GB| --gres=gpu:A100.40gb:nGPUs  | 8                   | 128        |1TB  |

The way the GPU nodes are partitioned will likely change over time to optimize utilization.

The best way to take advantage of this Multi-Instance GPU (MIG) mode operation is to analyze the demands of your job and determine which GPU slice is available and suitable for it. For example, if your simulation uses very small GPU memory, you would be better off using a 1g.5gb (where 5GB is the memory you get in the GPU) slice and leaving the bigger slices to jobs that need more GPU memory. Another consideration for machine learning jobs is the difference in demands of training and inference tasks. Training tasks require more computation and memory, therefore they perform best on a full GPU node or a large slice, whereas inference tasks can be adequately performed on smaller slices.

You would modify your SLURM script so that you are now requesting a suitable GPU slice:

``` 
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                           # need to select 'gpu' QoS
#SBATCH --job-name=python-gpu
## Separate output and error messages into 2 files.
## NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID
#SBATCH --output=/scratch/%u/%x-%N-%j.out  # Output file
#SBATCH --error=/scratch/%u/%x-%N-%j.err   # Error file
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1                 # up to 128; 
#SBATCH --gres=gpu:1g.10gb:1                 # request a slice of the GPU
#SBATCH --mem-per-cpu=3500M                 # memory per CORE; total memory is 1 TB (1,000,000 MB)
#SBATCH --export=ALL 
#SBATCH --time=0-01:00:00                   # set to 1hr; please choose carefully

set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi

module load gnu10                            
module load python

python myscript.py

```

Read more about the Hopper GPU nodes and other examples on the [DGX USER GUIDE.](DGX_A100_User_Guide.md)

## Running Parallel Python Jobs 

When working on a cluster computer, it is natural to ask how to take
advantage of all these nodes and cores in order to speed up
computation as much as possible. On a laptop, one common approach is to
use the `Pool` class in the Python `multiprocessing` library to
distribute computation to other cores on the machine. While this
approach certainly works on a cluster too, it does not allow you to take
full advantage of the available computing power. Each job is limited to
a single node and all the cores that are currently available on it. 

### Multithreaded Python Job

Below is an example SLURM script that can be used to run a Python script
that implements the 'multiprocessing' module.

```

#!/bin/sh

## Give your job a name to distinguish it from other jobs you run.
#SBATCH --job-name=threaded
#SBATCH --partition=normal                 # will run on any cpus in the 'normal' partition
## Separate output and error messages into 2 files.
## NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID
#SBATCH --output=/scratch/%u/%x-%N-%j.out  # Output file
#SBATCH --error=/scratch/%u/%x-%N-%j.err   # Error file
#SBATCH --constraint=amd ## or intel
#SBATCH --nodes=1   ## all threads need to be on a single node
#SBATCH --cpus-per-task=24   ## 48 or 64 processor
#SBATCH --mem=5G        # Total memory needed per task (units: K,M,G,T)
#SBATCH --time=0-01:00:00                   # set to 1hr; please choose carefully 
set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi
## Load the relevant modules needed for the job
module load gnu10                            
module load python

## Run your program or script
python <your-threaded-script>.py

```

The Python script below which can be downloaded here: [multithreaded.py](parallel_jobs_examples/mpithread.py) can be used as a test case for threaded jobs in Python.

```python
#!/usr/bin/env python

import numpy as np
import multiprocessing as mp

if __name__ == '__main__':
  np.random.seed(0);

  # create two matrices to be passed
  # to two different processes
  mat1 = np.random.rand(3,3);
  mat2 = np.random.rand(2,2);

  # define number of processes
  ntasks =2;

  # create a pool of processes
  p = mp.Pool(ntasks);

  # feed different process with same task
  # but different data and print the result
  print(p.map(np.linalg.eigvals,[mat1,mat2]))

```

### Distributed Python Jobs with mpi4py

The `mpi4py` library has a Pool-like class that is very similar to the
one in the `multiprocessing` library. Here, we describe how to setup a Python virtual environment
to use `mpi4py` run Python code to take advantage
of a much larger number of cores.

#### Installing mpi4py in a Python Virtual Environment

When installing Python modules, we recommend using a [Python Virtual
Environment](Python_Virtual_Environment.md). When working on a
project you may want to install a number of different
packages. We recommend creating one VE for each project and installing
everything that you need into it.

For the purposes of this demonstration, let’s create a virtual
environment called MPIpool and install `mpi4py` into it.

```
[UserId@hopper1 ~]$ module load gnu10
[UserId@hopper1 ~]$ module load python
[UserId@hopper1 ~]$ module load openmpi4/4.1.2
[UserId@hopper1 ~]$ python -m venv ~/MPIpool
[UserId@hopper1 ~]$ source ~/MPIpool/bin/activate
(MPIpool) [UserId@hopper1 ~]$ pip install mpi4py
Collecting mpi4py
  Using cached h⁣ttps://files.pythonhosted.org/packages/04/f5/a615603ce4ab7f40b65dba63759455e3da610d9a155d4d4cece1d8fd6706/mpi4py-3.0.2.tar.gz
Installing collected packages: mpi4py
  Running setup.py install for mpi4py ... done
Successfully installed mpi4py-3.0.2

```

#### Using MPIPoolExecutor in a Python Program

Here we have a sample Python program (which can be downloaded here: [MPIpool.py](parallel_jobs_examples/MPIpool.py) ) that calculates prime numbers. It
uses the `MPIPoolExecutor` class to farm out calculations to "workers".
The workers can be running on any node and core in the cluster. There
must always be one "manager" that is responsible for farming out the
work, and collecting the results when finished.

```

# MPIpool.py

from mpi4py.futures import MPIPoolExecutor
import math
import textwrap

def calc_primes(range_tuple):
    """Calculate all the prime numbers in the given range."""
    low, high = range_tuple
    if low <= 2 < high:
        primes = [2]
    else:
        primes = []

    start = max(3,low)   # Don't start below 3
    if start % 2 == 0:   # Make sure start is odd, i.e. skip evens
        start += 1

    for num in range(start, high, 2):  # increment by 2's, i.e. skip evens
        if all(num % i != 0 for i in range(3, int(math.sqrt(num)) + 1, 2)):
            primes.append(num)

    return primes

def determine_subranges(fullrange, num_subranges):
    """
    Break fullrange up into smaller sets of ranges that cover all
    the same numbers.
    """
    subranges = []
    inc = fullrange[1] // num_subranges
    for i in range(fullrange[0], fullrange[1], inc):
        subranges.append( (i, min(i+inc, fullrange[1])) )
    return( subranges )

if __name__ == '__main__':
    fullrange = (0, 100000000)
    num_subranges = 1000
    subranges = determine_subranges(fullrange, num_subranges)

    executor = MPIPoolExecutor()
    prime_sets = executor.map(calc_primes, subranges)
    executor.shutdown()

    # flatten the list of lists
    primes = [p for plist in prime_sets for p in plist]
    print(textwrap.fill(str(primes),80))

```

The main work is done in the `calc_primes()` function, which is what the
workers run. It calculates all the prime numbers within a range defined
by `rangeTuple`, a vector that contains two values: the lower and upper
bounds of the range.

The rest of the code runs on the "manager". It calls the
`determine_subranges()` function to define the different pieces of work
to send to the workers. The `MPIPoolExecutor.map()` function actually
handles all the complexity of coordinating communications with workers,
farming out the different tasks, and then collecting the results.

The `mpi4py` [documentation](https://mediawiki.org) suggest that when
using `MPIPoolExecutor`, your code should use the
`if __name__ == '__main__':` code construct at the bottom of your main
file in order to [prevent workers from spawning more
workers](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html#mpipoolexecutor).

#### Submitting the Program to SLURM

Here we provide a SLURM script for running such a job.

```

#!/bin/sh

## Give your job a name to distinguish it from other jobs you run.
#SBATCH --job-name=MPIpool
#SBATCH --partition=normal
## Separate output and error messages into 2 files.
## NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayID, %a=arrayTaskID
#SBATCH --output=/scratch/%u/%x-%N-%j.out  # Output file
#SBATCH --error=/scratch/%u/%x-%N-%j.err   # Error file
## Slurm can send you updates via email
#SBATCH --mail-type=BEGIN,END,FAIL         # ALL,NONE,BEGIN,END,FAIL,REQUEUE,..
#SBATCH --mail-user=<GMUnetID>@gmu.edu     # Put your GMU email address here
## Specify how much memory your job needs. (2G is the default)
#SBATCH --mem=8G        # Total memory needed per task (units: K,M,G,T)
## Specify how much time your job needs. (default: see partition above)
#SBATCH --time=0-02:00   # Total time needed for job: Days-Hours:Minutes
#SBATCH --ntasks=51   # 50 workers, 1 manager
set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi
## Load the relevant modules needed for the job
module load gnu10                            
module load python
module load openmpi4/4.1.2
source ~/MPIpool/bin/activate

## Run your program or script
mpirun -np $SLURM_NTASKS python -m mpi4py.futures MPIpool.py

```

<span style="color:red">Be sure to replace the <GMUnetID> (including the
`<` and `>`) with you own email address.</span>

Note that we set `--ntasks=51` in order to allocate 1 manager and 50
workers. There must always be only 1 manager and at least 1 worker. Note
that we use the `$SLURM_NTASKS` environment variable in the call to
`mpirun` to make sure that the number of cores used equals the number
allocated by the `--ntasks=` option.

Because `mpi4py` is based on the MPI libraries, we need to load one of
the MPI modules. Here we have chosen OpenMPI. The `mpirun` or `mpiexec`
program must be used to properly launch an MPI program, and this program
is no exception.

The runtime for this program using 50 workers is about 1 minute. That is
significantly faster than the 45 minutes needed to run the program using
a single core. Of course, there is a point of diminishing returns (and
even an added cost) in adding more and more workers. It is good to
experiment with different numbers to see how many workers are optimal.
The maximum number of cores that a user can request is currently 300.
This may change in the future.

This is an example of an algorithm that is "embarrassingly parallel". It
is very easy to divide it up into smaller pieces and pass them out. Many
algorithms are not so easy to parallelize in this way. MPI is a very
mature library, and it has the tools to handle problems that are much
more complex than this. It is the de facto standard for doing large
scale parallelization, and if that is your goal you can benefit from
learning more about it. Those interested in a more "Pythonic" library
may want to look into [Dask](https://dask.org/).
## Using External Python Packages

To install and run your Python code with external Python packages, after loading the Python module, first
create a directory for storing those packages (e.g. `~/python-packages/projectX `)

```
mkdir ~/python-packages
mkdir ~/python-packages/projectX

```

Then install the appropriate packages in there:

```
pip install <package1> -t ~/python-packages/projectX

```

To run your code with these extra packages, you would need to add the `export` command to your
SLURM submission script so that the last part  would now be

```
module load gnu10
module load python
export PYTHONPATH=~/python-packages/projectX:$PYTHONPATH

python myscript.py

```
If instead, you were running interactively, then you need to run the `export` command from the terminal

```
$ export PYTHONPATH=~/python-packages/projectX:$PYTHONPATH

```
 
## Running with Python Virtual Environments

To have better control over the Python packages and libraries you need to use on the cluster, the
best way to run Python is using Python Virtual Environments. This is especially useful for 
codes that use Tensorflow, Keras or Pytorch. Read our instructions on 
[building Python Virtual Environments](Python_Virtual_Environment.md)
and [how to run Tensorflow](How_to_Run_Tensorflow/Running_Tensorflow.md).

>  **Remember**  
>
>  When running on Argo, the SLURM scripts have to be updated so that they can be run on Argo. The main differences between
>  Argo and Hopper are detailed in [these pages](http://wiki.orc.gmu.edu/mkdocs/ARGO_vs_HOPPER/).

## Running with Jupyter NoteBooks

You also have the option of using Jupyter Notebooks (on Hopper) to run Python code. The steps for doing this are outlined in [these pages.](Running_Jupyter_Notebooks.md)  
