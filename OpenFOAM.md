# OpenFOAM

## Important links

- [OpenFOAM.org source repository](https://openfoam.org/download/source/)
- [Thirdparty on Github](https://github.com/OpenFOAM/ThirdParty-dev/)

## Installation and maintenance on HPC Baloo (Centos 7)


---

### Download the main source codes (development version)

```sh
cd /prog/MarOpSim/CFDTools/
git clone git@github.com:OpenFOAM/OpenFOAM-dev.git
git clone git@github.com:OpenFOAM/ThirdParty-dev.git
source /prog/MarOpSim/CFDTools/OpenFOAM-dev/etc/cshrc
```

---

### ThirdParty libraries

OpenFOAM relies some third-party software packages (in addition to OpenMPI) for some important tasks:

* Scotch and PT-Scotch for domain decomposition for parallel running (recommended/essential).
* ParaView visualization application (essential, without an alternative, compatible visualisation tool)
* CGAL Computational Geometry Algorithms Library used by the experimental mesher, foamyHexMesh (not essential).

---
#### **Compilation of Scotch**

To compile Scotch and PT-Scotch, 

```sh
cd $WM_THIRD_PARTY_DIR
```

Correct the following in Allwmake

```diff
67c67
<     if [ -r $MPI_ARCH_PATH/lib/libmpi.so ]
---
>     if [ -r $MPI_ARCH_PATH/lib${WM_COMPILER_LIB_ARCH}/libmpi.so ]
```

Run

```sh
./Allwmake
```

---
#### **Compilation of CGAL for foamyHexMesh**

CGAL requires MPFR and sdpsoft's version on the cluster is too old. Check https://github.com/OpenFOAM/ThirdParty-dev and replace the CGAL version number below accordingly. Which version of MPFR to download is of course dependent on the CGAL version. [As an alternative yumdownloader could be used to install missing system packages locally.]. 

```sh
# Installation directory (reference config.sh CGAL)
#export THIRD_PARTY_INSTALL_DIR=$WM_THIRD_PARTY_DIR/platforms/$WM_ARCH$WM_COMPILER$WM_PRECISION_OPTION$WM_LABEL_OPTION

export MPFR_VERSION=4.1.0
export CGAL_VERSION=5.0.2
export THIRD_PARTY_INSTALL_DIR=$WM_PROJECT_INST_DIR/ThirdParty-Common
export MPFR_INSTALL_DIR=$THIRD_PARTY_INSTALL_DIR/mpfr-$MPFR_VERSION
export CGAL_INSTALL_DIR=$THIRD_PARTY_INSTALL_DIR/CGAL-$CGAL_VERSION
export Qt5_DIR=/usr/lib64/cmake/Qt5 # required by CGAL

mkdir -p $THIRD_PARTY_INSTALL_DIR/build
cd $THIRD_PARTY_INSTALL_DIR/build

# Download and install MPFR
wget https://www.mpfr.org/mpfr-current/mpfr-$MPFR_VERSION.tar.xz 
tar -xJvf mpfr-$MPFR_VERSION.tar.xz
rm -f mpfr-$MPFR_VERSION.tar.xz
cd mpfr-$MPFR_VERSION
./configure --with-gmp-include=/prog/sdpsoft/gmp-6.1.2/include --with-gmp-lib=/prog/sdpsoft/gmp-6.1.2/lib --prefix=$MPFR_INSTALL_DIR
make && make install
cd $THIRD_PARTY_INSTALL_DIR/build && rm -rf mpfr-$MPFR_VERSION


# Download and install CGAL
cd $THIRD_PARTY_INSTALL_DIR/build
wget https://github.com/CGAL/cgal/releases/download/releases%2FCGAL-$CGAL_VERSION/CGAL-$CGAL_VERSION.tar.xz
tar -xJvf CGAL-$CGAL_VERSION.tar.xz
rm -f CGAL-$CGAL_VERSION.tar.xz
# Compile CGAL (need gmp.h in path in addition to GMP_INCLUDE_DIR)
cd CGAL-$CGAL_VERSION


# OpenFOAM doesn't use any of the CGAL image library and the ImageIO library 
# adds in an unnecessary OpenGL dependency
mkdir build && cd build
cmake   -DWITH_examples=OFF \
        -DWITH_demos=OFF \
        -DGMP_LIBRARIES_DIR:PATH=/prog/sdpsoft/gmp-6.1.2/lib \
        -DGMP_INCLUDE_DIR:PATH=/prog/sdpsoft/gmp-6.1.2/include \
        -DGMP_LIBRARIES:PATH=/prog/sdpsoft/gmp-6.1.2/lib/libgmp.so \
        -DMPFR_LIBRARIES_DIR:PATH=$MPFR_INSTALL_DIR/lib \
        -DMPFR_INCLUDE_DIR:PATH=$MPFR_INSTALL_DIR/include \
        -DMPFR_LIBRARIES:PATH=$MPFR_INSTALL_DIR/lib/libmpfr.so \
        -DBoost_INCLUDE_DIR:PATH=/prog/sdpsoft/boost-1.66.0/include \
        -DBoost_LIBRARIES:PATH=/prog/sdpsoft/boost-1.66.0/lib \
        -DCMAKE_INSTALL_PREFIX=$CGAL_INSTALL_DIR \
        -DCGAL_HEADER_ONLY=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DWITH_CGAL_Core=OFF \
        -DWITH_CGAL_ImageIO=OFF \
        -DWITH_CGAL_Qt5=OFF ..
make && make install
cd $THIRD_PARTY_INSTALL_DIR/build && rm -rf CGAL-$CGAL_VERSION
```

---
#### **Installation of ParaView**

OpenFOAM is built towards a specific paraview version which has been augmented with a specific reader. 

```sh
cd $WM_THIRD_PARTY_DIR
wget https://paraview.org/files/v$ParaView_MAJOR/ParaView-v$ParaView_VERSION.tar.xz
tar -xJvf ParaView-v$ParaView_VERSION.tar.xz
rm -rf ParaView-v$ParaView_VERSION.tar.xz
cd ParaView-v$ParaView_VERSION
mkdir build && cd build
cmake   -DCMAKE_INSTALL_PREFIX:PATH=$WM_THIRD_PARTY_DIR/platforms/linux64Gcc/ParaView-5.8.0 \
        -DPARAVIEW_BUILD_SHARED_LIBS:BOOL=ON \
        -DPARAVIEW_INSTALL_DEVELOPMENT_FILES:BOOL=ON \
        -DBUILD_TESTING:BOOL=OFF \
        -DPARAVIEW_USE_PYTHON=OFF \
        -DPARAVIEW_USE_QT=ON \
        -DQt5X11Extras_DIR=/private/tike/Software/LocalInstall/usr/lib64/cmake/Qt5X11Extras \
        -DQt5Help_DIR=/private/tike/Software/LocalInstall/usr/lib64/cmake/Qt5Help \
        -DQt5Svg_DIR=/private/tike/Software/LocalInstall/usr/lib64/cmake/Qt5Svg \
        -DCMAKE_BUILD_TYPE:STRING=Release \
        -DCMAKE_C_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/gcc \
        -DCMAKE_CXX_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/g++ \
        -DCMAKE_Fortran_COMPILER=/prog/sdpsoft/gcc-7.3.0/bin/gfortran \
        ..
make && make install
```

Alternatively don't bother with paraFoam and just use a binary downloaded from [Kitware](https://www.paraview.org/download/). This is the approach being adopted for general use. 

```sh
# establish a general directory
export PARAVIEW_DIR="/prog/MarOpSim/CFDTools/ParaView/"
# download paraview version e.g.
firefox https://www.paraview.org/download/ & # and browse/download
export PARAVIEW_VERSION="ParaView-5.8.1-MPI-Linux-Python3.7-64bit"
# unpack
tar -zxvf $PARAVIEW_VERSION.tar.gz # unpack (replace filename with downloaded version)
rm -f ParaView-5.8.1-MPI-Linux-Python3.7-64bit.tar.gz
# correct library path when using
cd ParaView-5.8.1-MPI-Linux-Python3.7-64bit/lib
export LD_LIBRARY_PATH=${PWD}:${LD_LIBRARY_PATH} # add this to /prog/MarOpSim/CFDTools/OpenFOAM-dev/etc/cshrc
cd $PARAVIEW_DIR/$PARAVIEW_VERSION/bin
export PATH=${PWD}:${PATH} # add this to /prog/MarOpSim/CFDTools/OpenFOAM-dev/etc/cshrc
paraFoam -builtin # of course ./paraview will also work
```

As noted above, if binaries ...

```sh
cd $FOAM_ETC
emacs -nw config.sh/paraview
# Add the following to the end of the file (save and close)
setenv ParaView_DIR "/prog/MarOpSim/CFDTools/ParaView/ParaView-5.8.1-MPI-Linux-Python3.7-64bit/"
setenv LD_LIBRARY_PATH ${ParaView_DIR}/lib:${LD_LIBRARY_PATH}
setenv PATH ${ParaView_DIR}/bin:${PATH}
```
---

### Compile OpenFOAM (development version)

```sh
cd $WM_PROJECT_DIR
export WM_NCOMPPROCS=12
./Allwmake -j 12
```

### Pull Latest build

```sh
cd $WM_PROJECT_DIR
export WM_NCOMPPROCS=12
git pull
./Allwmake -update -j 12
```

---

### Download the source codes (specific release)

```sh
wget -O - http://dl.openfoam.org/source/8 | tar xvz
wget -O - http://dl.openfoam.org/third-party/8 | tar xvz
mv OpenFOAM-8-version-8 OpenFOAM-8
mv ThirdParty-8-version-8 ThirdParty-8
```

---

### Testing installation

```sh
mkdir -p $FOAM_RUN
run
cp -r $FOAM_TUTORIALS/multiphase/interFoam/laminar/damBreak/damBreak .
cd damBreak
./Allrun

# worked
foamCloneCase
mpirun -np 4 interFoam -parallel > log &
```


## Configuring

To limit duplication, setup a `ThirdParty-Common`, soft link, and use foamConfigurePaths


```sh
# Download the latest ThirdParty-vXXXX.tgz from openfoam.com
# If ThirdParty-Common does not exist
mv ThirdParty-vXXXX ThirdParty-Common 
ln -s ThirdParty-Common ThirdParty-vXXXX # then build
# Otherwise unzip to ThirdParty_tmp







```