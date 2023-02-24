# Grid2Grid

## Installation on the HPC-Baloo

### Cloning the repo
```sh
git clone https://github.com/LHEEA/Grid2Grid.git
```

### Installing FFTW

- Using sdpsoft's installation of FFTW.


### Installing HDF5

- [Download HDF5 library from Cmake](https://support.hdfgroup.org/HDF5/release/cmakebuild.html) and extract to a folder ```$HDF5```.
- ```cd $HDF5``` and add the following lines in a file "HDF5options.cmake"

```sh
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DCMAKE_Fortran_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/gfortran")
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DCMAKE_CXX_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/g++")
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DCMAKE_C_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/gcc")
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DHDF5_BUILD_FORTRAN:BOOL=ON")
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DBUILD_SHARED_LIBS:BOOL=ON")
set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DHDF5_BUILD_CPP_LIB:BOOL=ON")
```

- Run build script ```sh build-unix.sh```

For the HPC Baloo, the ```HDF5``` folder is currently ```/prog/MarOpSim/CFDTools/ThirdParty-Common/CMake-hdf5-1.10.1```. 

### Installing Grid2Grid

- Navigate to the ```$Grid2Grid``` folder containing the cloned repository.
- Add the following lines to the top of the ```CMakeList.txt``` file

```sh
set(HDF5_LIB_PATH /prog/MarOpSim/CFDTools/ThirdParty-Common/CMake-hdf5-1.10.1/build/bin)
set(FFTW3_LIB_PATH /prog/sdpsoft/fftw-3.3.4/lib)
```
- Set your OpenFOAM environment (```of1912```) so that ```$FOAM_USER_LIBBIN``` is set.
- Then run the commands:

```sh
cmake -H. -BbuildOF -DBUILD_OF_LIB=ON -DHDF_LIBRARY:STRING="ON"
cmake --build buildOF
```

- To check the installation

```sh
ls -al $FOAM_USER_LIBBIN
# libGrid2Grid.so should be one of the items listed in the folder
```
