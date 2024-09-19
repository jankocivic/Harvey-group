# Dirac User Guide

**Note:** This document is a work in progress and may contain errors. Please email any suggestions to janko.civic@kuleuven.be. While official documentation is available [here](https://qcpci.quantchem.kuleuven.be/), this guide complements it by providing more detail on the basics for inexperienced users.

## Contents
1. [What is Dirac](#what-is-dirac)
2. [How to Connect](#how-to-connect)
3. [Dirac Nodes](#dirac-nodes)
4. [Storage on Dirac](#storage-on-dirac)
5. [Important Commands](#important-commands)
6. [How to Submit a Job](#how-to-submit-a-job)
    - [Submitting an Orca Job](#submitting-an-orca-job)
    - [Submitting a Gaussian Job](#submitting-a-gaussian-job)
    - [Submitting a Turbomole Job](#submitting-a-turbomole-job)
    - [Submitting an Amber Job](#submitting-an-amber-job)
    - [Submitting an xTB Job](#submitting-an-xtb-job)
    - [Submitting a Python Job](#submitting-a-python-job)
    - [Other Software](#other-software)
7. [Ideas to Include](#ideas-to-include)

## What is Dirac

Dirac serves as the computer cluster for the division of quantum chemistry and physical chemistry at KU Leuven. For any questions or issues, contact Hans Vansweevelt at hans.vansweevelt@kuleuven.be.

## How to Connect

To access Dirac, use any system that supports SSH. In your terminal, type `ssh user_name@dirac.quantchem.kuleuven.be`. Ensure you are connected to campusroam. If accessing from a different network, you'll need a VPN client. Setup instructions can be found [here](https://it-support.set.kuleuven.be/support/solutions/articles/7000071607#private).

## Dirac Nodes

Dirac is a high-performance computing system consisting of numerous interconnected computers, or nodes, each with its own set of resources including CPU cores, GPUs, memory, and disk space. These nodes have different purposes within the cluster.

One node, called "dirac", serves as the login node. This is the entry point for all users connecting to the server. The login node's primary function is to handle basic file system operations and submit job requests to other nodes. While capable of performing various tasks, users should avoid running computation or memory-intensive processes on the login node, as it's shared among all connected users and could become slow or unusable if overloaded.

For demanding tasks, users should utilize dedicated computation nodes by submitting jobs to them. The `qc` command lists all available nodes. These computation nodes are organized into queues based on their purpose. The `qsum` command displays a list of all available queues:

- **g-queues:** General-purpose queues where each queue corresponds to one node (e.g., g10 corresponds to node10) for jobs with moderate resource requirements.
- **p-queues:** High-performance queues where each queue corresponds to one node (e.g., p27 corresponds to node27) for jobs needing substantial memory or disk space.
- **d-queues:** GPU-enabled queues where each queue corresponds to one node (e.g., d33 corresponds to node33) for tasks requiring GPU resources, such as classical molecular dynamics simulations or machine learning model training. **They should not be used for tasks not requiring a GPU.**
- **m-queues:** Very different from g-, p- and d-queues. They are groups of several nodes (e.g., m0311 includes node03 to node11). Jobs submitted to these queues run on the first available node in the group. This setup offers convenience by eliminating the need to check individual node availability. Additionally, requesting a single CPU core in an m-queue will occupy only that core on a specific node, leaving other cores available for different jobs.

The status of m-queues and the occupancy of each node in the group can be checked using the `qm` command. Users should still select m-queues based on job requirements: m0311, m1219, and m2022 for general-purpose jobs (same as g-queues), and m131138 and m139146 for memory and disk space-intensive jobs (same as p-queues).

To maintain system efficiency and fairness, users should try to responsibly select the appropriate queue for their intended job and minimize computational resource usage. This approach ensures optimal performance for all users of Dirac.

## Storage on Dirac

There are two types of data storage to be aware of:

1. **Personal Home Folder** (`/home/<USER_NAME>/`):  
    This is where you should store all your important files. It is accessible from the login node and every computation node. You have up to 400 GB of storage available in this folder. To check how much space you have left, use the command `quota -s`. The contents of this folder are backed up daily around 9 PM. If you accidentally delete a file, contact Hans Vansweevelt for assistance with recovery.
2. **Temporary Storage** (`/temp0/<USER_NAME>/`):  
    Many software programs generate large temporary files that can exceed the limits of your home folder. To address this, each computing node has a temporary folder located at `/temp0/<USER_NAME>/`.  This folder is not directly accessible from the login node, but can be accessed by job scripts submitted to a node. Files stored on it are not backed up and are automatically deleted after a period of time. You can check the storage limit for this folder on each node using the `qc` command. When submitting a job via a script, it's recommended that the script copies all input files to `/temp0/<USER_NAME>/`, performs the calculations there, and at the end, only transfers the necessary output files back to your home folder. This minimizes the load on your home storage.

**Note for Molecular Dynamics Users:**

If you are running classical molecular dynamics simulations, be cautious with storage, as trajectory files can quickly exceed your home folder limits. To manage this, consider reducing the frequency of snapshot writing to the trajectory file or stripping unnecessary atoms, such as solvent atoms, from the system.

## Important Commands

Dirac utilizes the torque resource manager instead of the more common SLURM resource manager. For detailed command references, consult the manual [here](http://docs.adaptivecomputing.com/torque/6-1-0/adminGuide/torqueAdminGuide-6.1.0.pdf).

Here's a quick overview of key commands:

- `qsum`: Lists all available queues with occupancy information.
- `qm`: Shows the occupancy of each node in the m-queues (green bars indicate occupied CPU cores).
- `qc`: Lists detailed information for all nodes (`qc <QUEUE>` shows detailed information of a specific queue, `qc -q` shows detailed information of all queues)
- `qsub -q <QUEUE> <JOB_SH>`: Submits a job defined in `<JOB_SH>` to queue `<QUEUE>`. More details are provided further in the guide.
- `qstat`: Lists all running jobs (use `qstat -u $USER` to list only your jobs).
- `qdel <JOB_ID>`: Terminates a job; the job ID is displayed in `qstat` output.

_Bonus Tip:_ You can send a message to an user logged in on Dirac using the `write <USER_NAME>` command. An interactive prompt will appear, allowing you to type your message. After typing your message, press ENTER followed by Ctrl-C to send it.

## How to Submit a Job

When working on a personal computer, tasks are executed directly in the terminal. However, on a computer cluster system like Dirac, jobs should not be run on the login node. Instead, they must be submitted to computation nodes. A job is typically just a Bash script (beginning with `#!/bin/bash`) that contains all the commands needed to complete the task. This may include navigating to the correct directories, configuring environment variables for specific software, and initiating calculations. To run this script on a computation node, you can submit it to the job queue using the `qsub` command. The cluster will then execute the commands sequentially on an assigned node. Keep in mind that when a job runs on a computation node, the starting directory will be your home directory, not the directory from which you submitted the job. Therefore, it's typically necessary to include commands in your job script to navigate to the appropriate input directory.

To submit a job using `qsub`, the basic syntax is `qsub -q <QUEUE> <JOB_SH>`, where `<JOB_SH>` is your job script, and `<QUEUE>` specifies the queue. However, this basic command doesn’t allow you to control the number of CPU cores or the amount of memory allocated to your job. By default, the number of cores assigned to each queue can be viewed using the `qc -q` command under the "Default ppn" (processors per node) column.

For more control over resource allocation, you can use the following: `qsub -q <QUEUE> -l nodes=1:<QUEUE>:ppn=<NUM_CPUS>,mem=<MEMORY>gb <JOB_SH>`. This submits your job with the requested number of CPU cores and memory in GB. This approach is recommended as it allows you to request only the resources you need, ensuring that unused resources are available for other users. Available resources for each node can be seen in the output of the `qc` command.

Additional useful options to `qsub` include `-N <NAME>`, which assigns a name to your job, and `-e <OUTPUT_FILE> -j eo`, which redirects both the standard error and output to a file named `<OUTPUT_FILE>` in the submission directory.

Be aware that there are limits on the number of jobs a user can submit and the maximum runtime for each job. Additionally, not all queues are accessible to all users. You can view the queues you have access to using the `qsum -u` command or by checking the output of `qc -q`, which also shows the time limit (walltime) for each queue. PhD students and PostDocs can submit jobs to up to five queues simultaneously, while undergraduate and master students are limited to submitting to three queues at a time.

Running a job with specific software can be challenging, so the following sections provide detailed instructions for some of the commonly used software.

### Submitting an Orca Job

To simplify the submission of Orca jobs, the process of writing job scripts and submitting them can be automated using the `suborc6` script, which allows you to submit Orca jobs directly from input files without the need for separate job scripts. To submit a job, use the command: `/home/janko/Scripts/suborc6 --input <input_file> [--memory <memory>] [--cpus <cpus>] [--queue <queue>] [--name <name>] [--gbw <gbw_file>]`. The only mandatory argument is the input file, which must have an `.inp` extension. Optional arguments allow you to customize memory allocation, CPU count, queue, and job name. If not provided, the script defaults to submitting a job named `orc_job` to the `m0311` queue, using 1 CPU and 4 GB of memory. You can also provide a relative path to a gbw file used for the initial guess. The script automatically adjusts cores and memory by adding `%pal` and `%maxcore` blocks to the input file and sets the scratch directory to `/temp0/$USER`, where a uniquely named folder is created and deleted after job completion. During job execution, you can monitor the ORCA output file (with the `.out` extension) in the submission directory. Any standard output or error messages are logged in a file with the `-batch.out` extension, also located in the submission directory. The new gbw file is saved in the submission directory and has the same base name as the input file (the `.inp` extension is replaced with `.gbw`).

In case of unexpected behaviour, the generated `.job` file can be analysed, and the suborc6 script can be copied and modified to meet specific requirements.

### Submitting a Gaussian Job

To simplify the submission of Gaussian16 jobs, the process of writing job scripts and submitting them can be automated using the `subgau16` script, which allows you to submit Gaussian jobs directly from a Gaussian input file without the need for separate job scripts. To submit a job, use the command: `/home/janko/Scripts/subgau16 --input <input_file> [--memory <memory>] [--cpus <cpus>] [--queue <queue>] [--name <name>] [--chk <chk_file>]`. The only mandatory argument is the input file, which must have an `.inp` extension. Optional arguments allow you to customize memory allocation, CPU count, queue, and job name. If not provided, the script defaults to submitting a job named `gau_job` to the `m0311` queue, using 1 CPU and 10 GB of memory. You can also provide a relative path to a chk file used for the initial guess. The script automatically adjusts cores and memory by adding `%Mem` and `%NProcShared` lines to the input file and sets the scratch directory to `/temp0/$USER`, where a uniquely named folder is created and deleted after job completion. During job execution, you can monitor the Gaussian output file (with the `.log` extension) in the submission directory. Any standard output or error messages are logged in a file with the `-batch.out` extension, also located in the submission directory. The new checkpoint file is saved in the submission directory and has the same base name as the input file (the `.inp` extension is replaced with `.chk`).

In case of unexpected behaviour, the generated `.job` file can be analysed, and the suborc6 script can be copied and modified to meet specific requirements.

If you encounter a "galloc: could not allocate memory" error in your calculation, try rising the specified memory. See an explanation of why the error occurs [here](https://docs.alliancecan.ca/wiki/Gaussian_error_messages#galloc:_could_not_allocate_memory). 

For generating a formatted checkpoint file, from the login node use the command:
`/usr/local/chem/g16A03/formchk <GAUSSIAN_CHECKPOINT>`

### Submitting a Turbomole Job

The `subtm7.8` script simplifies the submission of Turbomole 7.8 jobs directly from a Turbomole job folder, which differs from Gaussian and Orca as Turbomole jobs are defined by an entire input folder rather than a single input file. To prepare the jobs, use the `define` module, which is not computationally intensive and can be run directly from the Dirac login node. Start by loading the Turbomole environment with `source /usr/local/chem/turbomole7.7.1/Config_turbo_env` and then run `define` by typing `define` in the terminal. Detailed instructions for preparing input files are available in the Turbomole manual located at `/home/janko/Manuals`. Once the input files are ready, submit the job using the `subtm7.8` script with the command `/home/janko/Scripts/subtm7.8 --modules <module1> <module2> ... [--memory <memory>] [--cpus <cpus>] [--queue <queue>] [--name <name>]`. The only mandatory arguments are the Turbomole modules you want to run, such as `ridft` and `escf` or `dscf` and `ricc2`. Optional arguments allow you to specify memory, the number of CPUs, the queue, and the job name. If these options are not provided, the script defaults to submitting a job named `tm_job` to the `m0311` queue with 1 CPU and 4 GB of memory. The script automatically adjusts memory settings by modifying the `$maxcor` block in the control file and sets the scratch directory to `/temp0/$USER`, where a uniquely named folder is created and deleted after the job completes. 

In case of unexpected behavior, you can analyze the generated `.job` file. The `subtm7.8` script can also be copied and modified to suit your specific needs.

### Submitting an Amber Job

Amber22 is available on all the nodes, while Amber24 is available on all the nodes besides the GPU nodes corresponding to the d-queues, but this is expected to change in the near future.

To submit an Amber job to one of the computation nodes you will need to write a job script which loads in Amber and potentially other necessary external software, navigates to the submission directory and contains the command to start an amber calculation. All necessary input files should be in the submission directory. Submit the script with the `qsub` command. In the following sections example job scripts will be shown for different types of calculations.

#### CPU Job
If you perform a calculation with Amber which only uses the CPU use `sander` as the MD driver and submit to the  g-queues or m0311, m1219, and m2022. 

Submission script Amber24:
```
#!/bin/bash
export LD_LIBRARY_PATH=/usr/local/OpenBLAS/lib:/usr/local/chem/xtb-6.7.1/lib:$LD_LIBRARY_PATH
source /usr/local/chem/amber24/amber.sh
cd <INP_DIR>

sander ...
```

Submission script Amber22:
```
#!/bin/bash
source /usr/local/chem/amber22/amber.sh
cd <INP_DIR>

sander ...
```

#### GPU Job
Dirac has four GPU nodes (their corresponding queues begin with d). It is important that you submit the submission script to one of the d-queues and use `pmemd.cuda` as the MD driver. Since each node in the d-queues contains two GPUs you can run two parallel jobs, each on one GPU by setting the `CUDA_VISIBLE_DEVICES` environment variable. `export CUDA_VISIBLE_DEVICES=0` to use the first GPU and `export CUDA_VISIBLE_DEVICES=1` for the second GPU.

Submission script Amber22:
```
TO DO
```

#### Amber + xTB
QM/MM simulations with xTB are not able to use the GPU. Use `sander` as the MD driver and submit to the  g-queues or m0311, m1219, and m2022.

Submission script Amber24:
```
#!/bin/bash
ulimit -s unlimited
export LD_LIBRARY_PATH=/usr/local/OpenBLAS/lib:/usr/local/chem/xtb-6.7.1/lib:$LD_LIBRARY_PATH
source /usr/local/chem/amber24/amber.sh
source /home/janko/Scripts/set_environment_xtb.sh

cd <INP_DIR>

sander ...
```
#### Amber + Orca
QM/MM simulations with Orca are not able to use the GPU. Use `sander` as the MD driver and submit to the  g-queues or m0311, m1219, and m2022.

Submission script Amber24:
```
#!/bin/bash
export LD_LIBRARY_PATH=/usr/local/OpenBLAS/lib:/usr/local/chem/xtb-6.7.1/lib:$LD_LIBRARY_PATH
source /usr/local/chem/amber24/amber.sh
ORCA=/usr/local/chem/orca6
MPI=/usr/local/openmpi-4.1.1-gcc-10.3.0
export PATH=$ORCA/bin:$MPI/bin:$PATH
export RSH_COMMAND=/usr/bin/ssh
export OMP_NUM_THREADS=16
export LD_LIBRARY_PATH=$MPI/lib:$ORCA/lib:$LD_LIBRARY_PATH

cd <INP_DIR>

sander ...
```

### Submitting an xTB Job
Like Turbomole, xTB uses entire input directories instead of conventional input files. Once all the necessary files are in the input directory, submit a job script using `qsub` to the g-queues or the m0311, m1219, and m2022 queues.

Submission script:
```
#!/bin/bash
ulimit -s unlimited
export LD_LIBRARY_PATH=/usr/local/OpenBLAS/lib:/usr/local/chem/xtb-6.7.1/lib:$LD_LIBRARY_PATH
source /home/janko/Scripts/set_environment_xtb.sh

cd <INP_DIR>

xtb ...
```


### Submitting a Python Job

To run Python scripts, it's recommended to use an Anaconda environment. One the login node, create one with `conda create --name <ENV_NAME>`, activate it using `conda activate <ENV_NAME>`, and install packages via `conda install <PACKAGE_NAME>`. The environments are stored in the hidden `.conda` folder in your home directory.

To execute a Python script `<YOUR_SCRIPT>.py` located in `<SUBDIR_PATH>` on a computation node, your job submission script should look like this:

```
#!/bin/bash
python3path=/home/<USER_NAME>/.conda/envs/<ENV_NAME>/bin
export PATH=$python3path:$PATH

cd <SUBDIR_PATH>

python <YOUR_SCRIPT>.py
```

If your script uses multithreaded packages (e.g., `rdkit`, `numpy`, `sklearn`), add `export OMP_NUM_THREADS=<NUM_THREADS>` to the job submission script. Submit the script to a queue with `qsub`. 

### Other Software

For other software you will need to write your own `<JOB_SH>` submission script. A list of available software can be seen [here](https://qcpci.quantchem.kuleuven.be/software/). On a node, the available software is usually located in the `/usr/local/chem/` directory.

## Ideas to Include
- How to write a good job script (making use of the scrtach directory)
- How to install software
