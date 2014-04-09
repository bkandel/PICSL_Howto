---
layout: page
permalink: /cluster-usage/
title: Cluster Usage
description: "Instructions on how to use the cluster."
tags: [cluster, software]
---

<section id="table-of-contents" class="toc">
  <header>
    <h3 >Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

General notes and instructions for using the PICSL cluster.  Before reading this introduction, it is worthwhile to read a few tutorials on cluster usage and architecture if youhave never used one before.  If you read nothing else, please read this sentence:  **THE HEAD NODE IS NOT TO BE USED FOR COMPUTATIONS**.  Details below. 

## Accessing the cluster
To get access to the cluster, you must contact Michael Stauffer for a logon id.  He will also provide you with IP addresses for the cluster machines.  Because the cluster is behind the UPHS firewall, you will need to VPN into the network to access the cluster from outside the lab.  Contact Michelle for information on getting VPN access.  Once you have the IP addresses and a username, you can add the IP address and your username to the `~/.ssh/config` file on your computer to make access easier.  To ease access to files on the cluster, it is worthwhile to mount the cluster from your computer using `sshfs` or similar.

## Machines 

The cluster is a parallel computing environment powered by the Sun Grid Engine. In total there are 256 available slots.  Access to the cluster is provided by the head node (`picsl-cluster`).  The head node is used to submit jobs to the compute nodes and for moving around the cluster. This machine serves as both the login node, from which cluster jobs can be submitted, and the file server for the entire cluster. If this machine is slow, everyone’s jobs are slow. If it’s slowed severely or crashes, it will affect all the submitted jobs. Therefore, the head node is *not* used for computation-heavy processes.   If you need to run a resource-heavy job interactively, use `qlogin` to create an interactive session on a compute node.  

For compiling programs, use the `picsl-build` machine and copy the binaries over to your `~/bin` directory. The `/mnt/build` directory is connected to this machine directly, so it provides the fastest access to a disk drive. 

## Storage

There are three places you can store files on the cluster: 

1.  `/home`:  This is the only storage site that is backed up.  There is a limited amount of storage here.  To see how much storage you have left, use `quota -s`.  This is the only storage location that is visible to the compute nodes, so all data used in submitted jobs must be stored here. This partition is shared by all users on all cluster nodes. Therefore it is critical that everyone read and write the minimum amount of data necessary. Moreover, it is important to avoid parallel jobs all trying to read and write at exactly the same time. Excess I/O on the home partition will slow down the cluster for everybody and can cause jobs to fail. Long term, unnecessary disk activity will accelerate wear and tear on the cluster RAID drives.
2. `/mnt/data`:  This is to be used for backing up data that is not currently being used.  Storage quotas are somewhat more liberal here. 
3. `/mnt/build`:  This is used for compiling binaries.  This drive is connected directly to `picsl-build`, so use this machine for compiling. 

## Submitting jobs 

To submit jobs to the cluster, use the `qsub` command.  The man page is very helpful and worth reading.  Some options that are particularly useful are: 

1. `-S`: Tells SGE what to use to execute your command. 
2. `-V`: Exports your environment variables to the qsub environment. 
3. `-pe serial`:  Tells qsub to assign more slots to your command.  This is useful if you need more memory for your jobs, as each slot is only allotted 2G. 
4. `-wd`: Where to put the output from the job. 
5. `-N`: Name of job.  Useful if you want to tell other jobs to wait for this job to finish. 
6. `hold_jid`: Hold this job until another job is finished.  

For example, if you wanted to run a script called `myscript.sh` with `bash`, assign 3 slots to your job, put the output in `/home/myname/data/somedir`, and wait for job `firstjob` to finish, you would run 
{% highlight bash %}
qsub -S /bin/bash -pe serial 3 -wd /home/myname/data/somedir -hold_jid firstjob myscript.sh
{% endhighlight %}

To see what jobs you have submitted, use `qstat`. `qstat -u "*"` shows everyone's jobs, which can be useful for looking at cluster load.  To delete a job, use `qdel JOBID`, or `qdel -u userid` to delete all jobs for a user (you are only allowed to delete your own jobs).  

## Cluster etiquette

Jobs are submitted to the cluster with qsub. The head node is the only node that can submit qsub
jobs. Once a job is submitted, it goes into a queue until there are available slots. Each user can
occupy up to 64 slots at one time, if available. Each compute node has 8 slots. If you crash a
compute node, you may kill 8 jobs, some of which might not be yours. Crash the head node,
and you will bring down the entire cluster.  To look at the load on the head cluster, use `top`. 

### Minimizing network disk usage

Most performance problems on the cluster have been caused by large amounts of disk usage on
`/home`. In general, disk I/O is slower than doing things in memory, so doing less of it will improve
efficiency for you as well as everyone else. Output files need to be written to /home so that they
will be accessible after the job finishes. Intermediate files should be written to the local hard drive
on the compute nodes.
When you submit a job with qsub, SGE creates for you a variable TMPDIR on the local
`/state/partition1` of the compute node. This directory is automatically removed when your
job terminates. Your code should use TMPDIR for all intermediate files. If $TMPDIR does not exist,
there are various choices for how to proceed. You can exit and tell the user to define TMPDIR, or
you can use a fallback. Be extremely careful about using /tmp as a fallback on the cluster - this
directory is on the root partition, and is not very large. If you fill the root partition of a node, it will
crash. Use the space on `/state/partition1`.

