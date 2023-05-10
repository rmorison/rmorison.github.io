+++
title = "Jupyter Setup with Mamba and Conda"
author = ["Rod Morison"]
date = 2023-05-10T16:54:00-07:00
draft = false
topics = ["Data Engineering", "Python", "Machine Learning"]
description = "A detailed, from scratch guide to Jupyter installation with Mamba and Conda for data engineering, machine learning, ..."
+++

{{< figure src="/ox-hugo/2_men_in_black_and_gray_jacket_standing_on_rock_formation_during_night_time-scopio-d494b247-0a74-4382-986d-24f0d5d6e930.jpg" >}}

Jupyter notebooks are widely used and a great tool for interactive Python work. Notebooks are especially useful for analytics, machine learning, and scientific computing demonstrations.


## Python Packaging (in about 100 words) {#python-packaging--in-about-100-words}

There are many (many) ways to install and run Jupyter notebooks. The one given here includes some recent best practices incorporated into the [Mamba](https://mamba.readthedocs.io/en/latest/index.html) package manager. It's worth noting the alternatives.

1.  [Conda](https://docs.conda.io/projects/conda/en/latest/index.html) - You'll get and use Conda with a Mamba install. In fact, it's the inspiration for Mamba: fast Conda. But you can work with Conda straight away, without Mamba, if you like.
2.  [Pip](https://pip.pypa.io/en/stable/) - The canonical Python installer. Available [when the need arises](https://www.anaconda.com/blog/using-pip-in-a-conda-environment), within your Mamba / Conda install.
3.  [Poetry](https://python-poetry.org/) - A leading choice for building Python libraries and service applications. However, since binary library dependencies are fulfilled by the operating system installation, can be a pita.
4.  [Pipenv](https://pipenv.pypa.io/en/latest/), [Flit](https://flit.pypa.io/en/stable/), [Hatch](https://hatch.pypa.io/latest/), ... - In the Poetry camp.


## Mamba Install {#mamba-install}

The [Mamba Installation](https://mamba.readthedocs.io/en/latest/installation.html) page sends you right over to the [miniforge repo](https://github.com/conda-forge/miniforge#mambaforge) for details. There are self-extracting shell archives there, you can download and install with. Or you can `curl` or `wget` the same file. I'll use `curl`.

```shell
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh"
bash Mambaforge-$(uname)-$(uname -m).sh
```


#### What Got Installed and How Mamba Runs {#what-got-installed-and-how-mamba-runs}

_This section is optional and covers how mamba is invoked, skip if you just want to install stuff._

It's worth taking a look at what the installer setup, insofar as how Mamba runs. Check the tail of your shell rc file. The exact file varies by shell and platform, but on Ubuntu for Bash,

```shell
tail -25 ~/.bashrc
```

gives

```shell
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/me/mambaforge/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/me/mambaforge/etc/profile.d/conda.sh" ]; then
	. "/home/me/mambaforge/etc/profile.d/conda.sh"
    else
	export PATH="/home/me/mambaforge/bin:$PATH"
    fi
fi
unset __conda_setup

if [ -f "/home/me/mambaforge/etc/profile.d/mamba.sh" ]; then
    . "/home/me/mambaforge/etc/profile.d/mamba.sh"
fi
# <<< conda initialize <<<
```

If you look at the `mamba.sh` and `conda.sh` you'll notice that `mamba` and `conda` are implemented as shell functions that call programs in `mambaforge/bin`.

This snippet from  `mamba.sh` is how Mamba wires up to your shell/terminal.

```shell
mamba() {
    \local cmd="${1-__missing__}"
    case "$cmd" in
	activate|deactivate)
	    __conda_activate "$@"
	    ;;
	install|update|upgrade|remove|uninstall)
	    __mamba_exe "$@" || \return
	    __conda_reactivate
	    ;;
	*)
	    __mamba_exe "$@"
	    ;;
    esac
}
```

The takeaway here is Mamba falls back to Conda for `activate` and `deactivate`, commands we'll cover below.

Lets look at the programs that actually get invoked, after this indirection.

```shell
(cd ~/mambaforge/bin/ && file conda* mamba* python*)
```

outputs (slightly edited)

```shell
conda:             a $HOME/mambaforge/bin/python script, ASCII text executable
conda2solv:        ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), ...
conda-env:         a $HOME/mambaforge/bin/python script, ASCII text executable
mamba:             a $HOME/mambaforge/bin/python script, ASCII text executable
mamba-package:     ELF 64-bit LSB pie executable, x86-64, version 1 (GNU/Linux), ...
python:            symbolic link to python3.10
python3:           symbolic link to python3.10
python3.1:         symbolic link to python3.10
python3.10:        ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), ...
python3.10-config: POSIX shell script, ASCII text executable, with very long lines (465)
python3-config:    symbolic link to python3.10-config
```

Tl;dr: The installation comes with a recent version of Python, which is (or aliased to) a compiled binary program, `python3.10` here. Mamba and Conda themselves are Python scripts. However, Mamba's performance comes from invoking compiled binary code via libraries and it's use of an [SAT package resolution algorithm](https://mamba.readthedocs.io/en/latest/advanced_usage/package_resolution.html). The [Mamba repo](https://github.com/mamba-org/mamba) has the implementation itself, if you're interested.


## Conda Environments {#conda-environments}

Post install, if you start a new shell or source the shell rc file cited above, e.g., `source ~/.bashrc` Mamba will prepend your shell prompt with an environment name in parens. If you have a fresh install that will look like

```shell
(base) rod@host:~$
```

It's `(base)` that we're looking for, the rest depends on your current shell prompt settings.


### What's a Conda Environment? {#what-s-a-conda-environment}

Python, similar to many other languages, has evolved over the decades of its use. The needs of Python developers and the products they create drove an evolution in how the language is installed and run. In days of yore programmers developed a variety of ad-hoc ways to keep the Python setup for different projects from colliding. Let's say project A only worked with Python V2 and project B required Python V3. Then project C used V3, but needed a different version of, say, the Django package.

The [venv](https://docs.python.org/3/library/venv.html) package is now a standard part of Python to solve this problem. Long story short, it only goes part way. Many "Pure Python" libraries are in wide use, that will work on any OS. But some libraries require "binary support", meaning compiled OS specific code. The `venv` package doesn't fully solve this problem.

Because very often the binary component of a Python library is used for performance and access to hardware acceleration, this problem is very common problem in big data, machine learning, and scientific computing spaces. The developers in these fields saw the need for a holistic Python virtual environment solution.

That then is the key differentiator between an "old school" Python virtualenv and a Conda environment: comprehensive isolation of Python, installed libraries, _and_ their binary components.

Should you use Conda environments for everything in Python? No. The [other solutions](#python-packaging--in-about-100-words) have strong use cases. But for data engineering, probably. Personally, if it's a project I'll want a Jupyter notebook for, it's Mamba ftw.


### Best Practice: Don't install in the base env {#best-practice-don-t-install-in-the-base-env}

From the [Base Environment](https://mamba.readthedocs.io/en/latest/user_guide/concepts.html#base-environment) section:

> This is a legacy environment from `conda` implementation that is still heavily used.
> The base environment contains the `conda` and `mamba` installation alongside a Python installation (since `mamba` and `conda` require Python to run).
>
> `mamba` and `conda`, being themselves Python packages, are installed in the base environment, making the CLIs available in all activated environments based on this base environment.

You'll find debate on Conda environment best practices [all over the net](https://duckduckgo.com/?q=conda+environment+best+practices&t=brave&ia=web). Lets just take "leave base clean" as a given and create a new env.


## Create a "default" Conda Environment {#create-a-default-conda-environment}

My practice is to create a Conda environment called "default", fill it up with all the things I need for experiments, ideation, quick projects, etc. Then, if I find myself building a production project, I'll create and activate a unique environment for that.

Here, we'll just create "default". The procedure for any other environment is the same. After the create, we'll activate that environment and list what's installed in it.

```shell
mamba create -n default
mamba activate default
mamba list
```

and `mamba list` tells us there's nothing much there...

```shell
# packages in environment at /home/me/mambaforge/envs/default-demo:
#
# Name                    Version                   Build  Channel
```

not even a Python program, as we saw in the base environment. Isolated environments, ftw.

Let's install the latest Python, Jupyter, and Pandas, three essential for most data work.

```shell
mamba install python jupyterlab pandas
mamba list
```

and you should see plenty of installed packages in your env.

Type

```shell
jupyter-lab
```

into your terminal with the activated environment and take it from there.

For reference, the Conda [Managing environments](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#) section has you covered for a wide range of environment operations.


## Final Touches {#final-touches}

I like my default Conda environment to be ready to go at a moment's notice. I add

```shell
mamba activate default
```

to my `.bashrc`. Jupyter is always just a terminal away.
