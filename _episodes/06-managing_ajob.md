---
title: "Managing a job"
teaching: 15
exercises: 25
questions:
- "Who is Justice Beaver??"
objectives:
- "Don't fall sleep"
keypoints:
- " What do you love to eat?"
---

What to do once jobs are submitted?

### Checking on the progress of jobs

You can check on your jobs with the condor_q command.

condor_q

~~~
-- Schedd: submit.chtc.wisc.edu : <127.0.0.1:9618?... @ 12/31/69 23:00:00
OWNER    BATCH_NAME    SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS
nemo     batch23       4/22 20:44      _      _      _      1      _ 3671850.0
nemo     batch24       4/22 20:56      _      _      _      1      _ 3673477.0
nemo     batch25       4/22 20:57      _      _      _      1      _ 3673728.0
nemo     batch26       4/23 10:44      _      _      _      1      _ 3750339.0
nemo     batch27       7/2  15:11      _      _      _      _      _ 7594591.0
nemo     batch28       7/10 03:22   4428      3      _      _   4434 7801943.0 ... 7858552.0
nemo     batch29       7/14 14:18   5074   1182     30     19  80064 7859129.0 ... 7885217.0
nemo     batch30       7/14 14:18   5172   1088     28     30  58310 7859106.0 ... 7885192.0

2388 jobs; 0 completed, 1 removed, 58 idle, 2276 running, 53 held, 0 suspended
~~~
{: .output}

Often, when you are starting out, and have few jobs, you may want to see one line of output per job. The -nobatch option to condor_q does this, and output might look something like:

condor_q -nobatch

~~~
-- Schedd submit.chtc.wisc.edu : <127.0.0.1:9618?...
ID          OWNER        SUBMITTED     RUN_TIME ST PRI SIZE CMD
1297254.0   nemo         5/31 18:05  14+17:40:01 R  0   7.3  condor_dagman
1297255.0   nemo         5/31 18:05  14+17:39:55 R  0   7.3  condor_dagman
1297256.0   nemo         5/31 18:05  14+17:39:55 R  0   7.3  condor_dagman
1297259.0   nemo         5/31 18:05  14+17:39:55 R  0   7.3  condor_dagman
1297261.0   nemo         5/31 18:05  14+17:39:55 R  0   7.3  condor_dagman
1302278.0   nemo         6/4  12:22   1+00:05:37 I  0   390.6 mdrun_1.sh
1304740.0   nemo         6/5  00:14   1+00:03:43 I  0   390.6 mdrun_1.sh
1304967.0   nemo         6/5  05:08   0+00:00:00 I  0   0.0  mdrun_1.sh

14 jobs; 4 idle, 8 running, 2 held
~~~
{: .output}

The output contains many columns of information about the queued jobs. The ST column (for status) shows the status of current jobs in the queue:

- R: The job is currently running.
- I: The job is idle. It is not running right now, because it is waiting for a machine to become available.
- H: The job is the hold state. In the hold state, the job will not be scheduled to run until it is released. See the condor_hold and the condor_release manual pages.

Another useful method of tracking the progress of jobs is through the job event log.

When a job begins to run, HTCondor starts up a condor_shadow process on the submit machine. 

You can also find all the machines that are running your job through the condor_status command. For example, to find all the machines that are running jobs submitted by breach@cs.wisc.edu, type:

~~~
condor_status -constraint 'RemoteUser == "breach@cs.wisc.edu"'

Name       Arch     OpSys        State      Activity   LoadAv Mem  ActvtyTime

alfred.cs. INTEL    LINUX        Claimed    Busy       0.980  64    0+07:10:02
biron.cs.w INTEL    LINUX        Claimed    Busy       1.000  128   0+01:10:00
cambridge. INTEL    LINUX        Claimed    Busy       0.988  64    0+00:15:00
falcons.cs INTEL    LINUX        Claimed    Busy       0.996  32    0+02:05:03
happy.cs.w INTEL    LINUX        Claimed    Busy       0.988  128   0+03:05:00
istat03.st INTEL    LINUX        Claimed    Busy       0.883  64    0+06:45:01
istat04.st INTEL    LINUX        Claimed    Busy       0.988  64    0+00:10:00
istat09.st INTEL    LINUX        Claimed    Busy       0.301  64    0+03:45:00
...
~~~
{: .output}
To find all the machines that are running any job at all, type:

~~~
condor_status -run

Name       Arch     OpSys        LoadAv RemoteUser           ClientMachine

