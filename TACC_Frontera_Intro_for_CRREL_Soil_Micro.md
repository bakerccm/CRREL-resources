# TACC Frontera Intro for CRREL Soil Micro

**Note:** Refer to the [TACC User Guide](https://frontera-portal.tacc.utexas.edu/user-guide/) for more detailed and up-to-date information. Arrangements on TACC Frontera may change over time. The advice in this quick start guide may be useful for other HPC clusters but it is *not* guaranteed. Different clusters are set up differently or may have different policies on how to do things.

**Tip:** Use a text editor, *not* a word processor like Microsoft Word. You can spend a lot of time chasing down errors caused by invisible formatting characters or subtle changes to your code introduced by Word. (e.g. if Word replaces plain quotation marks like "" with styled ones like “”, these will probably not work the same!) Some suggestions:

 - [Atom](https://atom.io) (cross-platform)
 - [BBedit](https://www.barebones.com/products/bbedit/) (Mac)
 - [Notepad++](https://notepad-plus-plus.org) (Windows) (?)
 - [Komodo](https://www.activestate.com/products/komodo-ide/downloads/edit/) (cross-platform) (?)
 - Command line options -- e.g. nano, Vim

## Logging on

Connect to the Frontera login nodes using `ssh`, e.g.
```bash     
ssh -X your_username@frontera.tacc.utexas.edu
```
You will be asked to provide your password and a passcode from your authenticator app.

## Running simple commands on the login nodes

Compute- and memory-intensive jobs (e.g. running `R`) should not be run on the login nodes. Small tasks, however, are fine. For example, simple things like listing the files in a directory, creating a new folder, moving a bunch of files from one folder to another, etc are OK. Some useful Linux commands might include:

```bash
ls # list files and folders in the current directory
mkdir newdir # make a new folder called newdir
cd newdir # change into the folder called newdir
rm my_old_file.txt # delete my_old_file.txt
which [command] # list the location where [command] will be found if it is run
pwd # print the full path to the current folder
```

## Getting help on commands

If you need help on a particular command, see if there is a manual available:
```bash
man [command]
```
Alternatively, try `[command] -h` or `[command] --help` to see if there is a help file.

Note that commands and programs can vary subtly between different HPC clusters, and may even be customized by the HPC system administrators. So if in doubt (and especially if you are getting a weird error message) check the documentation directly on TACC Frontera if possible. It might be the case that, for example, an option that was available to you on a different system is not available to you on Frontera, or is specified slightly differently.

## Interacting with slurm

Frontera uses the `slurm` job scheduler. Anyone wanting to run a compute job on the cluster submits the job to `slurm` to be queued up and run when the job's turn comes. The [TACC user guide](https://frontera-portal.tacc.utexas.edu/user-guide/running/) provides more guidance on how it works. There are several commands that allow you to view information about jobs and system resources, and to interact with the scheduler, including the following.

Note that each of these commands has many options available. You can use `man [command]` to find out more about the options.

```bash
# submits a file to slurm containing your job commands
sbatch [file]
# optionally, you can include arguments that override values in
# your file (or avoid the need to specify values in the file)
# e.g. this sends your job to the development partition
sbatch -p development [file]
```

```bash
# displays information about jobs in the slurm scheduling queue
squeue
# but in practice you probably want to restrict the jobs displayed to some subset
# that you're interested in (rather than showing all the jobs on the cluster)
# so for example this shows only the jobs that belong to you:
squeue -u $(whoami)
# or customize the display format to show whatever info you want about the jobs
# e.g. this is the command that I use on Frontera to show info about my own jobs
squeue -u $(whoami) -o "%.10i %.9P %.24j %.2t %.10M %.5D %.4C %.18R %E"
# this is clearly not practical to just remember and type in, so you might like
# to save the command somewhere to paste in, or you could save it as an 'alias'
# e.g. this is how I save my preferred squeue command:
alias sq='squeue -u $(whoami) -o "%.10i %.9P %.24j %.2t %.10M %.5D %.4C %.18R %E"'
# after running this alias command, I can simply type
sq
# and hit enter, and it executes the whole squeue command with formatting
# as if I had typed it all out at the command prompt.
```

```bash
# cancel job number jobID (you can find the jobID for a running job using squeue)
scancel [jobID]
```

```bash
# displays information about slurm nodes and partitions
sinfo
# but in practice you probably want to restrict the output depending on what you
# want to know e.g. if you were having problems running jobs on the development
# partition, you might check the status of the partition using something like this
sinfo -p development --summarize
```

```bash
# displays accounting data for jobs and job steps in the slurm log
# by default on Frontera, this command only shows jobs that belong to you
sacct
```

## Storage on Frontera

The storage options available are described in the [TACC User Guide](https://frontera-portal.tacc.utexas.edu/user-guide/files/#table-2-frontera-file-systems).

Currently you have three main storage spaces on Frontera:
 1. your home directory (go there by typing `cd $HOME` or `cd ~` or `cdh`);
 2. your work directory (go there by typing `cd $WORK` or `cdw`); and
 3. scratch space (go there by typing `cd $SCRATCH` or `cds`).

`$HOME` is only 25GB and is slow (compared to `$SCRATCH`), but it is backed up and not purged (i.e. system admins don't periodically clean up files). `$WORK` is 1TB and is slow-medium speed (compared to `$SCRATCH`). Unlike `$HOME`, it is not backed up, but it is also not purged. `$SCRATCH` is effectively unlimited and is fast, BUT it is not backed up AND it is subject to purge for files older than 10 days. i.e. if you run some computations and then leave the output on `$SCRATCH` for two weeks, it may be deleted!

So these spaces are good for different things. You might use `$HOME` to save configuration files, small datasets, install some software like conda etc. But you probably wouldn't use it for large computational outputs, or to store easily replaceable data (e.g. reference databases that you can just download from the internet). Sometimes it makes sense to store the final output of a pipeline here, if the files are small enough, since it's backed up. `$WORK` might be useful for larger datasets (make sure they're backed up somewhere else though), reference databases that you can easily download, or computational outputs that you need to save for more than a few days. `$SCRATCH` is great for intermediate files that get generated while your pipeline runs. One strategy you might use is to operate out of `$SCRATCH` and then just transfer any files you need to keep back to `$WORK` at the end.

Note the use of environment variables ($HOME, $WORK, $SCRATCH) to access your directories on Frontera. It can be tempting to use the actual file paths for your directories (e.g. my `$HOME` is actually at `/home1/08186/cbaker`), but you can run into problems if your directories get moved, which can happen e.g. when hardware gets upgraded. By using the environment variables you don't need to worry about this because the variables will get updated any time there is a need to move your folders.

## Transferring files to and from your laptop

One option to transfer files from your laptop to Frontera is to use the command line program `scp`:

```bash
# scp transfer a single file from the Desktop on your laptop to your $HOME folder on Frontera
# N.B. execute this in a new shell on your laptop, not in a shell that is logged in to Frontera
scp ~/Desktop/some_code.R your_username@frontera.tacc.utexas.edu:\$HOME
# scp transfer a whole folder of files from your laptop to your $HOME on Frontera
scp -r ~/Desktop/your_folder/* your_username@frontera.tacc.utexas.edu:\$HOME
```

You can also use `scp` to transfer files from Frontera to your laptop by swapping the order of the two file paths:

```bash
# scp transfer a single file from Frontera to your laptop
# N.B. execute this in a new shell on your laptop, not in a shell that is logged in to Frontera
scp your_username@frontera.tacc.utexas.edu:\$HOME/some_sequences.fasta ~/Desktop
# scp transfer a whole folder of files from Frontera to your laptop
scp -r your_username@frontera.tacc.utexas.edu:\$HOME/some_folder/\* ~/Desktop
```

In each case, you will be prompted to enter your password and your passcode before the transfer takes place.

Other command-line options you might see people using include `ftp`, `sftp` and `rsync`. In most cases, any of these programs will carry out your file transfers just fine.

## Other file transfer options

If you prefer a graphical interface for your file transfers, programs you might consider include [FileZilla](https://filezilla-project.org), [WinSCP](https://winscp.net/eng/download.php) (for Windows) and [Cyberduck](https://cyberduck.io).

Some clusters allow you to map storage as drives on your laptop. I don't think this option is available on Frontera.

Frontera can also be used with [Globus](https://portal.tacc.utexas.edu/tutorials/globus), which might help if you are transferring datasets between Frontera and a sequencing center or another HPC cluster.

Frontera interfaces with Google Cloud, Amazon Web Services and Microsft Azure. The [TACC user guide](https://frontera-portal.tacc.utexas.edu/user-guide/cloud/) provides detailed instructions on how these connections can be set up.

## Downloading files from the internet

Small file transfers can be executed directly on the login nodes. For example, you could use the program `wget` to download files from a URL:

```bash
wget https://raw.githubusercontent.com/bakerccm/ailaoshan/main/code/Ailaoshan_gis.R
```

`curl` is another option with slightly different syntax:

```bash
curl https://raw.githubusercontent.com/bakerccm/ailaoshan/main/code/Ailaoshan_coinertia_etc.R --output Ailaoshan_coinertia_etc.R
```

It may be better to carry out transfers of many files, or large files, using the compute nodes. For example, using an interactive job:

```bash
# launch interactive compute job on Frontera
# (this is a custom command on Frontera so it won't work on other clusters)
idev

# as an example, downloads in parallel the first 10 files from the permafrost thaw metagenomic dataset
# (see https://www.ncbi.nlm.nih.gov/sra?linkname=bioproject_sra_all&from_uid=542925 for all files)
for sra in SRR15048733 SRR15048734 SRR15048735 SRR15048736 SRR15048737 SRR15048738 SRR15048739 SRR15048740 SRR15048741 SRR15048742
do
    wget --quiet https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos3/sra-pub-run-25/${sra}/${sra}.1 &
done
wait
```
