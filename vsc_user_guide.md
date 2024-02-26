# Info running jobs on VSC

## Account creation 

To access the VSC, you'll have to create an account and setup SSH keys for remote login. See [link](https://docs.vscentrum.be/access/vsc_account.html) for more info.

You will have to add yourself to a specific group. Ask a team member which group you should take. I'm not going to put it here since this repo is public.

## Login to VSC and queue info

Once your account has been setup, you can login via terminal through 

`$ ssh vscXXXXX@login.hpc.kuleuven.be`

where XXXXX is your designated user number. You'll be asked to login via 2FA so make sure to have your QR scanner at hand.

Upon login, you'll get a really useful overview of the availability on both Genius and wICE (the two compute clusters of the KULeuven on VSC). To get this info if you have already logged in some time ago, use the following command : 

`$ cat /etc/motd.d/vsc_queuestate`

## Software available on the VSC

After your account creation, you will be invited to an introduction to the VSC system by VSC administrators of the KU Leuven. Be sure to follow it, it will give more details about the use of software on the VSC.

To use available software, you will have to use `module load SOFTWARE/VERSION` either in an interactive shell on a compute node or in your batch runfile. 

See [link](https://docs.vscentrum.be/software/software_stack.html#using-the-module-system) for more info on using already available software on the VSC.

## Submitting jobs to the VSC

### Copying files to and from the cluster

If you look at the [documentation](https://docs.vscentrum.be/data/transfer.html) of the VSC, they'll only mention using `scp` or `sftp` to transfer files from and to the cluster.

However, I've noticed that using `rsync` also works and makes it somewhat easier if you're copying multiple folders with multiple files simultaneously. 

My script to transfer folders from my local pc to the cluster looks like this :

```
#!/bin/bash

folders$(ls -d some_dir_available_in_current_dir/*/)

for folder in $folders ; do
cat << eof > file_list.txt
$folder/some_folder/
$folder/another_folder/
eof

rsync -av --relative --progress --partial-dir=/tmp --files-from=file_list.txt -r . vscXXXXX@login.hpc.kuleuven.be:$VSC_DATA
```
a couple of notes : 
- the folders to be synchronized have to be mentioned in the `cat << eof` block
- make sure that every directory path ends with a `/` otherwise files won't get copied
- `vscXXXXX`has to be replaced with your own vsc username
- `$VSC_DATA has to be an absolute path, this starts with `/data/leuven/....` so be sure to modify this

To retrieve files from the cluster to my local system, I use a simple oneliner on my local machine:

`$ rsync -av --progress -r --partial-dir=/tmp vscXXXXX@login.hpc.kuleuven.be:$VSC_DATA/some_dir_available_in_current_dir/ ./some_dir_available_in_current_dir/`

### Example submit script

VSC uses `slurm` as scheduler and resource manager. **Note:** Even if resources may seem available, `slurm`might be freeing up for a big job. 

Jobs are submitted via `srun` or `sbatch`. See [link](https://docs.vscentrum.be/jobs/running_jobs.html) for more info. I typically provide everything in the batch script file and queue with `sbatch runfile.slurm`

An example `runfile.slurm` for an MMPBSA.py calculation with AmberTools looks like this : 

```
#!/bin/bash -l
#SBATCH --account=lp_GROUP
#SBATCH --time=4:00:00
#SBATCH --clusters=genius
#SBATCH --partition=batch
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=36

source activate AmberTools

cat << eof > mmgbsa.in
Sample input file for GB calculation
&general
startframe = 1, 
endframe = 5000,
interval = 1,
keep_files = 0,
use_sander = 1,
verbose = 2,
/
&gb
igb = 5,
saltcon = 0,15,
/
eof

#complex_dry
ante-MMPBSA.py -p complex_solv.prmtop -c complex_dry.prmtop -s "!(:1-128)" --radii=mbondi2
# receptor_dry
ante-MMPBSA.py -p complex_solv.prmtop -c receptor_dry.prmtop -s "!(:1-127)" --radii=mbondi2
# ligand
ante-MMPBSA.py -p complex_solv.prmtop -c ligand_dry.prmtop -s "!(:128)" --radii=mbondi2

mpirun -np 72 MMPBSA.py.MPI -O \
	-i mmgbsa.in \
	-sp complex_solv.prmtop \
	-cp complex_dry.prmtop \
	-rp receptor_dry.prmtop \
	-lp ligand_dry.prmtop \
	-y md_trajectory.nc 
```

a couple of notes : 
- the `--account` has to be specified to charge credits. This is the same account you chose during the setup of your account.
- your job will end if you exceed the time set with `--time`. Try to choose wisely...
- KU Leuven VSC has two clusters: [Genius](https://docs.vscentrum.be/leuven/tier2_hardware/genius_hardware.html) and [wICE](https://docs.vscentrum.be/leuven/tier2_hardware/wice_hardware.html). Check the links for the hardware in the partitions of both clusters and the maximum number you can assign to `--ntasks-per-node`
- this script assumes you have AmberTools installed in a conda environment called AmberTools. Additionally, both `complex_solv.prmtop` and `md_trajectory` have to be present in the current working directory

Hope this helps!! - CA