# [PMLR25] Student Cluster Guide

[`AI Center Projects in Machine Learning Research Spring 2025`](https://vlg.inf.ethz.ch/teaching/Digital-Humans-FS-24.html) | `13.03.2025`

ETH Zurich's Department of Computer Science (D-INFK) offers a dedicated student cluster equipped with GPUs for teaching purposes. Detailed guidance for utilizing the cluster is available in the help section [here](https://www.isg.inf.ethz.ch/Main/HelpClusterComputingStudentCluster). Instead of individual logins, please use the ETH credentials of your team representative — the student whose ETH ID was submitted in the forms to receive access to GPU resources.


## Outline

1. **[Logging into the Cluster](#1-logging-into-the-cluster)**
2. **[Useful Configuration Files](#2-useful-configuration-files)**
3. **[Python Environment Setup](#3-python-environment-setup)**
4. **[Accessing GPUs: Running a Job Using `srun`](#4-accessing-gpus-running-a-job-using-srun)**
5. **[Accessing GPUs: Running a Batch of Jobs Using `sbatch`](#5-accessing-gpus-running-a-batch-of-jobs-using-sbatch)**
6. **[Accessing GPUs: Monitoring Jobs](#6-accessing-gpus-monitoring-jobs)**
7. **[Storage Options](#7-storage-options)**
8. **[Usual Issues](#8-usual-issues)**

## 1. Logging into the Cluster

To log in for the first time, execute the following command in your favorite terminal emulator, replacing `login_name` with the **ETH username of your team representative!!!** (the one in your `login_name@ethz.ch` email):

```bash
ssh <login_name>@student-cluster.inf.ethz.ch
# password: ... <-- enter your ETH email password
# Last login: ...
```

This command connects you to a "login" node. These nodes, lacking GPUs, are intended for job scheduling and management rather than computation. **Avoid running intensive processes on login nodes** to prevent potential access restrictions.

Access from outside the ETH network requires a VPN connection. Refer to the VPN setup instructions [here](https://www.isg.inf.ethz.ch/Main/ServicesNetworkVPN).

## 2. Useful Configuration Files

Consider updating your `~/.bashrc` file for your convenience and needs. You should export there your path to your Conda install of choice.

## 3. Python Environment Setup

You may use either conda or venv for setting up your Python environment. To install conda, follow these steps:

```bash
# 1. Download and install conda
cd ~/
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# - Accept the license agreement
# - Use your home directory as the installation location
# - Allow the installer to update your shell profile for conda initialization

# 2. Reactivate your shell for the changes to take effect
exit
ssh login_name@student-cluster.inf.ethz.ch

# 3. Create and activate a new conda environment
conda create -n <your_env_name> python=3.11 -y
conda activate <your_env_name>

# 4. Install PyTorch (adjust version numbers as needed per https://pytorch.org/get-started/previous-versions/)
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.6 -c pytorch -c conda-forge

# Note: GPUs are not available on login nodes
python -c "import torch; print('Cuda available?', torch.cuda.is_available())"
# Expected output: False
# GPU nodes will return `True` when queried

# 5. Run an interactive session before downloading packages
srun --pty -A pmlr -t 60 bash

# 6. Install additional packages
pip install --upgrade pip
pip install tqdm  # and others as needed

# 7. Sanity check to run in a test file
import torch

print("=== PyTorch Cluster Test Script ===")
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"CUDA version: {torch.version.cuda}")
    print(f"GPU count: {torch.cuda.device_count()}")

# It should output something like this:
# (<your_env_name>) tdieudonne@studgpu-node01:~$ python test.py
# === PyTorch Cluster Test Script ===
# PyTorch version: 2.5.1
# CUDA available: True
# CUDA version: 12.4
# GPU count: 1
```

## 4. Accessing GPUs: Running a Job Using `srun`

To initiate a GPU session, use the following command:

```bash
srun --pty -A pmlr -t 60 bash
```

This command requests a GPU session with a specified time limit. Once resources are allocated, you'll gain access to an interactive shell on a GPU node. Note that the hostname in your shell prompt should now have been changed, for example from `login_name@student-cluster:~` to `login_name@studgpu-node01:~`. This means that you are now in an interactive shell session on the `studgpu-node01` machine.

For more information, please look [here](https://www.isg.inf.ethz.ch/Main/HelpClusterComputingStudentCluster).

## 5. Accessing GPUs: Running a Batch of Jobs Using `sbatch`

For running a batch of jobs, we recommend using `sbatch` instead of `srun`. Basic instructions and examples are available [here](https://www.isg.inf.ethz.ch/Main/HelpClusterComputingStudentClusterRunningJobs), with further details found in the [official documentation](https://slurm.schedmd.com/sbatch.html) and [ETH's scientific computing wiki](https://scicomp.ethz.ch/wiki/Using_the_batch_system).

## 6. Accessing GPUs: Monitoring Jobs

To oversee your jobs within the Slurm scheduling queue, use `squeue`. Jobs listed are either in execution or pending execution. Waiting jobs are accompanied by a reason for their delayed start.

To cancel a job, use its JOB_ID found via `squeue` and run `scancel JOBID` (e.g., `scancel 48105793`). To cancel all your jobs, execute `scancel -u login_name`. For further job management options, explore [`scontrol`](https://slurm.schedmd.com/scontrol.html).

## 7. Storage Options

Every individual user is provided 20GB of disk space in their home folder `/home/<login_name>`. The usage of the space is printed after logging in to the cluster, e.g., as `Your home has 1124MB free space of 20000MB total`. You can use your home folder to setup your environment (e.g., via conda), store your code, logs, and data.

In case you need more space for downloading datasets, store large model checkpoints, etc., you can use your team's shared space in `/work/courses/pmlr/<group_ID>`. We have reserved 1TB of shared space for the course in total. Please be considerate with your space usage, e.g., using 40GB per team should be safe and feasible. If you need much more disk space, reach out to Tanguy.

## 8. Usual Issues

- Cannot suddenly SSH into the cluster from VSC? Make sure to be on the ETH/Eduoram network and check if your home directory isn’t full (`pip cache purge` or `conda clean --all` will help in this case).

- If you are installing packages, cloning repos… make sure to first create a conda environment and run an interactive environment (otherwise some packages won’t detect the presence of CUDA which will cause issues in the installs). 
