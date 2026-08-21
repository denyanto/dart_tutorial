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

make -j$(nproc)
make install
cd ../..
```
