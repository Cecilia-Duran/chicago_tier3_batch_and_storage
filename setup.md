---
title: Setup
---
This lesson describes how to manage jobs at the UChicago Analysis Facility

# Quick Start

Just log in to the UChicago Analysis Facility 
~~~
ssh -Y <your_user_name>@login.af.uchicago.edu
~~~

Create a directory for the examples (optional)
~~~
mkdir htcondor_module
~~~

For this tutorial we will work at the /home area  but remember to move to the /work (or /data) area when you use bigger data files.

small files (git repositories, source code, text files, configuration files, etc) ---> $HOME
large files (input data, output, etc) ----> $WORK (or $DATA)

> ## What will be covered in this tutorial?
>
> - What is HTCondor?
> 
> - Running a job with HTCondor
> 
> - Submitting multiple jobs with HTCondor
> 
> - How HTCondor mathces and runs jobs
> 
> - Testing and trouble shouting
> 
> - Automation 
>
{: .output}
{: .language-bash}


{% include links.md %}
