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

## Activate Intel oneAPI Compilers
```console
ml restore intel
# Verify the compilers are active in your session
icx --version
ifx --version
```

## Build and Install netcdf-c
```console
git clone https://github.com/Unidata/netcdf-c.git
cd netcdf-c
export CC=icx
export CXX=icpx
export CFLAGS="-O3 -xHost"
export CONDA_PREFIX=$CONDA_PREFIX
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX \
         -DCMAKE_PREFIX_PATH=$CONDA_PREFIX \
         -DENABLE_DAP=ON

make -j 4
make install
cd ../..
```

## Build and Install netcdf-fortran
```console
git clone https://github.com/Unidata/netcdf-fortran.git
cd netcdf-fortran
export CC=icx
export FC=ifx
export F77=ifx
export CPPFLAGS="-I${CONDA_PREFIX}/include"
export LDFLAGS="-L${CONDA_PREFIX}/lib"
export LD_LIBRARY_PATH="${CONDA_PREFIX}/lib:${LD_LIBRARY_PATH}"
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX \
  -DNETCDF_C_LIBRARY=$CONDA_PREFIX/lib/libnetcdf.so \
  -DNETCDF_C_INCLUDE_DIR=$CONDA_PREFIX/include
make -j 4
make install
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
```

Edit in the DART/build_templates/mkmf.template file
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
