---
title: "Working on a cluster"
teaching: 15
exercises: 10
questions:
- "What is a cluster?"
- "How does a cluster work?"
- "How do I log on to a cluster?"
objectives:
- "Connect to a cluster."
- "Understand the general cluster architecture."
keypoints:
- "A cluster is a set of networked machines."
- "Clusters typically provide a login node and a set of worker nodes."
- "Files saved on one node are available everywhere."
---

By now, we are all expert Bash users.
Well, maybe not experts, but we know everything we need to in order to start
using a high-performance computing "supercomputer".
Before we start though, let's go over a few key concepts.

## What is a cluster?

The words "cloud", "cluster", and "high-performance computing" get thrown around a lot.
So what do they mean exactly?
And more importantly, how do we use them for our work?

The *cloud* is a generic term commonly used to refer to remote computing
resources of any kind -- that is, any computers that you use but are not
right in front of you.  
Cloud can refer to webservers, remote storage, API endpoints,
as well as more traditional "compute" resources.
A *cluster* on the other hand, is a term used to describe a network of computers.
The computers in a cluster typically share a common purpose,
and are used to accomplish tasks that might otherwise be too big for any one computer.

![The cloud is made of Linux](../fig/linux-cloud.jpg)

## Where are we?

Very often, many users are tempted to think of a high-performance
computing installation as one giant, magical machine.
Sometimes, people will assume that the computer they've logged onto is the entire computing cluster.
So what's really happening? What computer have we logged on to?
The name of the current computer we are logged onto can be checked with the `hostname` command.
(Clever users will notice that the current hostname is also part of our prompt!)

```
[remote]$ hostname
```
{: .bash}
```
gra-login3
```
{: .output}

Individual computers that compose a cluster are typically called *nodes* (although
  you will also hear people call them *servers*, *computers* and *machines*).  On a cluster,
  there are different types of nodes for different types of tasks.  
The node where you are right now is called the *head node*, *login node* or
*submit node*.  A login node serves as an access point to the cluster.
As a gateway, it is well suited for uploading and downloading files,
setting up software, and running quick tests.
It should never be used for doing actual work.

The real work on a cluster gets done by the *worker* (or *execute*) *nodes*
Worker nodes come in many shapes and sizes,
but generally are dedicated to doing all of the heavy lifting that needs doing.

All interaction with the worker nodes is handled by a specialized piece of software called a scheduler
(the scheduler used in this lesson is called SLURM).  We'll learn more about how to use 
the scheduler to submit jobs next, but for now, it can also tell us more information about 
the worker nodes.  

For example, we can view all of the worker nodes with the `sinfo` command.

