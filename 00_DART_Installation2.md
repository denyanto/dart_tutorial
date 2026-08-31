# DART Configure on HPC TUNA-MMS
Configuration of DART on HPC

These are some installation notes taken in the process of installing DART on HPC TUNA-MMS using ROME. This tutorial is for **free**

## Module load and save as intel module
```console
cd $HOME
ml load mpi compiler
ml save intel
```

## Activate and load Intel oneAPI Compilers
```console
# Activate Intel oneAPI Compilers
ml restore intel

# Verify the compilers are active in your session
icc --version
ifort --version

# Load Intel oneAPI Compilers
export CC=icc
export CXX=icpc
export FC=ifort
export F77=ifort
export CFLAGS="-O2"
export CXXFLAGS="-O2"
export FCFLAGS="-O2"
export AR=/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin-llvm/llvm-ar
export RANLIB=/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin-llvm/llvm-ranlib
```


## Setting install location and library version
```console
INSTALL_DIR=/home/inanwp/.libs
NC_C_VERSION=4.10.1
NC_F_VERSION=4.6.4
HDF5_VERSION=2.0.0
ZLIB_VERSION=1.3.2
J=8
export LD_LIBRARY_PATH=$INSTALL_DIR/lib:$LD_LIBRARY_PATH
```

## Build and Install zlib
```console
wget https://zlib.net/zlib-$ZLIB_VERSION.tar.gz
tar xzf zlib-$ZLIB_VERSION.tar.gz
cd zlib-$ZLIB_VERSION
./configure --prefix=$INSTALL_DIR 
make -j$J
make install
cd ..
```

## Build and Install hdf5
```console
wget https://github.com/HDFGroup/hdf5/archive/refs/tags/$HDF5_VERSION.tar.gz
tar xzf $HDF5_VERSION.tar.gz
cd hdf5-$HDF5_VERSION
mkdir build && cd build
export HDF5_ROOT=$INSTALL_DIR
export CPPFLAGS="-I${HDF5_ROOT}/include"
export LDFLAGS="-L${HDF5_ROOT}/lib -Wl,-rpath,${HDF5_ROOT}/lib"
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
  -DBUILD_SHARED_LIBS=ON \
  -DBUILD_STATIC_LIBS=ON \
  -G "Unix Makefiles" \
  -DCMAKE_MAKE_PROGRAM=/usr/bin/gmake \
  -DCMAKE_C_COMPILER=$CC \
  -DCMAKE_AR=$AR \
  -DCMAKE_RANLIB=$RANLIB \
  -DHDF5_ROOT=$HDF5_ROOT \
  -DHDF5_BUILD_FORTRAN=ON \
  -DHDF5_BUILD_HL_LIB=ON \
  -DHDF5_ENABLE_ZLIB_SUPPORT=ON \
  -DCMAKE_PREFIX_PATH=$HDF5_ROOT
cmake --build . -j$J
cmake --install .
cd ../..
```

## Build and Install netcdf-c
```console
wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v$NC_C_VERSION.tar.gz
tar xzf v$NC_C_VERSION.tar.gz
cd netcdf-c-$NC_C_VERSION
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_AR=$AR \
  -DCMAKE_RANLIB=$RANLIB \
  -DNETCDF_ENABLE_DAP=OFF \
  -DBUILD_SHARED_LIBS=OFF \
  -DBUILD_STATIC_LIBS=ON \
  -DHDF5_ROOT=$INSTALL_DIR \
  -DZLIB_ROOT=$INSTALL_DIR \
  -DNETCDF_ENABLE_HDF5=ON \
  -DNETCDF_ENABLE_PARALLEL4=ON \
  -DNETCDF_ENABLE_TESTS=OFF
cmake --build . -j$J
cmake --install .
cd ../..
```

## Build and Install netcdf-fortran
```console
wget https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v$NC_F_VERSION.tar.gz
tar xzf v$NC_F_VERSION.tar.gz
cd netcdf-fortran-$NC_F_VERSION
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DBUILD_STATIC_LIBS=ON \
  -DNETCDF_ROOT=$INSTALL_DIR \
  -DENABLE_TESTS=OFF \
  -DCMAKE_C_COMPILER=mpiicx \
  -DCMAKE_Fortran_COMPILER=mpiifx
cmake --build . -j$J
cmake --install .
cd $INSTALL_DIR/lib
ln -sf $INSTALL_DIR/lib64/libnetcdf* .
cd 
```


## Verify the Netcdf Installation
```console
# Check the C configuration tool
nc-config --cc

# Check the Fortran configuration tool
nf-config --fc
which nc-config
```

## Downloading DART
```console
cd $HOME/apps
git clone https://github.com/NCAR/DART.git
cd DART
cp build_templates/mkmf.template.intel.linux build_templates/mkmf.template
```

Edit in the build_templates/mkmf.template file
```console
MPIFC = mpiifx
MPILD = mpiifx
FC = ifx
LD = ifx
NETCDF = $CONDA_PREFIX
INCS = -I$(NETCDF)/include
LIBS = -L$(NETCDF)/lib -lnetcdf -lnetcdff -lhdf5_hl -lhdf5 -lz -lmpi
FFLAGS = -O2 $(INCS)
LDFLAGS = $(FFLAGS) $(LIBS)
```

## Create conda environment **dart**
```console
conda create --name dart
conda activate dart
conda install -c conda-forge cmake make curl libcurl
conda install -c conda-forge mamba
mamba install -n dart -c conda-forge nco
mamba install -n dart -c conda-forge cdo
mamba install -n dart -c conda-forge ncl
```

Build and test DART 
```console
cd models/lorenz_63/work
./quickbuild.sh
```
