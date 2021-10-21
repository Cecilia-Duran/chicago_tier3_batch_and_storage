---
title: "Submitting a Job"
teaching: 10
exercises: 30
questions:
- "The worst thing about prison was the dementors"
- "Improversation"
objectives:
- "Fly! you fools!!"
- ""
keypoints:
- "In this lesson, mention some other useful options to run our jobs??  ˒˒˒˒(>ړ”)> "
---
## Submitting a Job

The `condor_submit` command takes a job description file as input and submits the job to HTCondor.  Items such as the name of the executable to run, the initial working directory, and command-line arguments to the program all go into the submit description file. `condor_submit` creates a job ClassAd based upon the information, and HTCondor works toward running the job.

## Sample submit description files


#### Example 1

This example is one of the simplest submit description files possible. It queues the program myexe for execution somewhere in the pool. As this submit description file does not request a specific operating system to run on, HTCondor will use the default, which is to run the job on a machine which has the same architecture and operating system it was submitted from.

Before submitting a job to HTCondor, it is a good idea to test it first locally, by running it from a command shell. This example job might look like this when run from the shell prompt.

```bash
./myexe SomeArgument
```
The corresponding submit description file might look like the following

`example1.sub`
```bash
# Example 1
# Simple HTCondor submit description file
# Everything with a leading # is a comment

executable   = myexe
arguments    = SomeArgument

output       = outputfile   
error        = errorfile
log          = myexe.log

request_cpus   = 1
request_memory = 1024
request_disk   = 10240

should_transfer_files = yes

queue
```

> ## Explanation of variables in example 1
>
> output        :   The standard output for this job will go to the file `outputfile`.
>
> error :  The standard error output will go to `errorfile`.
>
> log :    Appended evenst about the job to the log file named `myexe.log`.
>
> request_cpus  :   The job should be allocated 1 cpu core.
>
> request_memory:   1024 megabytes of memory.
>
> request_disk  :   10240 kilobytes of scratch disk space.
>
> queue         :   Hey HTCondor! I've finished the description of the job, send it to the queue.
>
{: .callout}

#### Example 2

A simple example

`example2.sub`
```bash
# Example 2: Show off some fancy features,
# including the use of pre-defined macros.

executable     = foo
arguments      = input_file.$(Process)

request_memory = 4096
request_cpus   = 1
request_disk   = 16383

error   = err.$(Process)
output  = out.$(Process)
log     = foo.log

should_transfer_files = yes
transfer_input_files = input_file.$(Process)

# submit 150 instances of this job
queue 150
```

Each instance of this program works on one input file. 
We prepare 150 copies of this input file in the current directoy, and name them input_file.0, ... up to input_file.149. 
Whit transfer_input_files, we tell HTCondor which input file to send to each instance of the program.

#### Example 3

A simple example with a python script

We write our executable in the file:

`hello.py`
```bash
#!/usr/bin/env python
import sys
import time
i=1
while i<=6:
        print i
        i+=1
        time.sleep(1)
print 2**8
print "hello world received argument = " +sys.argv[1]
```

```bash
python hello.py
```
Create the directory:

```bash
mkdir output
```

Now we need or submit file:

hello.sub
```bash
Universe        = vanilla
Executable      = hello.py
Output          = output/hello.out.$(Cluster).$(Process).txt
Error           = output/hello.error.$(Cluster).$(Process).txt
Log             = output/hello.log.$(Cluster).$(Process).txt
notification = Never
Arguments = $(Process)
PeriodicRelease = ((JobStatus==5) && (CurentTime - EnteredCurrentStatus) > 30)
OnExitRemove = (ExitStatus == 0)
Queue 4
```

submit out job

```bash
condor_submit hello.sub
```

check status:

```bash
condor_q
```

### Example 4

A simple example with a small program in C.

Let's start creating a directory for this example 

```bash
mkdir simple_c
cd simple_c
```

Write your executable in the file:

`simple.c`
```bash
#include <stdio.h>

main(int argc, char **argv)
{
    int sleep_time;
    int input;
    int failure;

    if (argc != 3) {
        printf("Usage: simple <sleep-time> <integer>\n");
        failure = 1;
    } else {
        sleep_time = atoi(argv[1]);
        input      = atoi(argv[2]);

        printf("Thinking really hard for %d seconds...\n", sleep_time);
        sleep(sleep_time);
        printf("We calculated: %d\n", input * 2);
        failure = 0;
    }
    return failure;
}
```

