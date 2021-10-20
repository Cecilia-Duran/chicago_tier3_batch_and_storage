---
title: "HTCondor Quick Start"
teaching: 5
exercises: 15
questions:
- "Brief Description"
objectives:
- "Review concepts"
keypoints:
- "what did we learn"
---
HTCondor is a job scheduler, you give HTCondor a file containing commands that tell it how to run jobs. HTCondor locates a machine that can run each job within the pool of machines, packages up the job and ships it off to this execute machine, the jobs run and output is returned to the machine that submitted the jobs

This guides provides enough guidance to submit and observer the successful completion of a first job, it then suggests extensions that you can apply to your particular jobs
	
> ## This guide presumes that
>
> - HTcondor is running.
>
> - You have access to a machine withn the pool that may submit jobs.
>
> - You are logged into a working on the submit machine.
>
> - Your program executable, your submit description file and any needed input files are all on the file system of the submit machine.
>
> - Your job (the program executable) is able to run without any interactive input.
>
{: .callout}	
	
##  A first HTCondor job 

For HTCondor to run a job it must be given details such as the names and location of the exe and all needed input files in a submit description file.

This first executable program is a shell script. Try this example, log in to the submit machine, and use an editor to type in or copy and paste the file contents. Name the resulting file:


`sleep.sh`
```bash
#!/bin/bash
# file name: sleep.sh

TIMETOWAIT="6"
echo "sleeping for $TIMETOWAIT seconds"
/bin/sleep $TIMETOWAIT

```
Change the sleep.sh file to be executable by running the following command:

```bash
chmod u+x sleep.sh
```

The submit description file describes the job. To submit this sample job, again use an editor to create the file sleep.sub.

```bash
# sleep.sub -- simple sleep job

executable              = sleep.sh
log                     = sleep.log
output                  = outfile.txt
error                   = errors.txt
should_transfer_files   = Yes
when_to_transfer_output = ON_EXIT
queue
```

The first line of this submit description file is a comment. Comments begin with the # character. Comments do not span lines.

Each line of the submit description file has the form

```bash
command_name = value
```

The command name is case insensitive and precedes an equals sign. Values to right of the equals sign are likely to be case sensitive. 

Next in this file is a specification of the executable to run. It specifies the program that becomes the HTCondor job. For this example, it is the file name of the script. A full path and executable name, or a path and executable relative to the current working directory may be specified.

The log command causes a job event log file named sleep.log to be created on the submit machine once the job is submitted.

If this script/batch file were to be invoked from the command line, and outside of HTCondor, its single line of output

```
sleeping for 6 seconds
```
would be sent to standard output. When submitted as an HTCondor job, standard output of the execute machine is on that execute machine. HTCondor captures standard output in a file due to the output command in the submit description file. This example names the redirected standard output file outfile.txt, and this file is returned to the submit machine when the job completes. The same structure is specified for standard error, as specified with the error command.

The commands

```bash
should_transfer_files   = Yes
when_to_transfer_output = ON_EXIT
```
direct HTCondor to explicitly send the needed files, including the executable, to the machine where the job executes. These commands will likely not be necessary for jobs in which the submit machine and the execute machine access a shared file system. However, including these commands will allow this first sample job to work under a large variety of pool configurations.

The queue command tells HTCondor to run one instance of this job.

### Submitting the job

With this submit description file, all that remains is to hand off the job to HTCondor. With the current working directory being the one that contains the sleep.sub submit description file and the executable (sleep.sh or sleep.bat), this job submission is accomplished with the command line

```bash
condor_submit sleep.sub
```
{: .output}

If the submission is successful, the terminal will display a response that identifies the job, of the form

~~~
Submitting job(s).
1 job(s) submitted to cluster 6.
~~~
{: .output}

### Monitoring the job

Once the job has been submitted, command line tools may help you follow along with the progress of the job. `The condor_q` command prints a listing of all the jobs currently in the queue. For example, a short time after the user Kris submits the sleep job the submit machine on a pool that has no other queued jobs, the output may appear as

