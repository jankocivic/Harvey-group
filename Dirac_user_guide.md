# Dirac User Guide

**Note:** This document is a work in progress and may contain errors. Please email any suggestions to janko.civic@kuleuven.be.

## What is Dirac?

Dirac serves as the computer cluster for the division of quantum chemistry and physical chemistry at KU Leuven. For any questions or issues, contact Hans Vansweevelt at hans.vansweevelt@kuleuven.be.

## How to Connect

To access Dirac, use any system that supports SSH. In your terminal, type `ssh user_name@dirac.quantchem.kuleuven.be`. Ensure you are connected to campusroam. If accessing from a different network, you'll need a VPN client. Setup instructions can be found [here](https://it-support.set.kuleuven.be/support/solutions/articles/7000071607#private).

## Important Commands

Dirac utilizes the torque resource manager instead of the more common SLURM resource manager. For detailed command references, consult the manual [here](http://docs.adaptivecomputing.com/torque/6-1-0/adminGuide/torqueAdminGuide-6.1.0.pdf).

Here's a quick overview of key commands:

- `qsum`: Lists all available nodes with occupancy information.
- `qstat`: Lists all running jobs (use `qstat -u $USER` to list only your jobs).
- `qsub -q <NODE> <JOB_SH>`: Submits a job defined in `<JOB_SH>` to node `<NODE>`.
- `qdel <JOB_ID>`: Terminates a job; the job ID is displayed in `qstat` output.
- `qc`: Lists detailed information for all nodes (`qc -q` shows the corresponding queue name) 

_Bonus Tip:_ You can send a message to an user logged in on Dirac using the `write <USER_NAME>` command. An interactive prompt will appear, allowing you to type your message. After typing your message, press ENTER followed by Ctrl-C to send it.

## How to Submit a CPU Job

All nodes listed by `qsum` are CPU nodes. Unlike other clusters, Dirac requires manual node selection for job submission. For most cases, use one of the "g" nodes. Note that some "g" nodes listed in `qsum` may not be functional. To check if a node is functional and other details use the command:
`qc -q`

### Submitting a Gaussian Job

A special script facilitates submitting Gaussian16 jobs directly from gaussian input files. The script sets the CPU cores automatically. However, specify memory usage with `%mem=...` and checkpoint file location with `%chk=...`. To submit, type into the terminal:
`subg16 <NODE> <GAUSSIAN_INPUT_FILE> /temp0/user_name`

For generating a formatted checkpoint file, type into the terminal:
`/usr/local/chem/g16A03/formchk <GAUSSIAN_CHECKPOINT>`

### Other Software

For software other than Gaussian16, use the `qsub` command with a custom `<JOB_SH>` file loading necessary software. Most software is located in `/usr/local/chem/`.
To submit type into the terminal:
`qsub -q <NODE> <JOB_SH>`

Sample `<JOB_SH>` files can be found in `/home/janko/Examples`, including:
- `Amber_xTB.sh`: Example for QM/MM calculation with Amber and xTB.
- `Amber_CPU.sh`: Example for an Amber CPU job.
- `xTB.sh`: Example for an xTB calculation.
- `orca.sh`: Example for an Orca calculation

## How to Submit a GPU Job

Dirac's GPU nodes are private and don't have a scheduling system. For now, our group has 2 private GPU nodes with 2 GPUs each:
- node115: 2 x NVIDIA GeForce GTX 1080 Ti
- node120: 2 x NVIDIA Quadro RTX 6000

To access them, first connect to dirac, then SSH directly into `node115` or `node120` by typing `ssh nodeXXX`. Contact Hans Vansweevelt if you lack permission. After successful connection, the terminal prompt should start with `user_name@nodeXXX`.

Once connected, run jobs as you would on a personal computer; no `qsub` commands needed. Before running, check GPU usage with `nvidia-smi` or running processes with `ps -aux`.

To submit a GPU job, create an executable `<JOB_SH>` file, then submit with:
`nohup ./<JOB_SH> &`

The `nohup` command enables closing the terminal without stopping the job. `&` will allow you to continue use the command line as normal.

Sample `<JOB_SH>` files can be found in `/home/janko/Examples`, including:
- `Amber_GPU.sh`: Example for an Amber GPU calculation
- `gromacs.sh`: Example for a gromacs GPU calculation

The GPU nodes have 2 GPUs each and in most cases only one is used in calculations. This means that at the same time it is possible to run two independent GPU calculations. By default Amber or Gromacs will use the first GPU listed with the `nvidia-smi` command. To use the second GPU it is required to add `export CUDA_VISIBLE_DEVICES=1` to your `<JOB_SH>` file.

To terminate a running process, first identify its Process ID (PID) using the `ps -aux` command, then use the `kill <PID>` command to end it.

## Ideas to include
- How to run a memory demanding job
- How to submit to the m and p queues
- How to write a good job script (making use of the scrtach directory)
- How to manage your storage
- How to install software