compile the program

```bash
gcc -o simple simple.c
```

Now run the program and tell it to sleep for four seconds and calculate 10 * 2: 
```bash
./simple 4 10
```
output
```
Thinking really hard for 4 seconds...
We calculated: 20
```
{: .output}

Submitting the job

`simple.sub`

```bash
Universe   = vanilla
Executable = simple
Arguments  = 4 10
Log        = simple.log
Output     = simple.out
Error      = simple.error
Queue
```
Ask HTCondor to run the job

```bash
condor_submit simple.sub
```
Output
```
Submitting job(s)con.
Logging submit event(s).
1 job(s) submitted to cluster 6075
```
{: .output}

watch the job run

```
condor_q

-- Submitter: ws-03.gs.unina.it : <192.167.2.23:34353> : ws-03.gs.unina.it
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
   2.0   temp-01         3/15 16:27   0+00:00:00 I  0   0.0  simple 4 10       

1 jobs; 1 idle, 0 running, 0 held

% condor_q

-- Submitter: ws-03.gs.unina.it : <192.167.2.23:34353> : ws-03.gs.unina.it
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
   2.0   temp-01         3/15 16:27   0+00:00:01 R  0   0.0  simple 4 10       

1 jobs; 0 idle, 1 running, 0 held

% condor_q


-- Submitter: ws-03.gs.unina.it : <192.167.2.23:34353> : ws-03.gs.unina.it
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               

0 jobs; 0 idle, 0 running, 0 held
```
{: .output}

#### Using parameters in the simple job

If we would like to have our program calculate a whole set of values for different inputs. How can we do that?

`simple_set.sub`
```bash
Universe   = vanilla
Executable = simple
Arguments  = 4 10
Log        = simple_set.log
Output     = simple_set.$(Process).out
Error      = simple_set.$(Process).error
Queue

Arguments = 4 11
Queue

Arguments = 4 12
Queue
```

Now see what happens when we ask HTCondor to run the job and check the status

```
%  condor_submit submit
Submitting job(s)...
Logging submit event(s)...
3 job(s) submitted to cluster 2.

% condor_q 

-- Submitter: roy@ws-03.gs.unina.it : <192.167.2.23:32787> : ws-03.gs.unina.it
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
   2.0   roy             1/25 12:28   0+00:00:00 R  0   0.0  simple 4 10       
   2.1   roy             1/25 12:28   0+00:00:00 R  0   0.0  simple 4 11       
   2.2   roy             1/25 12:28   0+00:00:00 R  0   0.0  simple 4 12       

3 jobs; 0 idle, 3 running, 0 held

% condor_q 


-- Submitter: roy@ws-03.gs.unina.it : <128.105.48.160:32787> : ws-03.gs.unina.it
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               

0 jobs; 0 idle, 0 running, 0 held

% ls simple*out
simple.0.out  simple.1.out  simple.2.out  simple.out

% cat simple.0.out
Thinking really hard for 4 seconds...
We calculated: 20

% cat simple.1.out
Thinking really hard for 4 seconds...
We calculated: 22

% cat simple.2.out
Thinking really hard for 4 seconds...
We calculated: 24
```
{: .output}


## Submitting many similar jobs with one queue command

A wide variety of job submissions can be specified with extra information to the queue submit command. This flexibility eliminates the need for a job wrapper or Perl script for many submissions.

The form of the queue command defines variables and expands values, identifying a set of jobs. Square brackets identify an optional item.

queue [ < int expr > ]

queue [ < int expr > ] [ < varname > ] in [ slice ] < list of items > 

queue [ < int expr > ] [ < varname > ] matching [ files  l  dirs ] [ slice ] < list of items with file globbing > 

queue [ < int expr > ] [ < list of varnames > ] from [ slice ] < file name >  l  < list of items > 

All optional items have defaults:


- If < int expr > is not specified, it defaults to the value 1.

- If < varname > or < list of varnames > is not specified, it defaults to the single variable called ITEM.

- If slice is not specified, it defaults to all elements within the list. This is the Python slice [::], with a step value of 1.

- If neither files nor dirs is specified in a specification using the from key word, then both files and directories are considered when globbing.


The optional slice specifies a subset of the list of items using the Python syntax for a slice. Negative step values are not permitted.

Here are a set of examples.

### Example 1

```bash
transfer_input_files = $(filename)
arguments            = -infile $(filename)
queue filename matching files *.dat
```

