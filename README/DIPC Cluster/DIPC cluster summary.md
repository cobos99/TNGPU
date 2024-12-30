# DIPC Cluster Summary

**Author:** Jesús Cobos Jiménez

See [DIPC official documentation](https://scc.dipc.org/docs/) for complete information.

## Logging in into the cluster

There are [three clusters](https://scc.dipc.org/docs/general/overview/) available. We are mostly interested on the `Hyperion` cluster, which is the one with the most powerful GPUs available. `Atlas EDR` cluster also has some nodes equipped with GPUs, but there are fewer of them. If long queues are found in `Hyperion`, `Atlas EDR` should be used for prototyping. The addresses of each of the clusters are the following

| Name      | Address             |
|-----------|---------------------|
| Hyperion  | hyperion.sw.ehu.es  |
| Atlas EDR | atlas-edr.sw.ehu.es |
| Atlas FDR | atlas-fdr.sw.ehu.es |

To log into the `Hyperion` cluster, one uses `ssh` from terminal.

```bash
ssh <username>@hyperion.sw.ehu.es
```

**Note** that on first log in you will be prompted to change your default password ([See here](https://scc.dipc.org/docs/access/accounts/#modifying-your-password)).

Here, we have access to a Linux shell, which should not be used to directly execute our jobs, but to manage our files and submit our jobs using the job manager `SLURM`.

## Uploading/Downloading files to the cluster

Once logged in any of the clusters, we have access to two main roots of the file system, namely `/dipc/username/` and `/scratch/username/`. The first is path `/dipc/` is where we are intended to save large files that we want to save for long as it is backed up regularly. This root is shared among all the clusters, using it can be useful to store scripts that we will then submit to many clusters. The second path points to high performance storage, and it is from where we should submit our jobs. This root is NOT shared among clusters. My recommendation is to use temporarily `/scratch/` when submitting jobs and then downloading the results directly to your computer.

The best way to manage the files on the cluster is to use [FileZilla](https://filezilla-project.org/), which is a very convenient FTP file explorer. Configuring FileZilla to access the cluster is simple.

### 1. Open the server menu in FileZilla

![Image](/README/DIPC%20Cluster/Summary%20figures/FTP%20server%20add%20button.png)

### 2. Configure SFTP access

![Image](/README/DIPC%20Cluster/Summary%20figures/FTP%20server%20credentials.png)

### 3. Navigate through the cluster files

![Image](/README/DIPC%20Cluster/Summary%20figures/Complete%20FileZilla%20view.png)

The right column show the files and folder on the cluster while the left one shows the local files. Files are uploaded/downloaded by drag and drop.

## Running jobs

Jobs are submitted via [SLURM](https://scc.dipc.org/docs/jobs/slurm/). `SLURM` is the software that manages resource allocation among all the users of the cluster. To submit a job, the first step is to create a folder where to store all the scripts and results in the `/scratch/user/` directory. Once this folder is created and all the program script are uploaded a `SLURM` script must be created to execute our program. A `SLURM` script is a `bash` script with an additional header in which the execution options (resource allocation, timeout, nodes...) are set. The next is an example of `SLURM` script with me most common options. This script does **not** include the options for GPU usage, which are presented next. The execution commands may vary from our particular task.

```bash
#!/bin/bash

# ------------------------------- #
#          SLURM OPTIONS          #
# ------------------------------- #

#SBATCH --job-name=9-27-tiheisenberg        # Job name
#SBATCH --output=9-27-tiheisenberg.out      # Output file
#SBATCH --error=9-27-tiheisenberg.err       # Error file
#SBATCH --mail-user=<email>                 # Email address to send notifications
#SBATCH --mail-type=END                     # Email notifications (BEGIN, END, FAIL, ALL)
#SBATCH --time=48:00:00                     # Time limit DD-HH:MM:SS
#SBATCH --partition=general                 # Partition (Options in hyperion: general/preemption/preemption-gpu)
#SBATCH --qos=long                          # Quality of Service (https://scc.dipc.org/docs/systems/hyperion/jobs/slurm/#qos-and-partitions)
#SBATCH --nodes=1                           # Number of nodes
#SBATCH --ntasks-per-node=1                 # Number of tasks (Individual instances of the job) per node
#SBATCH --cpus-per-task=48                  # Threads per task
#SBATCH --mem=128G                          # Maximum RAM (Only a number is considered MB, adding G at the end changes to GB)
#SBATCH --nodelist=hyperion-[208-224]       # Nodes to be used

# ------------------------------- #
#        EXECUTION COMMANDS       #
# ------------------------------- #

# Load necessary modules
module load Julia

# Run the Julia script
srun julia --project=./env -t auto tiheisenberg.jl 9 27 21 -p
```

Some useful links to understand part of the options above are the following:

- [Quality of service & Partitions on Hyperion](https://scc.dipc.org/docs/systems/hyperion/jobs/slurm/#qos-and-partitions)

- [Nodes description](https://scc.dipc.org/docs/systems/hyperion/overview/#specifications)

#### Running on GPUs

To use GPUs one should add the following options

```bash
#SBATCH --gres=gpu:2             # Number of GPUs to be used 
#SBATCH --constraint=rtx3090     # Model of GPU to be used
```

The list of nodes equipped with GPUs and the respective GPU models can be found in the following [link](https://scc.dipc.org/docs/systems/hyperion/jobs/slurm/gpus/#hyperion-system). In another future README we will describe how to write programs for GPUs.

