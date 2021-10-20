---
title: "Submitting a Job"
teaching: 10
exercises: 30
questions:
- "The worst thing about prison was the dementors"
- "Improversation"
objectives:
- "Retrieve jobs"
- ""
keypoints:
- "y"
- ""
---
## Submitting a Job

The `condor_submit` command takes a job description file as input and submits the job to HTCondor.  Items such as the name of the executable to run, the initial working directory, and command-line arguments to the program all go into the submit description file. `condor_submit` creates a job ClassAd based upon the information, and HTCondor works toward running the job.

It is easy to submit multiple runs of a program to HTCondor with a single submit description file. To run the same program many times with different input data sets, arrange the data files accordingly so that each run reads its own input, and each run writes its own output. Each individual run may have its own initial working directory, files mapped for stdin, stdout, stderr, command-line arguments, and shell environment.

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


The submit description file for this example queues 150 runs of program foo.

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

## Submitting many similar jobs with one queue command

A wide variety of job submissions can be specified with extra information to the queue submit command. This flexibility eliminates the need for a job wrapper or Perl script for many submissions.

The form of the queue command defines variables and expands values, identifying a set of jobs. Square brackets identify an optional item.

queue [ < int expr > ]

queue [ < int expr > ] [ < varname > ] in [ slice ] < list of items >

queue [ < int expr > ] [ < varname > ] matching [ files | dirs ] [ slice ] < list of items with file globbing >

queue [ < int expr > ] [ < list of varnames > ] from [ slice ] < file name > | < list of items >

All optional items have defaults:


- If < int expr > is not specified, it defaults to the value 1.

- If < varname > or < list of varnames > is not specified, it defaults to the single variable called ITEM.

- If slice is not specified, it defaults to all elements within the list. This is the Python slice [::], with a step value of 1.

- If neither files nor dirs is specified in a specification using the from key word, then both files and directories are considered when globbing.

The list of items uses syntax in one of two forms. One form is a comma and/or space separated list; the items are placed on the same line as the queue command. The second form separates items by placing each list item on its own line, and delimits the list with parentheses. The opening parenthesis goes on the same line as the queue command. The closing parenthesis goes on its own line. The queue command specified with the key word from will always use the second form of this syntax. Example 3 below uses this second form of syntax. Finally, the key word from accepts a shell command in place of file name, followed by a pipe | (example 4).

The optional slice specifies a subset of the list of items using the Python syntax for a slice. Negative step values are not permitted.

Here are a set of examples.

### Example 1

```bash
transfer_input_files = $(filename)
arguments            = -infile $(filename)
queue filename matching files *.dat
```

The use of file globbing expands the list of items to be all files in the current directory that end in .dat. Only files, and not directories are considered due to the specification of files. One job is queued for each file in the list of items. For this example, assume that the three files initial.dat, middle.dat, and ending.dat form the list of items after expansion; macro filename is assigned the value of one of these file names for each job queued. That macro value is then substituted into the arguments and transfer_input_files commands. The queue command expands to

```
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

### Example 2

```bash 
queue 1 input in A, B, C
```

Variable input is set to each of the 3 items in the list, and one job is queued for each. For this example the queue command expands to

```bash
input = A
queue
input = B
queue
input = C
queue
````

Example 3

```bash
queue input, arguments from (
  file1, -a -b 26
  file2, -c -d 92
)
```
Using the from form of the options, each of the two variables specified is given a value from the list of items. For this example the queue command expands to

```bash
input = file1
arguments = -a -b 26
queue
input = file2
arguments = -c -d 92
queue
```

## Example 4

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
## Including Submit Commands Defined Elsewhere

Externally defined submit commands can be incorporated into the submit description file using the syntax

```bash
include : <what-to-include>
```

The <what-to-include> specification may specify a single file, where the contents of the file will be incorporated into the submit description file at the point within the file where the include is. Or, <what-to-include> may cause a program to be executed, where the output of the program is incorporated into the submit description file. The specification of <what-to-include> has the bar character (|) following the name of the program to be executed.

The include key word is case insensitive. There are no requirements for white space characters surrounding the colon character.

Included submit commands may contain further nested include specifications, which are also parsed, evaluated, and incorporated. Levels of nesting on included files are limited, such that infinite nesting is discovered and thwarted, while still permitting nesting.

Consider the example

```bash
include : ./list-infiles.sh |
```

In this example, the bar character at the end of the line causes the script list-infiles.sh to be invoked, and the output of the script is parsed and incorporated into the submit description file. If this bash script is in the PATH when submit is run, and contains

```bash
#!/bin/sh

echo "transfer_input_files = `ls -m infiles/*.dat`"
exit 0
```

then the output of this script has specified the set of input files to transfer to the execute host. For example, if directory infiles contains the three files A.dat, B.dat, and C.dat, then the submit command

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

- the defined keyword followed by the name of a variable. If the variable is defined, the statement(s) are incorporated into the expanded input. If the variable is not defined, the statement(s) are not incorporated into the expanded input. As an example,

```bash
    if defined MY_UNDEFINED_VARIABLE
       X = 12
    else
       X = -1
    endif
```

results in X = -1, when MY_UNDEFINED_VARIABLE is not yet defined.

- the version keyword, representing the version number of of the daemon or tool currently reading this conditional. This keyword is followed by an HTCondor version number. That version number can be of the form x.y.z or x.y. The version of the daemon or tool is compared to the specified version number. The comparison operators are

    - '==' for equality. Current version 8.2.3 is equal to 8.2.

    - '>=' to see if the current version number is greater than or equal to. Current version 8.2.3 is greater than 8.2.2, and current version 8.2.3 is greater than or equal to 8.2.

    - '<=' to see if the current version number is less than or equal to. Current version 8.2.0 is less than 8.2.2, and current version 8.2.3 is less than or equal to 8.2.

    As an example,

    ```bash
    if version >= 8.1.6
       DO_X = True
    else
       DO_Y = True
    endif
    ```

    results in defining DO_X as True if the current version of the daemon or tool reading this if statement is 8.1.6 or a more recent version.

    - True or yes or the value 1. The statement(s) are incorporated.

    - False or no or the value 0 The statement(s) are not incorporated.

    - $(<variable>) may be used where the immediately evaluated value is a simple boolean value. A value that evaluates to the empty string is considered False, otherwise a value that does not evaluate to a simple boolean value is a syntax error.

The syntax

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

Here is an example use of a conditional in the submit description file. A portion of the sample.sub submit description file uses the if/else syntax to define command line arguments in one of two ways:

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

```
arguments = -n 1 -debug
```


## Interactive Jobs

An interactive job is a Condor job that is provisioned and scheduled like any other vanilla universe Condor job onto an execute machine within the pool. The result of a running interactive job is a shell prompt issued on the execute machine where the job runs. The user that submitted the interactive job may then use the shell as desired, perhaps to interactively run an instance of what is to become a Condor job. This might aid in checking that the set up and execution environment are correct, or it might provide information on the RAM or disk space needed. This job (shell) continues until the user logs out or any other policy implementation causes the job to stop running. A useful feature of the interactive job is that the users and jobs are accounted for within Condor’s scheduling and priority system.

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

