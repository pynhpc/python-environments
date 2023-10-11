---
title: "Tutorial"
permalink: /docs/tutorial/
excerpt: "How to manage python virtual environments on HPC clusters."
last_modified_at: #2021-06-07T08:48:05-04:00
redirect_from:
  - /theme-setup/
toc: true
---


# Managing Python Packages with Conda Environments on the Cluster

Conda environments are an excellent method for managing python packages and libraries in the cluster environment. One approach on clusters is using a centralized Anaconda3 Module, but from experience 
this usually caused path issues and prevented other modules from working correctly. Instead, it is possible to use a mini version of Anaconda, miniconda which
includes just conda and its dependencies. It is also very small and can be downloaded directly to your /home directory.


The following instructions have steps for idownloading and installing Miniconda in your home directory on the cluster and running it.

## Installing Maker in a python virtual environment with Miniconda3

1. Download miniconda3:

```console
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
```

2. Install it in your preferred path:

```console
 bash miniconda.sh -b -p $HOME/miniconda
```

You can now create a custom conda environment. In the steps below, a conda environment for the package 'Maker' is created from
a conda environment file.

3. Activate base conda environment:

```console
 source $HOME/miniconda/bin/activate
 export PYTHONNOUSERSITE=true

```

4. Download and create your virtual environment for maker using the environment yml file. This was generated from tests on the cluster so it should have all the necessary libraries. To download the environment.yml file [maker_3.01.03_environment.yml](maker/maker_3.01.03_environment.yml) use:

```console
wget https://wiki.orc.gmu.edu/mkdocs/maker/maker_3.01.03_environment.yml
```
Then create the virtual environemnt with:


```console
conda env create -f maker_3.01.03_environment.yml

```

5. You should now be able to activate it with:

```console
conda activate maker_3.01.03

```

6. Alternatively, you can simply create the environment with

```console
conda env create -n maker_env
```

then activate it and install necessary libraries/use python with

```console
conda activate maker_3.01.03

python
```

## Running in batch mode with SLURM

Below is an example of a SLURM script ([run.slurm](maker/run.slurm)) to run in this conda environment. You can modify the different SLURM parameters to match what you need:

Download the run.slurm script:

```console
wget https://wiki.orc.gmu.edu/mkdocs/maker/run.slurm
```


```bash
#!/bin/bash
#SBATCH --job-name=maker_test
#SBATCH --partition=normal
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=64
#SBATCH --output=maker_test_%j.out
#SBATCH --error=maker_test_%j.err
#SBATCH --mem=50GB
#SBATCH --time=0-12:00:00

### Load modules
module unload openmpi4
module load gnu10 openmpi

### Activate virtual environment
source /home/$USER/miniconda/bin/activate
source activate maker_3.01.03

### Set environment variables:
export LIBDIR=/home/$USER/miniconda/envs/maker_3.01.03/share/RepeatMasker/Libraries

### Execute program
mpiexec -n ${SLURM_NTASKS_PER_NODE} maker [OPTIONS]
```


Run this with the sbatch command:

```console
sbatch run.slurm
```

If you ssh onto the node on which the jobs starts, you should see it now utilizes all the available cpus for maker. To see which nodes the job is running on, you can use the squeue command:

```console
squeue --me
```


## Examples:

<figure>
  <img src="{{ '/assets/images/mm-gh-pages.gif' | relative_url }}" alt="creating a new branch on GitHub">
</figure>

You can also install the theme by copying all of the theme files[^structure] into your project.

To do so fork the [Minimal Mistakes theme](https://github.com/mmistakes/minimal-mistakes/fork), then rename the repo to **USERNAME.github.io** --- replacing **USERNAME** with your GitHub username.

<figure>
  <img src="{{ '/assets/images/mm-theme-fork-repo.png' | relative_url }}" alt="fork Minimal Mistakes">
</figure>

**GitHub Pages Alternatives:** Looking to host your site for free and install/update the theme painlessly? [Netlify][netlify-jekyll], [GitLab Pages][gitlab-jekyll], and [Continuous Integration (CI) services][ci-jekyll] have you covered. In most cases all you need to do is connect your repository to them, create a simple configuration file, and install the theme following the [Ruby Gem Method](#ruby-gem-method) above.
{: .notice--info}



### Remove the Unnecessary

If you forked or downloaded the `minimal-mistakes-jekyll` repo you can safely remove the following folders and files:

- `.editorconfig`
- `.gitattributes`
- `.github`
- `/docs`
- `/test`
- `CHANGELOG.md`
- `minimal-mistakes-jekyll.gemspec`
- `README.md`
- `screenshot-layouts.png`
- `screenshot.png`

**Note:** If forking the theme be sure to update `Gemfile` as well. The one found at the root of the project is for building the theme's Ruby gem and is missing dependencies. To properly setup a [`Gemfile`](https://github.com/mmistakes/minimal-mistakes/blob/master/docs/Gemfile) with the theme, consult the "[Install Dependencies](https://mmistakes.github.io/minimal-mistakes/docs/installation/#install-dependencies)" section.
{: .notice--warning}

## Setup Your Site

Depending on the path you took installing Minimal Mistakes you'll setup things a little differently.

**ProTip:** The source code and content files for this site can be found in the [`/docs` folder](https://github.com/mmistakes/minimal-mistakes/tree/master/docs) if you want to copy or learn from them.
{: .notice--info}





Then run `bundle update` and add `theme: minimal-mistakes-jekyll` to your `_config.yml`.

**v4 Breaking Change:** Paths for image headers, overlays, teasers, [galleries]({{ "/docs/helpers/#gallery" | relative_url }}), and [feature rows]({{ "/docs/helpers/#feature-row" | relative_url }}) have changed and now require a full path. Instead of just `image: filename.jpg` you'll need to use the full path eg: `image: /assets/images/filename.jpg`. The preferred location is now `/assets/images/` but can be placed elsewhere or externally hosted. This applies to image references in `_config.yml` and `author.yml` as well.
{: .notice--danger}

---

That's it! If all goes well running `bundle exec jekyll serve` should spin-up your site.
