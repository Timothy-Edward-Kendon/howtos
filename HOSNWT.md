# High Order Spectral Numerical Wave Tank (HOS-NWT)

## Installation on the HPC-Baloo

### Cloning the repo
- git clone git@github.com:LHEEA/HOS-NWT.git HOS-NWT

### Compilation

- Navigate to the HOS-NWT folder and edit the makefile

```sh
cd HOS-NWT
vim makefile
```

- Alter LINKLIB in the makefile to suit the installation

```sh
# for the HPC Baloo the following changes were made
FFTWLIBDIR  = /prog/sdpsoft/fftw-3.3.4/lib
BLASLIBDIR  = /prog/sdpsoft/lapack-3.7.0/lib
LINKLIB = ${FFTWLIBDIR}/libfftw3.a ${BLASLIBDIR}/liblapack.a ${BLASLIBDIR}/librefblas.a
```

- Close the makefile and ```make``` the project.

- To check the installation, the executable ```HOS-NWT``` should  be in the ```bin/``` folder of the NWT-HOS directory

## Running HOS-NWT

Example files can be found under ```HOS-NWT/Benchmark```.

1. Copy the  ```Irreg_2D``` example to ```$FOAM_RUN```
2. Type ```cd $FOAM_RUN/Irreg_2D``` to navigate to the run folder
3. Type ```mkdir Results``` to make a results folder
4. Call the ```HOS-NWT``` executable