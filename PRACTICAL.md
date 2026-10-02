# SUfRs - HPC Workshop
## Overview of the Practical System
The system used in this practical session is Lochan. The system is a highly heterogeneous cluster, running Rocky Linux 9. It was built and is managed by Research Computing as a Service (RCaaS), and is running Slurm as its scheduler.

## Connecting
Go ahead and open your terminal, then log in to the cluster. Replace `<yourUsername>` with your GUID. You will only be able to access the system, if you requested an account before the workshop!

```
$ ssh <yourUsername>@lochan.hpc.gla.ac.uk
```

When prompted, you can trust the login-node by typing `yes` then `Enter`. The characters you type after the password prompt are not displayed on the screen. Normal output will resume once you press `Enter`.

When you are logged in, you can use the `who` command to see who else is currently logged into the system. You can see that you are sharing the system with other users.

```
[<yourUsername>@ln01 ~]$ who
999999x   pts/0        <date-time> (<IP>)
xx999x    pts/0        <date-time> (<IP>)
999999y   pts/0        <date-time> (<IP>)
…
```

This is why you should not run any computational work on this server. This server does not have a lot of resource and slowing it down, will impair the experience for other users of the platform.

## Storage
If you use `pwd`, you should see you are currently in your home directory:

```
[<yourUsername>@ln01 ~]$ pwd
/mnt/home/<yourUsername>
```

Have a look at the contents of your current directory in list form using `ls -l` and you can see that you have both your local- and sharedscratch directory linked to you in your home for easy access.

```
[<yourUsername>@ln01 ~]$ ls -l  
localscratch -> /tmp/localscratch/<yourUsername>
sharedscratch -> /mnt/scratch/<yourUsername>
```