```
[remote]$ sinfo
```
{: .bash}
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
compute*     up 7-00:00:00      1 drain* gra259
compute*     up 7-00:00:00     11  down* gra[8,99,211,268,376,635,647,803,852,966,986]
compute*     up 7-00:00:00      1   drng gra272
compute*     up 7-00:00:00     31   comp gra[988-991,994-1002,1006-1007,1015,1017,1021-1022,1028-1033,1037-1041,1043]
compute*     up 7-00:00:00     33  drain gra[225-251,253-256,677,1026]
compute*     up 7-00:00:00    323    mix gra[7,13,25,41,43-44,56,58-77,107-108,112-113,117,125-126,163,168-169,173,180-203,205-210,220,224,257-258,300-317,320,322-349,385-387,398,400-428,448,452,460,528-529,540-541,565-601,603-606,618,622-623,628,643-646,652,657,660,665,678-699,710-711,713-728,734-735,737,741-751,753-755,765,767,774,776,778,796-798,802,805-812,827,830,832,845-846,853,856,865-866,872,875,912,914,916-917,925,928,930,934,953-954,959-960,965,969-971,973,1004,1008-1009,1011,1013-1014,1023-1025]
compute*     up 7-00:00:00    464  alloc gra[1-6,9-12,14-19,21-24,26-40,42,45-55,57,100-106,109-111,114-116,118-122,127,164-167,174-179,204,212-219,221-223,252,260-267,269-271,273-284,318-319,321,350-375,377-384,388-397,399,453-459,461-501,526-527,530-539,542-564,607-608,629-634,636-642,648-651,653-656,658-659,661-664,666-676,700-703,738,756-764,766,768-773,804,813-826,828-829,831,833-844,847-851,854-855,857-864,867-871,873-874,876-911,913,915,918-924,926-927,929,931-933,935-936,938-952,955-958,961-964,967-968,972,974-985,987,992-993,1003,1005,1010,1012,1016,1018-1020,1027,1034-1036,1042]
compute*     up 7-00:00:00    176   idle gra[78-98,123-124,128-162,170-172,285-299,429-447,449-451,502-525,602,609-617,619-621,624-627,704-709,712,729-733,736,739-740,752,775,777,779-795,799-800]
compute*     up 7-00:00:00      3   down gra[20,801,937]
```
{: .output}

There are also specialized machines used for managing disk storage, user authentication,
and other infrastructure-related tasks.
Although we do not typically logon to or interact with these machines directly,
they enable a number of key features like ensuring our user account and files are available throughout the cluster.
This is an important point to remember:
files saved on one node (computer) are available everywhere on the cluster!

## What's in a node? 

All of a cluster's nodes have the same components as your own laptop or desktop:
*CPUs* (sometimes also called
  *processors* or *cores*), *memory* (or *RAM*), and *disk* space.  
  CPUs are a computer's tool for actually running programs and calculations.
  Information about a current task is stored in the computer's memory.  Disk
  is a computer's long-term storage for information it will need later.

> ## Explore Your Computer
>
> Try to find out the number of CPUs and amount of memory available on your 
> personal computer.  
{: .challenge}

> ## Explore The Head Node
>
> Now we'll compare the size of your computer with the size of the head node: 
> To see the number of processors, run: 
> ```
> nproc --all
> ```
> {: .bash}
> or 
> ```
> cat /proc/cpuinfo
> ```
> {: .bash}
> to see full details.  
> 
> How about memory? Try running: 
> ```
> free -m
> ```
> {: .bash}
> or for more details: 
> ```
> cat /proc/meminfo free -m
> ```
> {: .bash}
{: .challenge}

> ## Explore a Worker Node
> 
> Finally, let's look at the resources available on the worker nodes where your jobs 
> will actually run.  
> Try running this command to see the name, CPUs and memory available on the worker nodes: 
> ```
> sinfo -n aci-377 -o "%n %c %m"
> ```
> {: .bash}
{: .challenge}

> ## Units
> 
> A computer's memory and disk are measured in units called *bytes*.  The magnitude 
> of a file or memory use is measured using the same prefixes of the metric system: 
> kilo, mega, giga, tera.  So 1024 bytes is a kilobyte, 1024 kilobytes is a megabyte, 
> and so on.  
>
{: .callout}

With all of this in mind, we will now cover how to talk to the cluster's scheduler,
and use it to start running our scripts and programs!

The scheduler used in this lesson is SLURM.
Although SLURM is not used everywhere,
running jobs is quite similar regardless of what software is being used.
The exact syntax might change, but the concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively.
Any command (or series of commands) that you want to run on the cluster is
called a *job*, and the process of using a scheduler to run the job is called
*batch job submission*.

In this case, the job we want to run is just a shell script.
Let's create a demo shell script to run as a test.

> ## Creating our test job
>
> Using your favorite text editor, create the following script and run it.
> Does it run on the cluster or just our login node?
>
>```
>#!/bin/bash
>
> echo 'This script is running on:'
> hostname
> sleep 120
>```
{: .challenge}

If you completed the previous challenge successfully,
you probably realize that there is a distinction between
running the job through the scheduler and just "running it".
To submit this job to the scheduler, we use the `sbatch` command.

```
[remote]$ sbatch example-job.sh
```
{: .bash}
```
Submitted batch job 36855
```
{: .output}

And that's all we need to do to submit a job.  Our work is done -- now the
scheduler takes over and tries to run the job for us.  While the job is waiting
to run, it goes into a list of jobs called the *queue*.
To check on our job's status, we check the queue using the command `squeue`.

```
[remote]$ squeue -u yourUsername
```
{: .bash}
```
   JOBID     USER ACCOUNT           NAME  ST REASON    START_TIME                TIME  TIME_LEFT NODES CPU
S
   36856 yourUsername yourAccount example-job.sh   R None      2017-07-01T16:47:02       0:11      59:49     1