~~~
condor_q
-- Submitter: example.wisc.edu : <128.105.14.44:56550> : example.wisc.edu
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
    6.0   kris            2/13 10:49   0+00:00:03 R  0   97.7 sleep.sh

1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
~~~~
{: .output}

The queue might contain many jobs. To see only Kris’ jobs, add an option to the condor_q command that specifies to only print Kris’ jobs:

```bash
condor_q -submitter kris
```
{: .output}

The first column of output from condor_q identifies the job; the identifier is composed of two integers separated by a period. The first integer is known as a cluster number, and it will be the same for each of the potentially many jobs submitted by a single invocation of condor_submit. The second integer in the identifier is known as a process ID, and it distinguishes between distinct job instances that have the same cluster number. These values start at 0.

Of interest in this output, the job is running, and it has used 3 seconds of time so far.

At job completion, the log file contains

~~~ Output
000 (006.000.000) 02/13 10:49:04 Job submitted from host: <128.105.14.44:46062>
...
001 (006.000.000) 02/13 10:49:24 Job executing on host: <128.105.15.5:43051?PrivNet=cs.wisc.edu>
...
006 (006.000.000) 02/13 10:49:30 Image size of job updated: 100000
        0  -  MemoryUsage of job (MB)
        0  -  ResidentSetSize of job (KB)
...
005 (006.000.000) 02/13 10:49:31 Job terminated.
        (1) Normal termination (return value 0)
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
        23  -  Run Bytes Sent By Job
        113  -  Run Bytes Received By Job
        23  -  Total Bytes Sent By Job
        113  -  Total Bytes Received By Job
        Partitionable Resources :    Usage  Request Allocated
           Cpus                 :                 1         1
           Disk (KB)            :   100000   100000   2033496
           Memory (MB)          :        0       98      2001
...
~~~
{: .output}

Each event in the job event log file is separated by a line containing three periods. For each event, the first 3-digit value is an event number.

### Removing a job

Successfully submitted jobs will occasionally need to be removed from the queue. Invoke the condor_rm command specifying the job identifier as a command line argument. Kris’ job may be removed from the queue with
```
condor_rm 6.0
```
{: .output}

Specification of the cluster number only as with the command

```
condor_rm 6
```
{: .output}

will cause all jobs within that cluster to be removed.

## The science Job Example

A second example job illustrates aspects of file specification for the job. Assume that the program executable is called science.exe. This program does not use standard input or output; instead, the command line to invoke this program specifies two input files and one output file. For this example, the command line to invoke science.exe (not as an HTCondor job) will be

```
science.exe infile-A.txt infile-B.txt outfile.txt
```
{: .output}

While the name of the executable is specified in the submit description file with the executable command, the remainder of the command line will be specified with the arguments command.

Here is the submit description file for this job:

``` bash
# science1.sub -- run one instance of science.exe
executable              = science.exe
arguments               = "infile-A.txt infile-B.txt outfile.txt"
transfer_input_files    = infile-A.txt,infile-B.txt
should_transfer_files   = IF_NEEDED
when_to_transfer_output = ON_EXIT
log                     = science1.log
queue
```
The input files infile-A.txt and infile-B.txt will need to be available on the execute machine within the pool where the job runs. HTCondor cannot interpret command line arguments, so it cannot know that these command line arguments for this job specify input and output files. The submit command transfer_input_files instructs HTCondor to transfer these input files from the machine where the job is submitted to the machine chosen to execute the job. The default operation of HTCondor is to transfer all files created by the job on the execute machine back to the submit machine. Therefore, there is no specification of the outfile.txt output file.

This example submit description file modifies the commands that direct the transfer of files from submit machine to execute machine and back again.

```bash
should_transfer_files   = IF_NEEDED
when_to_transfer_output = ON_EXIT
```
{: .output}