You can move data to the system using SFTP / SSH. A very accessible SFTP client with a GUI is [WinSCP](https://winscp.net/). Our teaching materials are saved in a public GitHub repository and for ease of access, we will just download them using `wget`.

```
[<yourUsername>@ln01 ~]$ wget <insert-link-here>
```

## Software
As the system uses Environment Modules, the available software on the system can be listed using the `module available` command. You will see a list of all available software on the system. In this workshop we will only be using a few.

```
[<yourUsername>@ln01 ~]$ module available
---- /mnt/software/modulefiles ----
apptainer/1.5.3/gnu1150  dmtcp/4.2.0/gnu1150     miniforge/26.3.2-3/gnu1150  R/4.6.1/gnu1150
cuda/12.5.1/gnu1150      juliaup/1.21.0/gnu1150  openmpi/5.0.10/gnu1150

[removed some of the output here for clarity]
```

Modules can make new software available to you, which you might have not had before. Use the `module load` command, followed by the name of the software you wish to load, as for example here with R:

```
[<yourUsername>@ln01 ~]$ R --version
-bash: R: command not found
[<yourUsername>@ln01 ~]$ module load R
[<yourUsername>@ln01 ~]$ R --version
R version 4.6.1 (2026-06-24) -- "Happy Hop"
Copyright (C) 2026 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu
```

It can also “overwrite” an already existing software, that is preinstalled on the system, here an example with python:

```
[<yourUsername>@ln01 ~]$ python3 --version
Python 3.9.25
[<yourUsername>@ln01 ~]$ module load python
[<yourUsername>@ln01 ~]$ python3 --version
Python 3.13.15
```

To see all modules, that are loaded into your environment right now, use the `module list` command:

```
[<yourUsername>@ln01 ~]$ module list
Currently Loaded Modulefiles:
 1) R/4.6.1/gnu1150   2) python/3.13.15/gnu1150
```

If you wish to remove a specific software, use `module unload` followed by the software name. if you want to remove all software from your environment, use `module purge`:

```
[<yourUsername>@ln01 ~]$ module unload R
[<yourUsername>@ln01 ~]$ module list
Currently Loaded Modulefiles:
 1) python/3.13.15/gnu1150
[<yourUsername>@ln01 ~]$ module purge
[<yourUsername>@ln01 ~]$ module list
No Modulefiles Currently Loaded.
```

Keep in mind, no software is loaded by default! Loading software should be part of a routine in your job. Running a job is just like logging on to the system, you should not assume a module loaded on the login node is loaded on a compute node!

## Scheduler
All the steps from here on are specific to Slurm. There are other schedulers with different syntax, the ideas and functionalities are usually very similar though.
Look at what resources you have available to you on the cluster, using the Slurm `sinfo` command. 

```
[<yourUsername>@ln01 ~]$ sinfo -o "%n %c %m %G" | column -t
HOSTNAMES  CPUS  MEMORY   GRES
node011    16    63757    (null)
node012    28    386341   (null)
node013    64    2051471  (null)
node014    64    2051471  (null)
node015    64    1019280  (null)
node016    64    1019280  (null)
node017    64    2063333  (null)
node018    64    515318   (null)
node019    64    515318   (null)
node020    64    515318   (null)
node021    64    515318   (null)
node023    64    256919   (null)
gpu003     32    256928   gpu:l40s:1
gpu004     36    256870   gpu:v100:3
```

The nodes are grouped into partitions in Slurm, this is a way to define what type of machine you want to work with. You can list all available partitions also using the `sinfo` command:

```
[<yourUsername>@ln01 ~]$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
cpu*         up 7-00:00:00     12   idle node[011-021,023]
mpi          up 7-00:00:00      4   idle node[018-021]
gpu          up 7-00:00:00      2   idle gpu[003-004]
```

For this course, node014 and node016 were reserved, meaning we’ll only be using these two. We also only make use of the default partition “cpu”, which is marked with a * in the output.

## Interactive Job
You can start an interactive job using the `srun` command. You can tell, you are on a different server by the prompt, which should now feature the name of a compute node:

```
[<yourUsername>@ln01 ~]$ srun --reservation="sufrs-hpc-workshop_2026-10-23" --pty bash
[<yourUsername>@node014 ~]$
```

Similar from how you switched from your PC to the login node, we now switched from the login node to the compute node. 

Here we can start doing computational work. As an example, we will run some python3 code.

1. Load the python module
```
[<yourUsername>@node014 ~]$ module load python 
```
2.	Run python3
```
 [<yourUsername>@node014 ~]$ python3
```
3.	Import python libraries
```
 >>> import os
```
4.	Get Slurm node name
```
 >>> node = os.getenv("SLURMD_NODENAME")
```
5.	Get Slurm JobID
```
 >>> jobid = os.getenv("SLURM_JOBID")
```
6.	Open a new text file
```
>>> myfile = open("demofile-" + jobid + ".txt", "w") 
```
7.	Write into the text file
```
 >>> myfile.write("I am doing compute work on " + node)
```
8.	Exit python3
```
 >>> exit()
```

Look at the file you just created using cat. You can see it used the name of the node we are working on in the output.

```
[<yourUsername>@node014 ~]$ cat demofile-<JobID>.txt
I am doing compute work on <ComputeNodeName>
```

This is our interactive job done, lets close our session with the `exit` command. This should bring us back to the login node, as indicated by the prompt again:

```
[<yourUsername>@node014 ~]$ exit
exit
[<yourUsername>@ln01 ~]$
```

## Batch job submission
If you are thinking the previous example was quite tedious and not very convenient, you will like batch job submissions.

First, we have to save our python code into a file. For this create a file with the .py ending and add the contents of step 3- 7in the interactive job. You can create and edit files using either `vi` or `nano`, whatever is more comfortable to you:

```
[<yourUsername>@ln01 ~]$ nano myPythonCode.py
```
```
import os
node = os.getenv("SLURMD_NODENAME")
myfile = open("demofile-" + jobid + ".txt", "w")
myfile.write("I am doing compute work on " + node)
exit()
```

Within your workshop material you should find a job script template. Also edit this file using `vi` or `nano`:

```
[<yourUsername>@ln01 ~]$ nano pythonTestJob.sh
```

Under the “SOFTWARE SETUP” subtitle add the required software, like in the interactive job example:

```
…
############# SOFTWARE SETUP #############
module load python
…
```

In the “MY CODE” section run the script you saved earlier by providing it as a parameter to the python3 executable:

```
…
############# MY CODE #############
python3 myPythonCode.py
```

Now you can submit your job using the `sbatch` utility and the path to the script:

```
[<yourUsername>@ln01 ~]$ sbatch pythonTestJob.sh
```

Within your current working directory, you should now find an output file named after your JobID and the file your python script created. Since your home storage is shared across all servers, you can see the output of your scripts in real time from the login node, even if the job ran on a compute node:

```
[<yourUsername>@ln01 ~]$ ls -l
slurm-<JobID>.out
demofile-<JobID>.txt
```

You can run this job as many times as you want by just using the sbatch command. Like this you have access to the power of the HPC compute nodes, without ever having to log into one yourself.

### Array Job

Lets say you want to run a job 100 times, it is still quite tedious having to type this like 100 times. This is where Array Jobs are your friend. You can easily modify a job to be an Array job and run it as many times as you want!

In your materials you should fine a submission script, lets look at it:

```
[<yourUsername>@ln01 ~]$ cat arrayTestJob.sh
```

You will see the parameter `--array` at the bottom of the "Slurm Settings". This parameter defines how many times this job script should be run in an array. So if we run this job, we will get 3 output files:

```
[<yourUsername>@ln01 ~]$ sbatch arrayTestJob.sh
[<yourUsername>@ln01 ~]$ ls -l slurm-<JobID>*
slurm-<JobID>_1.out
slurm-<JobID>_2.out
slurm-<JobID>_3.out
```

These jobs are fully independant, meaning, they will all be queued individually. So if the cluster has 5 CPUs free, and you run an array with 8 jobs each requesting 1 CPU, 5 jobs will be run and 3 queued.

### Parallel Job

You can also use the HPC to run parallel jobs. For this you’ll need a program, that I capable to do this. In our example, we are running a simple MPI program, that just reports back where it ran. MPI is even possible to run parallel over multiple nodes, thi is what we are doing here.

Have a look at the script from the workshop materials and note, that we set it to run over 2 nodes, with 4 tasks each:

```
[<yourUsername>@ln01]$ cat parallelTestJob.sh
```

Run the job using `sbatch` and look at the output file. Slurm and MPI work hand in hand here, t the program ran on each detected task on each node and wrote a message to the console. Slurm collects all these outputs in the slurm-<JobID>.out file:

```
[<yourUsername>@ln01]$ sbatch parallelTestJob.sh
[<yourUsername>@ln01]$ cat slurm-<JobID>.out
```

## Job monitoring and resource efficiency
Look at the monitoring script in your workshop materials in your console. Every step of the script is described with a comment above:

```
[<yourUsername>@ln01]$ cat monitoringTestJob.sh
```

The script does not need any adjustments and can be ran as is using `sbatch` to submit it to the scheduler. Upon submission you should get a JobID printed to the console, we will use this for the coming queries, indicated by <JobID>:

```
[<yourUsername>@ln01]$ sbatch monitoringTestJob.sh
Submitted batch job <JobID>
```

You can see all running jobs on the system using the `squeue` command. If you only want to see your own jobs, you can specify your username with the `-u` parameter. If you only want to see your job queued earlier, you can use the `-j` parameter followed by your JobID.

```
[<yourUsername>@headnode01]$ squeue
[<yourUsername>@headnode01]$ squeue -u <yourUsername>
[<yourUsername>@headnode01]$ squeue -j <JobID>
```

The output is as follows:
- **JOBID**: Unique identifier of the job. Counts up from 1 being the first job ever queued.
- **PARTITION**: Scheduler partition the job is queued into.
- **NAME**: Name of the job as defined by --job-name in your submission or your script name.
- **USER**: User who submitted the job to the scheduler the job.
- **ST**: Status of the job: R=Running, PD=Pending.
- **TIME**: Time the job has been running for.
- **NODES**: Number of nodes.
- **NODELIST**: List of names of allocated nodes.

By now your job should be finished. Use `squeue` to check if your job is still running. By default, jobs that are not running or pending, will not show in the list.

Slurm keeps a database with information of all jobs run using the system. To access this data, you can use the `sacct` command. Using the JobID you saved from your job, we can show a wide list of information for your job. Use the `-o` parameter followed by a list of Job-Accounting-Fields.

A list of all available Job-Accounting-Fields can be found here: [sacct manual](https://slurm.schedmd.com/sacct.html#SECTION_Job-Accounting-Fields)

In our first example, we’ll show an overview of the requested resources for that job. You should see, these are the values we provided in the “SLURM SETTINGS” section of the script:

```
[<yourUsername>@headnode01]$ sacct -j <JobID> -o JobID,User,ReqCPUS,ReqMem,ReqNodes,TimeLimit -X
```

Now let’s get some more information on how and where our job ran. In this output we see that the job ran for 1 minute, and it completed with exit code 0, which means there were no errors:

```
[<yourUsername>@headnode01]$ sacct -j <JobID> -o JobID,NodeList,Start,End,Elapsed,State,ExitCode -X
```

Say we want to check if our job ran efficiently, we could use the `seff` command. It uses data from the Slurm accounting database, to create calculate how efficiently your job ran. Let’s use this utility and try to adjust our script, so it is more efficiently specified:

```
[<yourUsername>@headnode01]$ seff <JobID>
```

Based on the following three lines, we’ll make the adjustments on your script. Its fine to have a little bit of a buffer, we don’t want a job to get killed just because it overran a couple of seconds of what is expected. For CPU efficiency however, we want to get as close to 100% as possible:

```
CPU Efficiency: 49.92% of 00:12:00 core-walltime
Job Wall-clock time: 00:03:00
Memory Utilized: 1.02 GB
```
```
#SBATCH --cpus-per-task=2
#SBATCH --time=0-00:02:00
#SBATCH --mem=2G
```
Save the changes and run the job again. Once it is done running check the efficiency using `seff` again.

Accurate job scripts help the queuing system efficiently allocate shared resources. And therefore, your jobs should run quicker.