```bash
transfer_input_files = initial.dat
arguments            = -infile initial.dat
queue
transfer_input_files = middle.dat
arguments            = -infile middle.dat
queue
transfer_input_files = ending.dat
arguments            = -infile ending.dat
queue
```

### Example 2

```bash 
queue 1 input in A, B, C
```

```bash
input = A
queue
input = B
queue
input = C
queue
````

### Example 3

```bash
queue input, arguments from (
  file1, -a -b 26
  file2, -c -d 92
)
```
Each of the two variables specified is given a value from the list of items. For this example the queue command expands to

```bash
input = file1
arguments = -a -b 26
queue
input = file2
arguments = -c -d 92
queue
```

### Example 4

```bash
queue from seq 7 9 |
```

feeds the list of items to queue with the output of seq 7 9:

```bash
item = 7
queue
item = 8
queue
item = 9
queue
```
## Variables in the Submit Description File


`$(Cluster) or $(ClusterId)`

`$(Process) or $(ProcId)`

`$$(a_machine_classad_attribute)`

`$$([ an_evaluated_classad_expression ])`
   
`$(ARCH)`

`$(OPSYS) $(OPSYSVER) $(OPSYSANDVER) $(OPSYSMAJORVER)`

`$(SUBMIT_FILE)`

`$(SUBMIT_TIME)`

`$(Year) $(Month) $(Day)`

`$(Item)`

`$(ItemIndex)`

`$(Step)`

`$(Row)`

## Including Submit Commands Defined Elsewhere

Externally defined submit commands can be incorporated into the submit description file using the syntax

```bash
include : <what-to-include>
```

The <what-to-include> specification may specify a single file, where the contents of the file will be incorporated into the submit description file at the point within the file where the include is. Or, <what-to-include> may cause a program to be executed, where the output of the program is incorporated into the submit description file. The specification of <what-to-include> has the bar character (|) following the name of the program to be executed.

Consider the example

```bash
include : ./list-infiles.sh |
```

```bash
#!/bin/sh

echo "transfer_input_files = `ls -m infiles/*.dat`"
exit 0
```


```bash
transfer_input_files = infiles/A.dat, infiles/B.dat, infiles/C.dat
```

is incorporated into the submit description file.

## Using Conditionals in the Submit Description File

Conditional if/else semantics are available in a limited form. The syntax:

```bash
if <simple condition>
   <statement>
   . . .
   <statement>
else
   <statement>
   . . .
   <statement>
endif
```

An else key word and statements are not required, such that simple if semantics are implemented. The <simple condition> does not permit compound conditions. It optionally contains the exclamation point character (!) to represent the not operation, followed by

```bash
    if defined MY_UNDEFINED_VARIABLE
       X = 12
    else
       X = -1
    endif
```

results in X = -1, when MY_UNDEFINED_VARIABLE is not yet defined.

- the version keyword, representing the version number of of the daemon or tool currently reading this conditional. This keyword is followed by an HTCondor version number. That version number can be of the form x.y.z or x.y. The version of the daemon or tool is compared to the specified version number. The comparison operators are

    - `==` for equality. Current version 8.2.3 is equal to 8.2.

    - `>=` to see if the current version number is greater than or equal to. Current version 8.2.3 is greater than 8.2.2, and current version 8.2.3 is greater than or equal to 8.2.

    - `<=  to see if the current version number is less than or equal to. Current version 8.2.0 is less than 8.2.2, and current version 8.2.3 is less than or equal to 8.2.

    As an example,

    ```bash
    if version >= 8.1.6
       DO_X = True
    else
       DO_Y = True
    endif
    ```

    - True or yes or the value 1. The statement(s) are incorporated.

    - False or no or the value 0 The statement(s) are not incorporated.

This sintax

```bash
if <simple condition>
   <statement>
   . . .
   <statement>
elif <simple condition>
   <statement>
   . . .
   <statement>
endif
```

is the same as syntax

```bash
if <simple condition>
   <statement>
   . . .
   <statement>
else
   if <simple condition>
      <statement>
      . . .
      <statement>
   endif
endif
```

Example

```bash
if defined X
  arguments = -n $(X)
else
  arguments = -n 1 -debug
endif
```

Submit variable X is defined on the condor_submit command line with
```bash
condor_submit  X=3  sample.sub
```

This command line incorporates the submit command X = 3 into the submission before parsing the submit description file. For this submission, the command line arguments of the submitted job become

