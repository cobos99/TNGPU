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
#SBATCH --qos=long                          # Quality of Service
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

Some options in the script above can just be removed (Eg. `--nodelist` and others). At least the options concerning number of jobs and nodes, memory, timeout, partition and quality of service should always be set.

#### Running on GPUs

To use GPUs one should add the following options

```bash
#SBATCH --gres=gpu:2             # Number of GPUs to be used 
#SBATCH --constraint=rtx3090     # Model of GPU to be used
```

The list of nodes equipped with GPUs and the respective GPU models can be found in the following [link](https://scc.dipc.org/docs/systems/hyperion/jobs/slurm/gpus/#hyperion-system). In another future README we will describe how to write programs for GPUs.

Once the `SLURM` script is ready, a job is submitted using the following command in the terminal

```bash
sbatch <PATH_TO_SLURM_SCRIPT>
```

The following message will be prompted on the terminal after a successful submission

```
Submitted job <JOB_ID>
```

## Managing jobs

To check the status of submitted jobs, the following command is used

```
squeue -u <user>
```

This will prompt a list of jobs with their corresponding `JOB_ID` and status.

Jobs can be cancelled using

```batch
scancel <JOB_ID>
```

## Connecting through VSCode for prototyping

It is possible to open a remote session on the cluster using the VSCode IDLE. This allows to develop program scripts directly on the cluster, which is extremely convenient when using GPUs (you will have to debug a lot, trust me). This process seems long at first but once it is set up it gets quick and it is really convenient to experiment with GPUs.

The first step to configure access to the cluster via VSCode is to set up SSH Public Key Athentication. This is required so that we can login securely through an SSH tunnel and everything works properly. First, we must generate a public key for our local machine using the following command

```bash
ssh-keygen -t rsa -b 4096 -C "<email>"
```

Adding `<email>` as comment allows to identify the public key. Then, we have to install this key into the server. This is basically telling the server that this key corresponds to our machine and it is done through the following command (for the `Hyperion` cluster)

```bash
ssh-copy-id hyperion.sw.ehu.es
```

You will be prompted to introduce your password. After successful completion, the terminal will prompt the following message

```bash
Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'hyperion.sw.ehu.es'"
and check to make sure that only the key(s) you wanted were added.
```

Once the SSH Public Key Authentication in set up, we must start a VSCode Server instance on the cluster to connect to the cluster from out local machine. You will have to repeat this process of launching a server each time you want to connect to the cluster via VSCode, so the most convenient thing to do is to create a script that does this automatically. Such a script is the following

```bash
#!/bin/bash
#SBATCH --qos=regular
#SBATCH --job-name=VSCodeServer
#SBATCH --output=vscodeserver.out
#SBATCH --cpus-per-task=2
#SBATCH --ntasks=2
#SBATCH --mem=16gb
#SBATCH --time=08:00:00

# The following module gives access to the ssh-server-start.sh command
module load DIPC-Utils/0.0.1

# Execute ssh server on the compute node
ssh-server-start.sh
```

You should save this script in the `/scratch/` directory of your user (Eg. with name `vscodeserver.slurm`) and launch it each time you want to connect via VSCode to the cluster using the following command

```bash
sbatch vscodeserver.slurm
```

You maybe have noticed that this process is the same as launching any other job in the cluster. It is possible that you have to wait some minutes until the server is launched in one of the nodes of the cluster. This is why it is recommended to set a long time on execution of the job (08h in the script above), so that you can connect many times over the day without waiting time.

Once the VSCode server starts running, a file will appear in your `/scratch/user/` folder with the following name

```bash
slurm-<JOB_ID>.out
```

This file contains the information required to login into the VSCode server. It will have the following form

```bash
Selecting a random available port...
Selected port 21507.
Using existing SSH configuration file.
Starting custom SSH server on port 21507...
SSHD started successfully with PID 2201883.
==============================================================
Custom SSH Server Started
==============================================================
Hostname: hyperion-022
Port: 21507
==============================================================
To securely connect to the custom SSH server, set up SSH tunneling:
1. Run the following command on your local machine:

   ssh -L <local-port>:hyperion-022:21507 <user>@hyperion.sw.ehu.es -N

2. Connect your SSH client to 'localhost:<local-port>'.

   ssh -p <local-port> cobos@localhost
==============================================================
Running under SLURM job ID: 1127566
```

We now have to choose a port in our local machine to set up the SSH tunnel into, I suggest using `2222`, because it is usually free. This is done by running the following command **in a new terminal window**.

```bash
ssh -L 2222:hyperion-022:21507 <user>@hyperion.sw.ehu.es -N
```

The new terminal window must remain open while our session is alive. We now can get into the last step and set up VSCode to connect into our local SSH tunnel that points to the VSCode server in the cluster. The set up procedure is as follows

#### 1. Download the Remote - SSH extension in VSCode

![Image](/README/DIPC%20Cluster/Summary%20figures/RemoteSSH%20installation.png)

#### 2. Save your credentials in VSCode

![Image](/README/DIPC%20Cluster/Summary%20figures/RemoteSSH%20credentials.png)

#### 3. Connect to the cluster through VSCode

![Image](/README/DIPC%20Cluster/Summary%20figures/RemoteSSH%20connect.png)

And that's finally it. We can now directly program on the cluster. The steps above should only be followed once. Once the first set up is completed the steps to connect are the following

1. Launch the VSCode server on the cluster

2. Open the SSH tunnel with the command provided by the server

3. Connect to the SSH tunnel through VSCode.