adriana.cs INTEL    LINUX        0.980  hepcon@cs.wisc.edu   chevre.cs.wisc.
alfred.cs. INTEL    LINUX        0.980  breach@cs.wisc.edu   neufchatel.cs.w
amul.cs.wi X86_64   LINUX        1.000  nice-user.condor@cs. chevre.cs.wisc.
anfrom.cs. X86_64   LINUX        1.023  ashoks@jules.ncsa.ui jules.ncsa.uiuc
anthrax.cs INTEL    LINUX        0.285  hepcon@cs.wisc.edu   chevre.cs.wisc.
astro.cs.w INTEL    LINUX        1.000  nice-user.condor@cs. chevre.cs.wisc.
aura.cs.wi X86_64   WINDOWS      0.996  nice-user.condor@cs. chevre.cs.wisc.
balder.cs. INTEL    WINDOWS      1.000  nice-user.condor@cs. chevre.cs.wisc.
bamba.cs.w INTEL    LINUX        1.574  dmarino@cs.wisc.edu  riola.cs.wisc.e
bardolph.c INTEL    LINUX        1.000  nice-user.condor@cs. chevre.cs.wisc.
...
~~~
{: .output}

## Peeking in on a running job’s output files

`The condor_tail` command can copy output files from a running job on a remote machine back to the submit machine. condor_tail uses the same networking stack as HTCondor proper, so it will work if the execute machine is behind a firewall.

```bash
condor_tail -f xx.yy
```
To copy a different file, run

```bash
condor_tail xx.yy name_of_output_file
```

## Removing a job from a queue

`condor_rm`

```bash
condor_q -nobatch

-- Schedd: froth.cs.wisc.edu : <128.105.73.44:33847> : froth.cs.wisc.edu
 ID      OWNER            SUBMITTED    CPU_USAGE ST PRI SIZE CMD
 125.0   raman           4/11 14:37   0+00:00:00 R  0   1.4  sleepy
 132.0   raman           4/11 16:57   0+00:00:00 R  0   1.4  hello

2 jobs; 1 idle, 1 running, 0 held

condor_rm 132.0
Job 132.0 removed.

condor_q -nobatch

-- Schedd: froth.cs.wisc.edu : <128.105.73.44:33847> : froth.cs.wisc.edu
 ID      OWNER            SUBMITTED    CPU_USAGE ST PRI SIZE CMD
 125.0   raman           4/11 14:37   0+00:00:00 R  0   1.4  sleepy

1 jobs; 1 idle, 0 running, 0 held
```
{: .output}

## Placing a job on hold

`condor_hold`
`condor_release`

Jobs that are running when placed on hold will start over from the beginning when released.

## Changing the priority of jobs

HTCondor provides each user with the capability of assigning priorities to each submitted job.

```bash
condor_q -nobatch raman

-- Submitter: froth.cs.wisc.edu : <128.105.73.44:33847> : froth.cs.wisc.edu
 ID      OWNER            SUBMITTED    CPU_USAGE ST PRI SIZE CMD
 126.0   raman           4/11 15:06   0+00:00:00 I  0   0.3  hello

1 jobs; 1 idle, 0 running, 0 held

condor_prio -p -15 126.0

condor_q -nobatch raman

-- Submitter: froth.cs.wisc.edu : <128.105.73.44:33847> : froth.cs.wisc.edu
 ID      OWNER            SUBMITTED    CPU_USAGE ST PRI SIZE CMD
 126.0   raman           4/11 15:06   0+00:00:00 I  -15 0.3  hello

1 jobs; 1 idle, 0 running, 0 held
```

It is important to note that these job priorities are completely different from the user priorities assigned by HTCondor. 

## Why is the job not running?

The most common reason why the job is not running is that HTCondor has not yet been through its periodic negotiation cycle, in which queued jobs are assigned to machines within the pool and begin their execution.

'-analyze` option of the condor_q command. Here is an example;

```bash
condor_q -analyze 27497829

-- Submitter: s1.chtc.wisc.edu : <128.104.100.43:9618?sock=5557_e660_3> : s1.chtc.wisc.edu
User priority for ei@chtc.wisc.edu is not available, attempting to analyze without it.
---
27497829.000:  Run analysis summary.  Of 5257 machines,
   5257 are rejected by your job's requirements
      0 reject your job because of their own requirements
      0 match and are already running your jobs
      0 match but are serving other users
      0 are available to run your job
        No successful match recorded.
        Last failed match: Tue Jun 18 14:36:25 2013

        Reason for last match failure: no match found

WARNING:  Be advised:
   No resources matched request's constraints

The Requirements expression for your job is:

    ( OpSys == "OSX" ) && ( TARGET.Arch == "X86_64" ) &&
    ( TARGET.Disk >= RequestDisk ) && ( TARGET.Memory >= RequestMemory ) &&
    ( ( TARGET.HasFileTransfer ) || ( TARGET.FileSystemDomain == MY.FileSystemDomain ) )


Suggestions:
    Condition                         Machines Matched Suggestion
    ---------                         ---------------- ----------
1   ( target.OpSys == "OSX" )         0                MODIFY TO "LINUX"
2   ( TARGET.Arch == "X86_64" )       5190
3   ( TARGET.Disk >= 1 )              5257
4   ( TARGET.Memory >= ifthenelse(MemoryUsage isnt undefined,MemoryUsage,1) )
                                      5257
5   ( ( TARGET.HasFileTransfer ) || ( TARGET.FileSystemDomain == "submit-1.chtc.wisc.edu" ) )
                                      5257
```
{: .output}