### Public data

Public data is stored in two places: `/mnt/data/PUBLIC` and `/home/local/PUBLIC`. Everyone can
write to `/mnt/data/PUBLIC`, but an admin needs to make the data you put there owned by root,
or it will still count towards your disk quota. The `/home/local/PUBLIC` directory is visible to the
compute nodes, and should be used to store data that many users need to access in qsub jobs. Only
an admin can write there.

### Memory
On the compute nodes, jobs are limited to 2 Gb RAM per slot, and they will die immediately when
they exceed this limit. If you need more RAM than this, you can reserve multiple slots with `qsub
-pe serial N`, where `N` is between 2 and 8. For example, if you ask for two slots per job, you can
run 32 jobs concurrently and each can use up to 4 Gb RAM.


### Don’t run things on the head node

The head node is a fairly capable system, but it has a lot to do. It needs to handle the terminals of
everyone logged into it, and it needs to act as the file server for all 256 jobs that can potentially run
at one time. Therefore you should avoid slowing it down, and definitely avoid crashing it.
Sometimes it is necessary to run things interactively on the cluster. You can do this with qlogin.
This command will look for an available slot, and give you a login shell on a compute node. This
is less efficient than running things via qsub, because you hold onto a slot (or two) the whole time
you’re logged in, and that stops other people using that slot even if you are out to lunch. For
this reason, you should exit from qlogin shells when you are done, and they will automatically
terminate after 24 hours.

Another way to avoid running things on the head node is to use SSHFS. This lets you use ssh to
mount the cluster file system on your local machine. Remember though that you are still using
the cluster home partition, so the same consideration of not reading and writing files unnecessarily
applies.

### Multi-threaded code
The compute nodes limit memory usage, but they can’t control CPU usage. If you run multi-thread
code, you can occupy all the cores on the machine, slowing down other jobs.
For ITK code, threads can be controlled with an environment variable. Put this in your .bashrc file
in your home directory (note - not .bash profile as this doesn’t get called by non-login shells).

{% highlight bash %}
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS=$NSLOTS
if [ [ $ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS - lt 1 ]] 
then
  ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS=1
fi
{% endhighlight %}

The `NSLOTS` variable is set automatically by qsub when you submit a job. Thus, you can use as
many threads as you have slots, ensuring that all cores on the node get used, but not overtaxed.
In Matlab, there’s less scope to modify the number of threads in his way, so you should run with
the `-singleCompThread` option to restrict the process to a single thread.



## Internet and VPN Access
Wireless access is provided by the network uphs-fast. You will need a UPHS email account to
use this (see below). The uphs-guest network is provides Internet access to anyone but is not
inside the firewall.
 
All machines within the lab, as well as the cluster, are within the UPHS firewall. This means they
are invisible to the outside world. From within the firewall, you can communicate with outside
servers via SSH. To access UPHS machines from outside the firewall, you need VPN access. This requires a UPHS
email account, which must be sponsored by a hospital faculty member. Once you have a UPHS email address and VPN authorization, download the appropriate client and
follow the instructions at [http://www.uphs.upenn.edu/network/](http://www.uphs.upenn.edu/network/).

Wired connections are the fastest. Use these for large data transfers.  About half of the ports around the lab are operational; usually they come in banks of two at a desk
and one is working.

## Computer security
The firewall protects us from a great number of external security threats, but we still need to take
responsibility for our computer security. Laptops, other mobile devices, and USB drives brought
into the network from outside can contain malware. Do not install or attempt to download illegally copied software. Use secure passwords. Most resources will allow you to use public-key
authentication for login, so you don’t have to enter your password all the time.


Keep your OS up to date. For Windows users, this means running a patched version of Vista or
later, since XP support is about to be discontinued. Apple does not formally discontinue support,
they just silently stop updating old versions. At the present time (April 2014) Snow Leopard (10.6)
has not been patched for several months.
Anti-virus software is available through the university [here](http://www.upenn.edu/computing/virus/desklaptop/download.html). For personal Mac machines, Sophos is also available for
free to the general public.


## Printers
There are three network printers: geelab1 (suite 370), geelab2 (suite 320) and geelabc (suite 370).
Print duplex on all of these to save paper, and print only what you need, especially in color. Contact
Michael Stauffer for the IP addresses.


## Common problems

### DNS
The DNS is often inconsistent, so it is helpful to know the IP addresses of the machines you use
frequently. Often the DNS works from some machines but not others, if you can’t connect by name
try the IP.

### Firewall
The UPHS blocks most traffic apart from a few standard protocols such as SSH, HTTP, HTTPS.
The main problem this causes for us is that we can’t access git or Subversion code repositories.
This is a fairly common problem so many repositories offer the option for http or https access. If
you have problems connecting to a network service outside UPHS, it may be blocked. All email
other than UPHS and mail.med is blocked, but webmail is allowed.
Certain web pages are also blocked. If a web page is blocked you will see a Websense error
message in your browser.

### core.xxx Files
If a compute node crashes, it may dump the memory into a `core.xxx` file.  These eat up a lot of memory and should be deleted.  If you don't know where it came from you can use `file core.xxx` to get more information on the file. 



