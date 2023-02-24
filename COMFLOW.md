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
- [2. Running COMFLOW and SWD example](#2-running-comflow-and-swd-example)
- [3. Running the DNV-HOSM code](#3-running-the-dnv-hosm-code)

---

<br>

## 1.2. Pre-requisites

Apply for access to the following in [AccessIT](https://accessit.equinor.com/).

1. [UNIX BASIC ACCESS](https://accessit.equinor.com/Search/Search?term=UNIX+BASIC+ACCESS)
2. [HPC CLUSTER - RD](https://accessit.equinor.com/Search/Search?term=HPC+CLUSTER+-+RD)
3. [MAROPSIM](https://accessit.equinor.com/Search/Search?term=%2Fwork54%2FMarOpSim+-+marop+%28Stavanger%29)

Apply for these sequentially. Wait until access has been granted before you proceed to the next in the list.

<span style="color:red"> Recently there have been issues for all new users applying for HPC CLUSTER access. New users are finding that their ${HOME} profile is not being created on the HPC CLUSTER nodes. Consequently the user will hit issues logging onto the HPC. If you experience issues with the next step (`Accessing an HPC node`), then it is likely this issue. To resolve, new users will need to raise the issue with Services@Equinor.</span>

<br>

## 1.3. Setting up your local environment

<br>

### 1.3.1. Accessing an HPC node

Log onto a Linux machine via [https://rgs.statoil.no](https://rgs.statoil.no/main.cgi). In a Linux `Terminal`, type the command

`ls /prog/MarOpSim`

If the command returns the content of the directory, then you have logged onto a `dedicated-HPC-node`.

If the command replies that `the directory is not found`, then you have logged onto a `non-HPC-node`, in which case you need to log onto one of the top nodes of the HPC to submit your job to the cluster. To do that, type the following into a Linux Terminal.

`ssh <hpc-node-name>.st.statoil.no`

where `<hpc-node-name>` should be replaced with one of the following node names: `st-lcmtop01`, `st-lcmtop03`, `st-lcmtop03`, `st-lcmtop04`. You will need to enter your standard unix/windows password to log on. Once logged on, _in the same Linux Terminal_ that you ran the `ssh` command in, type the command

`ls /prog/MarOpSim`

If the command returns the content of the directory, then you have successfully `ssh'd` onto one of the HPC's top nodes.

If the `ls` command is still complaining that the directory is not found, check that you are using the same Linux Terminal that you used to `ssh` onto the `<hpc-node-name>` and, if you are, then try a different `<hpc-node-name>`.

<span style="color:red"> With the exception of the last section (`Viewing results on a non-HPC node`), the `Linux terminal` in the text below refers to the one logged onto the HPC node (dedicated or top node)</span>.

<br>

### 1.3.2. Configuring your SHELL

In a Linux Terminal, type the following:

```sh
echo $SHELL
```

If `bin/csh` is returned, then type

```sh
cd
cp -f /prog/MarOpSim/CFDTools/Comflow/.comflow .
source .comflow
echo "source .comflow" >> .cshrc
```

The last line setups up your HPC profile for all future sessions. These lines do NOT need to be re-typed everytime you log on to an HPC node.

If `bin/bash` was returned from the `echo $SHELL` command, you are currently out of luck as a setup for bash has not been tested (however new users should not hit this issue).

At this point, if you type `comflow -v` into the Linux you should now get the Comflow version number returned.

<br>

## 1.4. Running your first case

In a Linux Terminal, type the following:

```sh
cd
mkdir Comflow && cd Comflow
cp -r $COMFLOWDIR/examples . && cd examples
ls
```

The final `ls` command should return a list of examples. Choose one and navigate into the folder. In the following, the `reinecker_fenton_2d` example is chosen.

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

`qstat -u $USER`

This will return tabular information about your jobs. Under the column title 'S' for status, your job is complete when this is set to 'C' (completed). Other status flags are:

- Q = queued
- R = running

<br>

## 1.5. Viewing results with Paraview

Results are viewed using the Kitware Paraview program. How to view results will depend upon whether you originally logged onto a `dedicated-HPC-node` or a `non-HPC-node`.

<br>

### 1.5.1. Viewing results on a dedicated HPC node

In the Linux Terminal, type the command `paraview` and open the file `proc_1/output_summary`. Further information can be read here [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html).

<br>

### 1.5.2. Viewing results on a non-HPC node or Windows

Owing to issues with unmounting, using `sshfs` is no longer a recommended setup for accessing HPC data from a remote machine. For viewing HPC files on a non-HPC node or on Windows, results will need to be `scp`'d over to the respective machine.

---

<br>

Windows 10 should have a built-in ssh client. To use

1. Press `<Windows key> + R` simulataneously and type `cmd` to open a command prompt
2. In the command prompt type
   ```batch
   cd <path to local results folder>
   scp -r <username>@st-lcmtop03.st.statoil.no:<path to remote results folder> .
   ```
   where you need to replace the items in brackets `<>`(including brackets) with the relevant info. More information on scp can be found on google e.g. https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/.

In Linux a similar set of `cd` and `scp` commands can be run from the Terminal to copy files from one machine to another.

---

<br>

It is now recommended that you download the Paraview binaries from the https://www.paraview.org/download website. Binaries for Linux and Windows exist. In both cases you may wish to add the final executable to your system PATH e.g. on Linux

1. Open `${HOME}/.cshrc` with a text editor e.g. `vim ${HOME}/.cshrc`
2. At the end of the file type
   `setenv PATH <path to paraview bin directory>:${PATH}`
   where you replace `<path to paraview bin directory>` with the path to your paraview installation
3. Open a new terminal, and type `paraview` and the program should run.

If on Linux, an alternative to installing the binaries from the website would be to `scp` the paraview folder `/prog/MarOpSim/CFDTools/ParaView` on the HPC to a local folder on your Linux machine.

---

<br>

To install the COMFLOW plugin to Paraview, you will need to `scp` the folder `/prog/MarOpSim/CFDTools/Comflow/4.3.1/python` from the hpc-top-node to your local machine. On Windows 10:

1. Press `<Windows key> + R` simulataneously and type `cmd` to open a command prompt
2. In the command prompt type
   ```batch
   cd <path to the chosen local folder for storing paraview plugins>
   scp -r <username>@st-lcmtop03.st.statoil.no:/prog/MarOpSim/CFDTools/Comflow/4.3.1/python .
   ```
   where you need to replace the items in brackets `<>` (including brackets) with the relevant info. More information on scp can be found on google e.g. https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/

In Linux a similar set of `cd` and `scp` commands can be run from the Terminal to copy files from one machine to another.

---

<br>

How to add the COMFLOW plugin to Paraview is detailed in a general sense at [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html). In Paraview, select the menu `Tools/Manage plugins`. In the section `Local Plugins`, press the button `Load New` and select the file `<path to the chosen local folder for storing paraview plugins>/CMFPlugins.py`. COMFLOW's `CMFPlugin` plugin should now appear at the bottom of the list of registered plugins. Click the arrow to the side of the `CMFPlugin` plugin to expand the options, and make sure to check the `Auto Load` box to automatically load the plugin upon restart of Paraview. All ComFLOW plugins are available in the menu “Filters/ComFLOW”.

In Paraview, navigate to the COMFLOW results under the folder `<path to local results folder>` noted above. Further information can be read here [https://poseidon.housing.rug.nl/comflow/paraview.html](https://poseidon.housing.rug.nl/comflow/paraview.html).

# 2. Running COMFLOW and SWD example

This example uses an [SWD](https://github.com/SpectralWaveData/spectral_wave_data) output file from a high-order spectral method code to initialize the COMFLOW domain and to prescribe time-varying Dirichlet boundary conditions at the inlet and outline of the simulation domain.

To run the test case on the HPC Baloo, type the following in a Linux Terminal:

```sh
cd
mkdir Comflow && cd Comflow
cp -r /prog/MarOpSim/CFDTools/Comflow/4.3.1/examples/comflow_workshop/06_hosm_waves_in_comflow/ .
cd 06_hosm_waves_in_comflow
qsub run_comflow
```

After completion, in Paraview open the `proc_1/output_summary.xml` file to view the COMFLOW results, and open `proc_1/hosm_vtk/kinematics*vtk` files to view the HOSM results that were stored on the SWD file.

# 3. Running the DNV-HOSM code

To generate an SWD file for coupling with COMFLOW, the DNV-HOSM code can be run on the HPC cluster. In a Linux terminal type,

```sh
source ~/.comflow   # No need to do this if already sourced from before
cd
mkdir HOSM && cd HOSM
cp -r /prog/MarOpSim/CFDTools/HOS-DNVGL/tutorial .
cd tutorial
hosm_gfortran_static doctered.inp
```

To understand the input file `doctored.inp`, read the manual `/prog/MarOpSim/CFDTools/HOS-DNVGL/dnvglhosm.pdf`. In short, the `doctored.inp` file prescribes a JONSWAP spectrum Hs 7.6 Tp 13.0 in a water depth of 21 m. The horizontal extent of the domain is controlled by the largest wave number in the simulation that we wish to resolve and the number of grid points. The difference between the `orig.inp` (provided by DNV) and the `doctored.inp` relates to features that were not available with the installed HOSM version.

To generate a new test case, a base python script (`process.py`) has been provided which can be modified as per need. To generate a new input file, type the following where the last three parameters relate to Hs [m], Tp [s] and water depth [m] respectively.

```sh
  python process.py generate-input 11.5 15.3 25
```

This script generates an input file `hos.inp`, which can be run thus `hosm_gfortran_static hos.inp`