## Job in the Hold State

For the example job ID 16.0, use:

condor_q  -hold  16.0


## Job Termination


A ticket of execution is usually issued by the condor_startd, and includes:

- when the condor_startd was told, or otherwise decided, to terminate the job (the when attribute);

- who made the decision to terminate, usually a Sinful string (the who attribute);

- and what method was employed to command the termination, as both as string and an integer (the How and HowCode attributes).

The relevant log events include a human-readable rendition of the ToE, and the job ad is updated with the ToE after the usual delay.

As of version 8.9.4, HTCondor only issues ToE in three cases:

- when the job terminates of its own accord (issued by the starter, HowCode 0);

- and when the startd terminates the job because it received a DEACTIVATE_CLAIM commmand (HowCode 1)

- or a DEACTIVATE_CLAIM_FORCIBLY command (HowCode 2).

## Job Completion

When an HTCondor job completes, either through normal means or by abnormal termination by signal, HTCondor will remove it from the job queue.

By default, HTCondor does not send an email message when the job completes. Modify this behavior with the notification command in the submit description file. The message will include the exit status of the job, which is the argument that the job passed to the exit system call when it completed, or it will be notification that the job was killed by a signal. Notification will also include the following statistics (as appropriate) about the job:

- **Submitted** at:

    when the job was submitted with condor_submit
- **Completed** at:

    when the job completed
- **Real Time:**

    the elapsed time between when the job was submitted and when it completed, given in a form of <days> <hours>:<minutes>:<seconds>
- **Virtual Image Size:**

    memory size of the job, computed when the job checkpoints


The job terminated event includes the following:

- the type of termination (normal or by signal)

- the return value (or signal number)

- local and remote usage for the last (most recent) run (in CPU-seconds)

- local and remote usage summed over all runs (in CPU-seconds)

- bytes sent and received by the job’s last (most recent) run,

- bytes sent and received summed over all runs,

- a report on which partitionable resources were used, if any. Resources include CPUs, disk, and memory; all are lifetime peak values.


## Summary of all HTCondor users and their jobs

When jobs are submitted, HTCondor will attempt to find resources to run the jobs. 

~~~
condor_status -submitters


Name                 Machine      Running IdleJobs HeldJobs

ballard@cs.wisc.edu  bluebird.c         0       11        0
nice-user.condor@cs. cardinal.c         6      504        0
wright@cs.wisc.edu   finch.cs.w         1        1        0
jbasney@cs.wisc.edu  perdita.cs         0        0        5

                           RunningJobs           IdleJobs           HeldJobs

 ballard@cs.wisc.edu                 0                 11                  0
 jbasney@cs.wisc.edu                 0                  0                  5
nice-user.condor@cs.                 6                504                  0
  wright@cs.wisc.edu                 1                  1                  0

               Total                 7                516                  5
~~~
{: .output }


## Automatically managing a job

### Automatically rerunning a failed job

If a job exits with a non-zero exit code, this usually means that some error has happened. 

```bash
# Example submit description with max_retries

executable   = myexe
arguments    = SomeArgument

# Retry this job 5 times if non-zero exit code
max_retries = 5

output       = outputfile
error        = errorfile
log          = myexe.log

request_cpus   = 1
request_memory = 1024
request_disk   = 10240

should_transfer_files = yes

queue
```

## Automatically removing a job in the queue

In the submit description file, set periodic_remove to a classad expression. For example, to automatically remove a job which has been in the queue for more than 100 hours, the submit file could have

```bash
periodic_remove = (time() - QDate) > (100 * 3600)
```

or, to remove jobs that have been running for more than two hours:
```bash
periodic_remove = (JobStatus == 2) && (time() - EnteredCurrentStatus) > (2 * 3600)
```

## Automatically releasing a held job

In the same way that a job can be automatically held, jobs in the held state can be released with the periodic_release command. Often, using a periodic_hold with a paired periodic_release is a good way to restart a stuck job.

```bash
periodic_hold = (JobStatus == 2) && (time() - EnteredCurrentStatus) > (2 * 3600)
periodic_hold_reason = "Job ran for more than two hours"
periodic_hold_subcode = 42
periodic_release = (HoldReasonSubCode == 42)
```

## Holding a completed job

A job may exit, and HTCondor consider it completed, even though something has gone wrong with the job. A held job informs users that there may have been a problem with the job that should be investigated. For example, if a job should never exit by a signal, the job can be put on hold if it does with

```bash
on_exit_hold = ExitBySignal == true
```


{: .challenge}


{% include links.md %}