```bash
arguments = -n 3
```

If the job were instead submitted with the command line

```bash
condor_submit  sample.sub
```

then the command line arguments of the submitted job become

```bash
arguments = -n 1 -debug
```
        
## Interactive Jobs

An interactive job is a Condor job that is provisioned and scheduled like any other vanilla universe Condor job onto an execute machine within the pool. The result of a running interactive job is a shell prompt issued on the execute machine where the job runs. 

Neither the submit nor the execute host for interactive jobs may be on Windows platforms.

The current working directory of the shell will be the initial working directory of the running job. The shell type will be the default for the user that submits the job. At the shell prompt, X11 forwarding is enabled.

Each interactive job will have a job ClassAd attribute of
```bash
InteractiveJob = True
```

Submission of an interactive job specifies the option -interactive on the condor_submit command line.

A submit description file may be specified for this interactive job. Within this submit description file, a specification of these 5 commands will be either ignored or altered:

1. executable

2. transfer_executable

3. arguments

4. universe . The interactive job is a vanilla universe job.

5. queue <n>. In this case the value of <n> is ignored; exactly one interactive job is queued.

The submit description file may specify anything else needed for the interactive job, such as files to transfer.

If no submit description file is specified for the job, a default one is utilized as identified by the value of the configuration variable INTERACTIVE_SUBMIT_FILE .

Here are examples of situations where interactive jobs may be of benefit.

- An application that cannot be batch processed might be run as an interactive job. Where input or output cannot be captured in a file and the executable may not be modified, the interactive nature of the job may still be run on a pool machine, and within the purview of Condor.

- A pool machine with specialized hardware that requires interactive handling can be scheduled with an interactive job that utilizes the hardware.

- The debugging and set up of complex jobs or environments may benefit from an interactive session. This interactive session provides the opportunity to run scripts or applications, and as errors are identified, they can be corrected on the spot.

- Development may have an interactive nature, and proceed more quickly when done on a pool machine. It may also be that the development platforms required reside within Condor’s purview as execute hosts.

## Submitting Lots of Jobs

When submitting a lot of jobs with a single submit file, you can dramatically speed up submission and reduce the load on the condor_schedd by submitting the jobs as a late materialization job factory.

A submission of this form sends a single ClassAd, called the Cluster ad, to the condor_schedd, as well as instructions to create the individual jobs as variations on that Cluster ad. These instructions are sent as a submit digest and optional itemdata. The submit digest is the submit file stripped down to just the statements that vary between jobs. The itemdata is the arguments to the Queue statement when the arguments are more than just a count of jobs.

The condor_schedd will use the submit digest and the itemdata to create the individual job ClassAds when they are needed. Materialization is controlled by two values stored in the Cluster classad, and by optional limits configured in the condor_schedd.

The max_idle limit specifies the maximum number of non-running jobs that should be materialized in the condor_schedd at any one time. One or more jobs will materialize whenever a job enters the Run state and the number of non-running jobs that are still in the condor_schedd is less than this limit. This limit is stored in the Cluster ad in the JobMaterializeMaxIdle attribute.

The max_materialize limit specifies an overall limit on the number of jobs that can be materialized in the condor_schedd at any one time. One or more jobs will materialize when a job leaves the condor_schedd and the number of materialized jobs remaining is less than this limit. This limit is stored in the Cluster ad in the JobMaterializeLimit attribute.

Late materialization can be used as a way for a user to submit millions of jobs without hitting the MAX_JOBS_PER_OWNER or MAX_JOBS_PER_SUBMISSION limits in the condor_schedd, since the condor_schedd will enforce these limits by applying them to the max_materialize and max_idle values specified in the Cluster ad.

To give an example, the following submit file:
```bash
executable     = foo
arguments      = input_file.$(Process)

request_memory = 4096
request_cpus   = 1
request_disk   = 16383

error   = err.$(Process)
output  = out.$(Process)
log     = foo.log

should_transfer_files = yes
transfer_input_files = input_file.$(Process)

# submit as a factory with an idle jobs limit
max_idle = 100

# submit 15,000 instances of this job
queue 15*1000
```

When submitted as a late materialization factory, the submit digest for this factory will contain only the submit statments that vary between jobs, and the collapsed queue statement like this:

```bash
arguments = input_file.$(Process)
error = err.$(Process)
output = out.$(Process)
transfer_input_files = input_file.$(Process)

queue 15000
```


{: .source}

{: .solution}


{% include links.md %}