These values are the HTCondor defaults, so are not needed in this example. They are included to direct attention to the capabilities of HTCondor. The should_transfer_files command specifies whether HTCondor should assume the existence of a file system shared by the submit machine and the execute machine. Where there is a shared file system, a correctly configured pool of machines will not need to transfer the files from one machine to the other, as both can access the shared file system. Where there is not a shared file system, HTCondor must transfer the files from one machine to the other. The specification IF_NEEDED asks HTCondor to use a shared file system when one is detected, but to transfer the files when no shared file system is detected. When files are to be transferred, HTCondor automatically sends the executable as well as a file representing standard input; this file would be specified by the input submit command, and it is not relevant to this example. Other files are specified in a comma separated list with transfer_input_files, as they are in this example.

When the job completes, all files created by the executable as it ran are transferred back to the submit machine.

## Expanding the science Job and the Organization of Files

A further example promotes understanding of how HTCondor makes the submission of lots of jobs easy. Assume that the science.exe job is to be run 40 times. If the input and output files were exactly the same for each run, then only the last line of the given submit description file changes: from

```bash
queue
```
to

```bash
queue 40
```
{: .output}

It is likely that this does not produce the desired outcome, as the output file created, outfile.txt, has the same name for each queued instance of the job, and thus this file of results for each run conflicts. Chances are that the input files also must be distinct for each of the 40 separate instances of the job. HTCondor offers the use of a macro that can uniquely name each run’s input and output file names. The $(Process) macro causes substitution by the process ID from the job identifier. The submit description file for this proposed solution uniquely names the files:

```bash
# science2.sub -- run 40 instances of science.exe
executable              = science.exe
arguments               = "infile-$(Process)A.txt infile-$(Process)B.txt outfile$(Process).txt"
transfer_input_files    = infile-$(Process)A.txt,infile-$(Process)B.txt
should_transfer_files   = IF_NEEDED
when_to_transfer_output = ON_EXIT
log                     = science2.log
queue 40
```
{: .output}

The 40 instances of this job will have process ID values that run from 0 to 39. The two input files for process ID 0 are infile-0A.txt and infile-0B.txt, the ones for process ID 1 will be infile-1A.txt and infile-1B.txt, and so on, all the way to process ID 39, which will be files infile-39A.txt and infile-39B.txt. Using this macro for the output file naming of each of the 40 jobs creates outfile0.txt for process ID 0; outfile1.txt for process ID 1; and so on, to outfile39.txt for process ID 39.

This example does not scale well as the number of jobs increases, because the number of files in the same directory becomes unwieldy. Assume now that there will be 100 instances of the science.exe job, and each instance has distinct input files, and produces a distinct output file. A recommended organization introduces a unique directory for each job instance. The following submit description file facilitates this organization by specifying the directory with the initialdir command. The directories for this example are named run0, run1, etc. all the way to run99 for the 100 instances of the following example submit file:

```bash
# science3.sub -- run 100 instances of science.exe, with
#  unique directories named by the $(Process) macro
executable              = science.exe
arguments               = "infile-A.txt infile-B.txt outfile.txt"
should_transfer_files   = IF_NEEDED
when_to_transfer_output = ON_EXIT
initialdir              = run$(Process)
transfer_input_files    = infile-A.txt,infile-B.txt
log                     = science3.log
queue 100
```

The input and output files for each job instance can again be the initial simple names that do not incorporate the $(Process) macro. These files are distinct for each run due to their placement within a uniquely named directory. This organization also works well for executables that do not facilitate command line naming of input or output files.

Here is a listing of the files and directories on the submit machine within this suggested directory structure. The files created due to submitting and running the jobs are shown preceded by an asterisk (*). Only a subset of the 100 directories are shown. Directories are identified using the Linux (and Mac) convention of appending the directory name with a slash character (/).

```output
science.exe
science3.sub
run0/
    infile-A.txt
    infile-B.txt
    * outfile.txt
    * science3.log
run1/
    infile-A.txt
    infile-B.txt
    * outfile.txt
    * science3.log
run2/
    infile-A.txt
    infile-B.txt
    * outfile.txt
    * science3.log
```
{: .output}


{% include links.md %}

