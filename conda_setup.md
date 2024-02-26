# How to setup a conda environment

**Note:** [Hans](https://www.kuleuven.be/wieiswie/nl/person/00016919) has confirmed Dirac has a Miniconda install available. This allows users to install conda environments in their home directory which is accesible on compute nodes. For more info about the use of python and conda on the VSC, see this [link](https://docs.vscentrum.be/software/python_package_management.html). 

**Note:** THIS PROCEDURE IS UNTESTED ON DIRAC. It has been used on VSC and in principle the workflow should be the same on Dirac. I have pointed out some VSC-specific steps.

## (VSC-specific) Install Miniconda

1. Get the executable for the latest Miniconda distribution :

    `$ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh`

2. Install Miniconda :


    `$ bash Miniconda3-latest-Linux-x86_64.sh -b -p $VSC_DATA/miniconda3`

   (generally advised to use `$VSC_DATA` instead of `$VSC_HOME` for installing conda environments due to storage [quota](https://docs.vscentrum.be/leuven/tier2_hardware/kuleuven_storage.html#ku-leuven-storage) - be careful since `$VSC_SCRATCH` is not permanent!)

3. Add your Miniconda install your `$PATH` by adding the following line to `.bashrc` in `$VSC_HOME` :

    `export PATH="${VSC_DATA}/miniconda3/bin:${PATH}"`

    (It might be necessary to create `.bashrc` and obviously change ${VSC_DATA} based on the installation directory of Miniconda.)

## Setup a Conda environment

**Note:** This is an example using [gmx_MMPBSA](https://valdes-tresanco-ms.github.io/gmx_MMPBSA/dev/). This workflow should also work for other conda installations, e.g. [AmberTools](https://ambermd.org/GetAmber.php).

**Note:** You may want to switch to the `mamba` solver. I have also found it to perform a lot better than the standard conda environment solver. Learn more in this [link](https://www.anaconda.com/blog/a-faster-conda-for-a-growing-community).

**Note:** at the time of writing the installation commands for dependencies of gmx_MMPBSA using conda & pip have already changed (especially version numbers of dependencies), see [installation instructions](https://valdes-tresanco-ms.github.io/gmx_MMPBSA/dev/installation/) for more info. 
1. Create the environment :

    `$ conda create -n gmxMMPBSA python=3.9`
    
    (This wil create a new environment with name gmxMMPBSA and use python3.9)

2. Activate the environment : 

    `$ source activate gmxMMPBSA`

3. **(VSC-specific)** VSC is able to use multiple nodes for a single calculation. To optimize MPI parallelisation it is advised to use `mpi4py` from the intel repository :

    `$ conda install -c intel mpi4py`

4. Install package dependencies :

    `$ conda install -c conda-forge ambertools=21.12 compilers=1.2.0 -y -q`
   
    `$ python -m pip install git+https://github.com/Valdes-Tresanco-MS/ParmEd.git@v3.4`

    (The second command will install a modified version of parmed via pip in the conda environment since we have activated it previously.)

6. (Optional - see gmx_MMPBSA installation instructions, link above) install gromacs : 

    `$ conda install -c conda-forge gromacs==2022.4 -y -q`

7. Install gmx_MMPBSA : 

    `$ python -m pip install git+https://github.com/Valdes-Tresanco-MS/gmx_MMPBSA`

Hope this helps !! - CA
