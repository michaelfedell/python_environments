# Virtual Environments

A Python Virtual Environment is a directory on your system that contains a particular version of python as well as any additional packages that are used.

This helps with a number of issues:

- messy system installation / risk of breaking core system tools
- specific version requirements differing across projects
- sharing a project's python setup with other collaborators
- ensuring that you can install and use packages in a consistent manner

The python standard docs also include [a tutorial on Virtual Environments and Packages](https://docs.python.org/3/tutorial/venv.html)

## Virtual Environment Managers

A number of solution exist for managing virtual environments and installing packages. You are likely already familiar with a few of them.

`pip` is a package manager, it handles the installation of packages and their dependencies
`venv` is an environment manager, it helps you create isolated virtual environments into which packages can be installed
`conda` is both an environment manager and a package manager and includes commands for both features

Additionally, you may come across:

- `virtualenv`: a separate tool installed by pip for creating virtual environments (replaced by python standard `venv`) ([see more](https://docs.python.org/3/library/venv.html))
- `pipenv`: a pip-based dual-function environment/package manager (similar to conda) ([see more](https://pypi.org/project/pipenv/) and [why pipenv](https://realpython.com/pipenv-guide/))
- `poetry`: a package manager with a focus on developing/building/publishing packages ([see more](https://python-poetry.org/docs/))

as well as:

- `pyenv`: a tool for managing python versions installed to your machine ([see more](https://github.com/pyenv/pyenv))
- `pipx`: a tool for installing applications (as opposed to libraries) from pip into isolated environments available globally ([see more](https://pypi.org/project/pipx/))

You can try out any of these tools (or others) and decide which works best for you, the important thing is that you decide on and make use of a virtual environment strategy. **THIS IS VERY IMPORTANT**!

## Dependency Files

There are several ways to track and control the packages installed to a particular environment. The most common (standard) way of doing so is with a `requirements.txt` file

## Conda Tutorial

You have likely used conda to install packages. However, conda is also used to create and manage virtual environments. When you install conda to your computer, you are given a "base" environment. This is activated by default based on a snippet the conda installer places in your `~/.bashrc`. If you need to activate the base environment yourself, you can simply run `conda activate`.

### Create a new environment

```bash
conda create --name tutorial python=3.7
```

### Activate your new environment

```bash
conda activate tutorial
```

### Deactivate your new environment

```bash
conda deactivate
```

## Installing packages in your new environment

### Installing packages

```bash
conda install numpy
```

### Installing packages from other `conda` channels

What if `conda install` doesn't work?! For example, the following will not work:

```bash
conda install geopy
```

This is because `geopy` is not part of the default `conda` channel. A `conda` channel is an online package repository for conda packages.

However, there exist numerous other channels that you can `conda install` from. You can search the [`conda` website](https://anaconda.org/) for channels that contain the package you are looking for.

[Search `geopy` in the conda website](https://anaconda.org/search?q=geopy). You will see that there are a number of channels you can install the package from, ordered by number of installs.

The channel with the highest number of installs is `conda-forge`, which is a popular and well-used channel. To install `geopy` via the `conda-forge` channel, run the following:

```bash
conda install -c conda-forge geopy
```

This command tells `conda` to install `geopy` from the channel (`-c`), `conda-forge`.

### Installing packages not in a `conda` channel

If you cannot locate a `conda` channel or the only channels that exist do not look well tested (i.e. have few installs), you can still use `pip`:

```bash
pip install geopy
```

> NOTE: Make sure that your active conda environment has `pip` installed, or else you may accidentally use a system-level pip which will install the package outside your virtual environment leading to much confusion

If the package does exist in a channel, however, it is preferable because `conda` takes care of the interactions of dependencies between the new package and packages already installed in the environment.

### Uninstall packages in your new environment

```bash
conda remove numpy scipy
```

## Reproducing your environment

You can reproduce your environment with a `environment.yml` file, which will list your version of Python, the conda channels to be used and the order in which to try them (i.e. install from the default channel and if the package does not exist in the default channel, install from `conda-forge`), and the list of packages with their versions. It will also include a list of the packages which were `pip install`ed rather than `conda install`ed (see more under _Listing project dependencies with `requirements.txt`_ for more on this).

### Creating `environment.yml`

To create your `environment.yml` file from a current environment, run:

```bash
conda env export > environment.yml
```

### Creating an environment using `environment.yml`

To reproduce an environment from an `environment.yml` file, run the following:

```bash
conda env create -n reproduced_env -f environment.yml
```

_Note_: this command will overwrite an `environment.yml` file already in the directory.

### Cloning an already existing environment

An `environment.yml` is useful for reproducing environments across servers, much like `requirements.txt` (discussed in a later section). However, to reproduce an environment already on the machine, you can `clone` an environment as follows:

```bash
conda create --name tutorial-clone --clone tutorial
```

> It can be nice to have a base environment with packages that you always use (such as `numpy` and `pandas`) that you can then clone and install packages into for each new project.

## Interacting with your environments

### Viewing a list of your environments

```bash
conda env list
```

### Removing an environment

```bash
conda env remove --name tutorial
```

## Listing project dependencies with `requirements.txt`

Every github repository and/or software based project using Python should have a `requirements.txt` file that includes all of the Python package dependencies of the project. This list should have no more and no less packages than are necessary to build an environment from scratch and execute the project code.

### Creating a `requirements.txt`

All Python packages necessary for a project should be added to the `requirements.txt` file using the following format:

```text
python-package-name==0.1.2
```

where `0.1.2` is the version of the Python package used to develop the project. You can use `conda list` or `pip list` to find the version numbers that you are using.

You will see in a lot of documentation that you can use

```bash
pip freeze > requirements.txt
```

to create the `requirements.txt` file. However, this freezes not only the exact versions of the packages you have explicitly installed but also the exact versions of the dependencies of those packages. However, those packages likely only have a dependency on the version being the current version or greater, e.g.:

```text
python-package-dependency-name>=1.2.3
```

but `pip freeze` will _freeze_ the dependency as:

```text
python-package-dependency-name==1.2.3
```

This more restrictive requirement could pose problems later on if you install additional packages in the environment that have a requirement such as:

```text
python-package-dependency-name>=1.2.4
```

You will run into a dependency issue if you add the new package because the `requirements.txt` will call for version 1.2.3 for `python-package-dependency-name` - even though it would be just fine with version 1.2.4.

One solution is to only add the python packages that you explicitly import in your code (manually create and manage `requirements.txt`). The other is to use `conda`.

## Testing your `requirements.txt` with `venv`

`conda` is a useful tool for environment management during development. However, it is generally not used in production. Therefore, to test that your requirements are adequate for your application, it is a good idea to use `venv`.

You can check that your `requirements.txt` works by doing the following:

```bash
python -m venv test-env

source test-env/bin/activate

pip install -r requirements.txt

deactivate
```

Then, run your code in the newly created environment and make sure it runs and passes tests.

## Resources

- [`conda` cheat sheet](http://know.continuum.io/rs/387-XNW-688/images/conda-cheatsheet.pdf)
- [`pip freeze` considered harmful](https://medium.com/@tomagee/pip-freeze-requirements-txt-considered-harmful-f0bce66cf895)
- [`conda` documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/overview.html)
- [Why you need Python environments and how to manage them with Conda](https://medium.freecodecamp.org/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c)
