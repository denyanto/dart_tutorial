# DART Configure on HPC
Configuration of DART on HPC

These are some installation notes taken in the process of installing DART. This tutorial is for **free**

## Module load and save as intel module
```console
cd $HOME
ml load oneapi/tbb/2022.2 oneapi/compiler-rt/2025.2.1 oneapi/umf/0.11.0 oneapi/compiler-intel-llvm/latest oneapi/mpi/2021.16
ml save intel
```

```console
cd WPS-4.6.0
export NETCDF=$CONDA_PREFIX
export WRF_DIR=../WRFV4.7.1
./configure --build-grib2-libs
```

Edit configure.wps 
```console
                        -L$(NETCDF)/lib -lnetcdff -lnetcdf

DM_FC               = mpif90
DM_CC               = mpicc
```

with 
```console
                        -L$(NETCDF)/lib -lnetcdff -lnetcdf -lhdf5_hl -lhdf5

DM_FC               = mpif90 -f90=$(SFC)
DM_CC               = mpicc -cc=$(SCC)
```

Compile
```console
./compile &> log &
tail -f log
```

If not successfull with ungrib.exe file
```console
cd external/jasper-1.900.29
make distclean 2>/dev/null || true
rm -f config.status config.cache
autoreconf -fiv
./configure --prefix=/home/danang-eko/misc/WPS-4.6.0/grib2
make
make install
cd ../..
```

Re-configure
```console
cp -r grib2 grib2lib
./clean
./configure
cp -r grib2lib grib2
```


Edit configure.wps 
```console
                        -L$(NETCDF)/lib -lnetcdff -lnetcdf

COMPRESSION_LIBS    = -L/home/danang-eko/misc/WPS-4.6.0/grib2/lib -ljasper -lpng -lz
COMPRESSION_INC     = -I/home/danang-eko/misc/WPS-4.6.0/grib2/include 

DM_FC               = mpif90 
DM_CC               = mpicc 
```

with 
```console
                        -L$(NETCDF)/lib -lnetcdff -lnetcdf -lhdf5_hl -lhdf5
COMPRESSION_LIBS    = -L/home/danang-eko/misc/WPS-4.6.0/grib2/lib -ljasper -lpng -lz
COMPRESSION_INC     = -I/home/danang-eko/misc/WPS-4.6.0/grib2/include -I/home/danang-eko/misc/WPS-4.6.0/grib2/inclu
de/jasper

DM_FC               = mpif90 -f90=$(SFC)
DM_CC               = mpicc -cc=$(SCC)
```


Re-compile
```console
./compile &> log &
tail -f log
```