1
```
{: .output}

We can see all the details of our job, most importantly that it is in the "R" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue ("PENDING") or have an error.
The best way to check our job's status is with `squeue`.
Of course, running `squeue` repeatedly to check on things can be a little tiresome.
To see a real-time view of our jobs, we can use the `watch` command.
`watch` reruns a given command at 2-second intervals.
Let's try using it to monitor another job.

```
[remote]$ sbatch example-job.sh
[remote]$ watch squeue -u yourUsername
```
{: .bash}

You should see an auto-updating display of your job's status.
When it finishes, it will disappear from the queue.
Press `Ctrl-C` when you want to stop the `watch` command.

## Customizing a job

The job we just ran used all of the scheduler's default options.
In a real-world scenario, that's probably not what we want.
The default options represent a reasonable minimum.
Chances are, we will need more cores, more memory, more time,
among other special considerations.
To get access to these resources we must customize our job script.

Comments in UNIX (denoted by `#`) are typically ignored.
But there are exceptions.
For instance the special `#!` comment at the beginning of scripts
specifies what program should be used to run it (typically `/bin/bash`).
Schedulers like SLURM also have a special comment used to denote special
scheduler-specific options.
Though these comments differ from scheduler to scheduler,
SLURM's special comment is `#SBATCH`.
Anything following the `#SBATCH` comment is interpreted as an instruction to the scheduler.

Let's illustrate this by example.
By default, a job's name is the name of the script,
but the `-J` option can be used to change the name of a job.

Submit the following job (`sbatch example-job.sh`):

```
#!/bin/bash
#SBATCH -J new_name

echo 'This script is running on:'
hostname
sleep 120
```

```
[remote]$ squeue -u yourUsername
```
{: .bash}
```
   JOBID     USER ACCOUNT           NAME  ST REASON    START_TIME                TIME  TIME_LEFT NODES CPUS
   38191 yourUsername yourAccount       new_name  PD Priority  N/A                       0:00    1:00:00     1  1
```
{: .output}

Fantastic, we've successfully changed the name of our job!

