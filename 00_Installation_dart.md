# DART Configure on HPC
Configuration of DART on HPC

These are some installation notes taken in the process of installing DART. This tutorial is for **free**

## Module load and save as intel module
```console
cd $HOME
ml load oneapi/tbb/2022.2 oneapi/compiler-rt/2025.2.1 oneapi/umf/0.11.0 oneapi/compiler-intel-llvm/latest oneapi/mpi/2021.16
ml save intel
```

## Create conda environment **dart**
```console
conda create --name dart
conda activate dart
conda install -c conda-forge cmake make hdf5 curl zlib libcurl
```


## Setting install location and library version
```console
INSTALL_DIR=$CONDA_PREFIX
NC_C_VERSION=4.10.1
NC_F_VERSION=4.6.4
HDF5_VERSION=1.14.0
ZLIB_VERSION=1.3.2
J=8
```

## Activate and load Intel oneAPI Compilers
```console
# Activate Intel oneAPI Compilers
ml restore intel

# Verify the compilers are active in your session
icx --version
ifx --version

# Load Intel oneAPI Compilers
export CC=icx
export CXX=icpx
export FC=ifx
export F77=ifx
export CFLAGS="-O2 -march=native"
export CXXFLAGS="-O2 -march=native"
export FCFLAGS="-O2 -march=native"
```

## Build and Install zlib
```console
wget https://zlib.net/zlib-$ZLIB_VERSION.tar.gz
tar xzf zlib-$ZLIB_VERSION.tar.gz
cd zlib-$ZLIB_VERSION
./configure --prefix=$INSTALL_DIR --static
make -j$J
make install
cd ..
```

## Build and Install hdf5
```console
wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.14/hdf5-$HDF5_VERSION/src/hdf5-$HDF5_VERSION.tar.gz
tar xzf hdf5-$HDF5_VERSION.tar.gz
cd hdf5-$HDF5_VERSION
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DHDF5_BUILD_STATIC_LIBS=ON \
  -DHDF5_BUILD_TOOLS=OFF \
  -DHDF5_ENABLE_Z_LIB_SUPPORT=ON
make -j$J #cmake --build . -j$J
make install #cmake --install .
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
  -DENABLE_NETCDF_4=ON \
  -DENABLE_DAP=OFF \
  -DBUILD_SHARED_LIBS=OFF \
  -DBUILD_STATIC_LIBS=ON \
  -DHDF5_ROOT=$INSTALL_DIR
make -j$J #cmake --build . -j$J
make install #cmake --install .
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
  -DNetCDF_ROOT=$INSTALL_DIR \
  -DENABLE_TESTS=OFF
cmake --build . -j$J
cmake --install .
cd ../..
```

## Verify the Netcdf Installation
```console
# Check the C configuration tool
nc-config --cc

# Check the Fortran configuration tool
nf-config --fc
```

## Downloading DART
```console
cd $HOME
git clone https://github.com/NCAR/DART.git
cd DART
cp build_templates/mkmf.template.ifx.linux build_templates/mkmf.template
```

Edit in the build_templates/mkmf.template file
```console
MPIFC = mpiifx
MPILD = mpiifx
FC = ifx
LD = ifx
NETCDF = $CONDA_PREFIX
INCS = -I$(NETCDF)/include
LIBS = -L$(NETCDF)/lib -lnetcdf -lnetcdff
FFLAGS = -O2 $(INCS)
LDFLAGS = $(FFLAGS) $(LIBS)
```

Build and test DART 
```console
cd models/lorenz_63/work
./quickbuild.sh
```
