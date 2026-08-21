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
export CFLAGS="-O2 -march=native"
export CONDA_PREFIX=$CONDA_PREFIX
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX \
  -DCMAKE_C_COMPILER=icx \
  -DCMAKE_C_FLAGS="-O2 -march=native" \
  -DBUILD_SHARED_LIBS=OFF \
  -DBUILD_TESTING=OFF \
  -DENABLE_NETCDF_4=ON \
  -DENABLE_DAP=OFF \
  -DENABLE_BYTERANGE=OFF \
  -DENABLE_NCZARR=OFF

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
  -DCMAKE_INSTALL_PREFIX=${CONDA_PREFIX} \
  -DCMAKE_Fortran_COMPILER=ifx \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_Fortran_FLAGS="-O2 -march=native" \
  -DNetCDF_C_LIBRARY=${CONDA_PREFIX}/lib/libnetcdf.a \
  -DNetCDF_C_INCLUDE_DIR=${CONDA_PREFIX}/include \
  -DENABLE_TESTS=OFF
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
