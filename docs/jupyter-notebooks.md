# Running Jupyter Notebooks on the ORC Clusters

JupyterLab is available as one of the interactive apps that can be launched directly from Open OnDemand. 

To use Jupyter Notebooks on Hopper, the first step is to start an Open OnDemand (OOD)session. In your browser, go to [ondemand.orc.gmu.edu](https://ondemand.orc.gmu.edu). You will need to authenticate with your Mason credentials before you can access the dashboard.

* Once you authenticate, you should be directed to the OOD dashboard.

![](img/g16-img/ood-interactive-apps.png){: style="height:300px"}                


* You'll need to click on the "Interactive Apps" icon to see the apps that are available. 


## From the OOD Dashboard

* Select JupyterLab from the listed apps

![](img/g16-img/ood-interactive-jn.png){: style="height:550px"}

* Set your confiuration from the given options and launch.

![](img/g16-img/ood-interactive-jn-launch.png){: style="height:550px"}

* Once your session is ready to start, you can 'Connect'

![](img/g16-img/ood-interactive-jn-launch-2.png){: style="height:550px"}

This will start a Jupyter Lab session where you can select to start Python Notebook.



## From the OOD Hopper Desktop App
The other way to run Jupyter Notebooks on the OOD server is by starting a desktop session. 
* Start by selecting the Hopper Desktop option from the interactive apps.
 
![](img/g16-img/ood-interactive-hop-desktop.png){: style="height:450px"}         

* Click on the Hopper Desktop and set your configuration (number of hours upto 12 and number of cores upto 12). You'll see the launch option once the resources become available.      

![](img/g16-img/ood-interactive-hop-desktop-launch.png){: style="height:550px"}  

* After launching the session, you'll get connected to a node on Hopper with a desktop environment from where you can now start the terminal (using the highlighted 'terminal emulator' icon)

![](img/g16-img/ood-interactive-hop-desktop-konsole.png){: style="height:585px"} 


* From the launched shell session, load the python module with      

```
module load python

```      
and start a Jupyter Notebook with   

```
jupyter-notebook
```
    

![](img/g16-img/ood-interactive-hop-desktop-jn-1.png){: style="height:510px"}

* The Jupyter Notebook session should now start.     


## Running Longer Python Jobs

The Jupyter Notebok sessions are interactive and can be run for a maximum of 12 hours at a time. To run longer
python jobs, you need to convert your code into a coherent python script that can then be submitted through SLURM.
Instructions on running conventional Python jobs are  given in [these pages](Running_Python.md) 

