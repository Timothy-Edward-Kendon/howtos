# 1. Running COMFLOW on the Baloo HPC

**Table of Contents**
- [1. Running COMFLOW on the Baloo HPC](#1-running-comflow-on-the-baloo-hpc)
  - [1.2. Pre-requisites](#12-pre-requisites)
  - [1.3. Setting up your local environment](#13-setting-up-your-local-environment)
    - [1.3.1. Accessing an HPC node](#131-accessing-an-hpc-node)
    - [1.3.2. Configuring your SHELL](#132-configuring-your-shell)
  - [1.4. Running your first case](#14-running-your-first-case)
    - [1.4.1. Checking the status of the run](#141-checking-the-status-of-the-run)
  - [1.5. Viewing results with Paraview](#15-viewing-results-with-paraview)
    - [1.5.1. Viewing results on a dedicated HPC node](#151-viewing-results-on-a-dedicated-hpc-node)
    - [1.5.2. Viewing results on a non-HPC node or Windows](#152-viewing-results-on-a-non-hpc-node-or-windows)

---

<br>

## 1.2. Pre-requisites

Apply for access to the following in [AccessIT](https://accessit.equinor.com/).

1. [UNIX BASIC ACCESS](https://accessit.equinor.com/Search/Search?term=UNIX+BASIC+ACCESS)
2. [HPC CLUSTER - RD](https://accessit.equinor.com/Search/Search?term=HPC+CLUSTER+-+RD)
3. [MAROPSIM](https://accessit.equinor.com/Search/Search?term=%2Fwork54%2FMarOpSim+-+marop+%28Stavanger%29)

Apply for these sequentially. Wait until access has been granted before you proceed to the next in the list. 

<span style="color:red">  Recently there have been issues for all new users applying for HPC CLUSTER access. New users are finding that their  ${HOME} profile is not being created on the HPC CLUSTER nodes. Consequently the user will hit issues logging onto the HPC. If you experience issues with the next step (```Accessing an HPC node```), then it is likely this issue. To resolve, new users will need to raise the issue with Services@Equinor.</span> 

<br>

## 1.3. Setting up your local environment

<br>

### 1.3.1. Accessing an HPC node


Log onto a Linux machine via [https://rgs.statoil.no](https://rgs.statoil.no/main.cgi). In a Linux ```Terminal```, type the command

```ls /prog/MarOpSim```

If the command returns the content of the directory, then you have logged onto a ```dedicated-HPC-node```. 

If the command replies that ```the directory is not found```, then you have logged onto a ```non-HPC-node```, in which case you need to log onto one of the top nodes of the HPC to submit your job to the cluster. To do that, type the following into a Linux Terminal. 

```ssh <hpc-node-name>.st.statoil.no```

where ```<hpc-node-name>``` should be replaced with one of the following node names: ```st-lcmtop01```, ```st-lcmtop03```, ```st-lcmtop03```, ```st-lcmtop04```. You will need to enter your standard unix/windows password to log on. Once logged on, *in the same Linux Terminal* that you ran the ```ssh``` command in, type the command

```ls /prog/MarOpSim```

If the command returns the content of the directory, then you have successfully ```ssh'd```  onto one of the HPC's top nodes. 

If the ```ls``` command is still complaining that the directory is not found, check that you are using the same Linux Terminal that you used to ```ssh``` onto the ```<hpc-node-name>``` and, if you are, then try a different ```<hpc-node-name>```.

<span style="color:red">  With the exception of the last section (```Viewing results on a non-HPC node```), the ```Linux terminal``` in the text below refers to the one logged onto the HPC node (dedicated or top node)</span>.  

<br>

### 1.3.2. Configuring your SHELL

In a Linux Terminal, type the following:

```sh
echo $SHELL
```

If ```bin/csh``` is returned, then type


```sh
cd
cp -f /prog/MarOpSim/CFDTools/Comflow/.comflow .
source .comflow
echo "source .comflow" >> .cshrc
```

The last line setups up your HPC profile for all future sessions. These lines do NOT need to be re-typed everytime you log on to an HPC node. 

If ```bin/bash``` was returned from the ```echo $SHELL``` command, you are currently out of luck as a setup for bash has not been tested (however new users should not hit this issue).

At this point, if you type ```comflow -v``` into the Linux you should now get the Comflow version number returned. 

<br>

## 1.4. Running your first case

In a Linux Terminal, type the following:

```sh
cd
mkdir Comflow && cd Comflow
cp -r $COMFLOWDIR/examples . && cd examples
ls
```

The final `ls` command should return a list of examples. Choose one and navigate into the folder. In the following, the ```reinecker_fenton_2d``` example is chosen. 

```sh
cd reinecker_fenton_2d
```

Now type the following to submit the job to the cluster

```sh 
qsub run_comflow
```

The job is now submitted to the cluster. 

<br>

### 1.4.1. Checking the status of the run

In the Linux Terminal, type the following 

```qstat -u $USER```

This will return tabular information about your jobs. Under the column title 'S' for status, your job is complete when this is set to 'C' (completed). Other status flags are:

- Q = queued
- R = running

<br>

## 1.5. Viewing results with Paraview

Results are viewed using the Kitware Paraview program. How to view results will depend upon whether you originally logged onto a ```dedicated-HPC-node``` or a ```non-HPC-node```.

<br>

### 1.5.1. Viewing results on a dedicated HPC node

In the Linux Terminal, type the command ```paraview``` and open the file ```proc_1/output_summary```. Further information can be read here [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html).

<br>

### 1.5.2. Viewing results on a non-HPC node or Windows

In the following, ```Linux Terminal``` refers to one that is NOT logged onto the ```HPC-top-node```. These instructions are outdated. 

In that Linux Terminal, type the following 

```sh
cd
mkdir hpc
sshfs $USER@st-lcmtop03.st.statoil.no:/ ~/hpc/
```



The above will create a directory ```hpc``` under you ${HOME} directory and map the (remote) hpc file system under that folder. <span style="color:red">**It is likely  that the last line above needs to be typed everytime you log onto a new ```non-HPC-node```**</span>.  


Now in the Linux Terminal, type

```sh
paraview
```

This will bring up a slightly older version of Paraview than is installed on the ```dedicated-HPC-node```. This version of Paraview is too old. The COMFLOW plugins to Paraview require Paraview version 5.6 or newer. Assuming that the plugins are to be used, close this version of Paraview.

To access a newer version of Paraview, one option is to download the paraview binaries from the https://www.paraview.org/download website; however, in this setup, the paraview version installed on the HPC can be used as we have access to it through the "mapped" file system. 

In the Linux terminal, type the following

```sh
cd
cp -f /private/${USER}/hpc/prog/MarOpSim/CFDTools/Comflow/.paraview_remote .
source .paraview_remote
echo "source .paraview_remote" >> .cshrc
```

The user profile should now be setup to run the HPC version of Paraview. Typing ```paraview``` in the Linux Terminal should now bring the up the HPC version of paraview. Although the executable will run locally, this will take a little longer to load (~30 seconds), so be patient. 

How to add the COMFLOW plugin to Paraview is detailed in a general sense at [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html). In Paraview, select the menu ```Tools/Manage plugins```. In the section ```Local Plugins```, press the button ```Load New``` and select the file ```~/hpc/prog/MarOpSim/CFDTools/Comflow/python/CMFPlugins.py```. COMFLOW's ```CMFPlugin``` plugin should now appear at the bottom of the list of registered plugins. Click the arrow to the side of the ```CMFPlugin``` plugin to expand the options, and make sure to check the ```Auto Load``` box to automatically load the plugin upon restart of Paraview. All ComFLOW plugins are available in the menu “Filters/ComFLOW”.

In Paraview, navigate to the COMFLOW results which will reside somewhere under the hpc folder. For the example run above, the run-folder can be found at ```hpc/private/<your user name>/Comflow/examples/reinecker_fenton_2d. In paraview```. Open the file ```proc_1/output_summary``` under that folder. Further information can be read here [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html).