> ## Setting up email notifications
>
> Jobs on an HPC system might run for days or even weeks.
> We probably have better things to do than constantly check on the status of our job
> with `squeue`.
> Looking at the [online documentation for `sbatch`](https://slurm.schedmd.com/sbatch.html)
> (you can also google "sbatch slurm"),
> can you set up our test job to send you an email when it finishes?
>
> Hint: you will need to use the `--mail-user` and `--mail-type` options.
{: .challenge}

### Resource requests

But what about more important changes, such as the number of CPUs and memory for our jobs?
One thing that is absolutely critical when working on an HPC system is specifying the
resources required to run a job.
This allows the scheduler to find the right time and place to schedule our job.
If you do not specify requirements (such as the amount of time you need),
you will likely be stuck with your site's default resources,
which is probably not what we want.

The following are several key resource requests:

* `-n <nnodes>` - how many nodes does your job need?

* `-c <ncpus>` - How many CPUs does your job need?

* `--mem=<megabytes>` - How much memory on a node does your job need in megabytes? You can also specify gigabytes using by adding a little "g" afterwards (example: `--mem=5g`)

* `--time <days-hours:minutes:seconds>` - How much real-world time (walltime) will your job take to run? The `<days>` part can be omitted.

Note that just *requesting* these resources does not make your job run faster!  We'll
talk more about how to make sure that you're using resources effectively in a later
episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 2 cpus, 4 gigabytes of memory, and 5 minutes of walltime.
{: .challenge}

> ## Job environment variables
>
> When SLURM runs a job, it sets a number of environment variables for the job.
> One of these will let us check our work from the last problem.
> The `SLURM_CPUS_PER_TASK` variable is set to the number of CPUs we requested with `-c`.
> Using the `SLURM_CPUS_PER_TASK` variable,
> modify your job so that it prints how many CPUs have been allocated.
{: .challenge}

Resource requests are typically binding.
If you exceed them, your job will be killed.
Let's use walltime as an example.
We will request 30 seconds of walltime,
and attempt to run a job for two minutes.

```
#!/bin/bash
#SBATCH -t 0:0:30

echo 'This script is running on:'
hostname
sleep 120
```

Submit the job and wait for it to finish.
Once it is has finished, check the log file.
```
[remote]$ sbatch example-job.sh
[remote]$ watch squeue -u yourUsername
[remote]$ cat slurm-38193.out
```
{: .bash}
```
This job is running on:
gra533
slurmstepd: error: *** JOB 38193 ON gra533 CANCELLED AT 2017-07-02T16:35:48 DUE TO TIME LIMIT ***
```
{: .output}

Our job was killed for exceeding the amount of resources it requested.
Although this appears harsh, this is actually a feature.
Strict adherence to resource requests allows the scheduler to find the best possible place
for your jobs.
Even more importantly,
it ensures that another user cannot use more resources than they've been given.
If another user messes up and accidentally attempts to use all of the CPUs or memory on a node,
SLURM will either restrain their job to the requested resources or kill the job outright.
Other jobs on the node will be unaffected.
This means that one user cannot mess up the experience of others,
the only jobs affected by a mistake in scheduling will be their own.

## Canceling a job

Sometimes we'll make a mistake and need to cancel a job.
This can be done with the `scancel` command.
Let's submit a job and then cancel it using its job number.

```
[remote]$ sbatch example-job.sh
[remote]$ squeue -u yourUsername
```
{: .bash}
```
Submitted batch job 38759

   JOBID     USER ACCOUNT           NAME  ST REASON    START_TIME                TIME  TIME_LEFT NODES CPUS
   38759 yourUsername yourAccount example-job.sh  PD Priority  N/A                       0:00       1:00     1    1
```
{: .output}

Now cancel the job with it's job number.
Absence of any job info indicates that the job has been successfully canceled.

```
[remote]$ scancel 38759
[remote]$ squeue -u yourUsername
```
{: .bash}
```
   JOBID     USER ACCOUNT           NAME  ST REASON    START_TIME                TIME  TIME_LEFT NODES CPUS
```
{: .output}

> ## Cancelling multiple jobs
>
> We can also all of our jobs at once using the `-u` option.
> This will delete all jobs for a specific user (in this case us).
> Note that you can only delete your own jobs.
>
> Try submitting multiple jobs and then cancelling them all with
> `scancel -u yourUsername`.
{: .challenge}

## Other types of jobs

Up to this point, we've focused on running jobs in batch mode.
SLURM also provides the ability to run tasks as a one-off or start an interactive session.

There are very frequently tasks that need to be done semi-interactively.
Creating an entire job script might be overkill,
but the amount of resources required is too much for a login node to handle.
A good example of this might be building a genome index for alignment with a tool like [HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml).
Fortunately, we can run these types of tasks as a one-off with `srun`.

`srun` runs a single command on the cluster and then exits.
Let's demonstrate this by running the `hostname` command with `srun`.
(We can cancel an `srun` job with `Ctrl-c`.)

```
[remote]$ srun hostname
```
{: .bash}
```
gra752
```
{: .output}

`srun` accepts all of the same options as `sbatch`.
However, instead of specifying these in a script,
these options are specified on the command-line when starting a job.
To submit a job that uses 2 cpus for instance,
we could use the following command
(note that SLURM's environment variables like `SLURM_CPUS_PER_TASK` are only available to batch jobs run with `sbatch`):

```
[remote]$ srun -c 2 echo "This job will use 2 cpus."
```
{: .bash}
```
This job will use 2 cpus.
```
{: .output}

### Interactive jobs

Sometimes, you will need a lot of resource for interactive use.
Perhaps it's our first time running an analysis
or we are attempting to debug something that went wrong with a previous job.
Fortunately, SLURM makes it easy to start an interactive job with `srun`:

```
[remote]$ srun --x11 --pty bash
```
{: .bash}

> ## Note for administrators
>
> The `--x11` option will not work unless the [slurm-spank-x11](https://github.com/hautreux/slurm-spank-x11) plugin is installed.
> You should also make sure `xeyes` is installed as an example X11 app
> (`xorg-x11-apps` package on CentOS).
> If you do not have these installed, just have students use `srun --pty bash` instead.
{: .callout}

You should be presented with a bash prompt.
Note that the prompt will likely change to reflect your new location,
in this case the worker node we are logged on.
You can also verify this with `hostname`.

> ## Creating remote graphics
>
> To demonstrate what happens when you create a graphics window on the remote node,
> use the `xeyes` command.
> A relatively adorable pair of eyes should pop up (press `Ctrl-c` to stop).
>
> Note that this command requires you to have connected with X-forwarding enabled
> (`ssh -X username@host.address.ca`). If you are using a Mac, you must have installed
> XQuartz (and restarted your computer) for this to work.
{: .challenge}

When you are done with the interactive job, type `exit` to quit your session.
