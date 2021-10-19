---
title: "Services for Running Jobs"
teaching: 10
exercises: 20
questions:
- "Good Practices"
objectives:
- "-"
keypoints:
- "-"
---

## Services for Running Jobs

Jobs can use these services to provide more reliable runs, to give logging and monitoring data for users, and to synchronize with other jobs. Note that different HTCondor job universes may provide different services. The functionality below is available in the vanilla universe, unless otherwise stated.

## Environment Variables

An HTCondor job running on a worker node does not, by default, inherit the environment variables from the machine it runs on or the machine it was submitted from. If it did, the environment might change from run to run, or machine to machine, and create non reproducible, difficult to debug problems.


The user may define environment variables for the job with the environment command in the submit file. 


In general, it is preferable to just declare the minimum set of needed environment variables with the environment command, as that clearly declares the needed environment variables. 

Commands within the submit description file may reference the environment variables of the submitter. Submit description file commands use $ENV(EnvironmentVariableName) to reference the value of an environment variable.

## Extra Environment Variables HTCondor sets for Jobs

Additional environment variables

- `_CONDOR_SCRATCH_DIR` names the directory where the job may place temporary data files. This directory is unique for every job that is run, and its contents are deleted by HTCondor when the job stops running on a machine. When file transfer is enabled, the job is started in this directory.

- `_CONDOR_SLOT` gives the name of the slot (for multicore machines), on which the job is run. On machines with only a single slot, the value of this variable will be 1, just like the SlotID attribute in the machine’s ClassAd. See the Policy Configuration for Execute Hosts and for Submit Hosts section for more details about configuring multicore machines.

- `_CONDOR_JOB_AD` is the path to a file in the job’s scratch directory which contains the job ad for the currently running job. The job ad is current as of the start of the job, but is not updated during the running of the job. The job may read attributes and their values out of this file as it runs, but any changes will not be acted on in any way by HTCondor. The format is the same as the output of the condor_q -l command. This environment variable may be particularly useful in a USER_JOB_WRAPPER.

- `_CONDOR_MACHINE_AD` is the path to a file in the job’s scratch directory which contains the machine ad for the slot the currently running job is using. The machine ad is current as of the start of the job, but is not updated during the running of the job. The format is the same as the output of the condor_status -l command. Interesting attributes jobs may want to look at from this file include Memory and Cpus, the amount of memory and cpus provisioned for this slot.

- `_CONDOR_JOB_IWD` is the path to the initial working directory the job was born with.

- `_CONDOR_WRAPPER_ERROR_FILE` is only set when the administrator has installed a USER_JOB_WRAPPER. If this file exists, HTCondor assumes that the job wrapper has failed and copies the contents of the file to the StarterLog for the administrator to debug the problem.

- `CUBACORES GOMAXPROCS JULIA_NUM_THREADS MKL_NUM_THREADS NUMEXPR_NUM_THREADS OMP_NUM_THREADS OMP_THREAD_LIMIT OPENBLAS_NUM_THREADS TF_LOOP_PARALLEL_ITERATIONS TF_NUM_THREADS` are set to the number of cpu cores provisioned to this job. Should be at least RequestCpus, but HTCondor may match a job to a bigger slot. Jobs should not spawn more than this number of cpu-bound threads, or their performance will suffer. Many third party libraries like OpenMP obey these environment variables.

- `X509_USER_PROXY` gives the full path to the X.509 user proxy file if one is associated with the job. Typically, a user will specify x509userproxy in the submit description file.

## Resource Limitations on a Running Job

HTCondor may configure the system a job runs on to prevent a job from using all the resources on a machine.

Jobs may see

- A private (non-shared) /tmp and /var/tmp directory

- A private (non-shared) /dev/shm

- A limit on the amount of memory they can allocate, above which the job may be placed on hold or evicted by the system.

- A limit on the amount of CPU cores the may use, above which the job may be blocked, and will run very slowly



{: .challenge_ brown box} 

{% include links.md %}

