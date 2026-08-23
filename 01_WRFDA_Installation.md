# WRFDA Configure on HPC
Configuration of WRFDA on HPC

These are some installation notes taken in the process of installing WRFDA. This tutorial is for **free**

## Module load and save as intel module
```console
cd $HOME
ml load oneapi/tbb/2022.2 oneapi/compiler-rt/2025.2.1 oneapi/umf/0.11.0 oneapi/compiler-intel-llvm/latest oneapi/mpi/2021.16
ml save intel
```
## Building WRF
First of all, we need to download the source code from
```console
cd $HOME/misc
wget https://github.com/wrf-model/WRF/archive/refs/tags/v4.7.1.tar.gz
tar -xvzf v4.7.1.tar.gz
cd WRFV4.7.1
```

Now we are able to run the configure 

```console
$ ./configure
```

You will see various options. Choose the option that lists the compiler you are using and the way you wish to build WRF (i.e., seriall or in parallel). Although there are 3 different types of parallel (smpar, dmpar, and dm+sm), it is recommend choosing dmpar option.

```console
Please select from among the following Linux x86_64 options:

  1. (serial)   2. (smpar)   3. (dmpar)   4. (dm+sm)   PGI (pgf90/gcc)
  5. (serial)   6. (smpar)   7. (dmpar)   8. (dm+sm)   PGI (pgf90/pgcc): SGI MPT
  9. (serial)  10. (smpar)  11. (dmpar)  12. (dm+sm)   PGI (pgf90/gcc): PGI accelerator
 13. (serial)  14. (smpar)  15. (dmpar)  16. (dm+sm)   INTEL (ifort/icc)
                                         17. (dm+sm)   INTEL (ifort/icc): Xeon Phi (MIC architecture)
 18. (serial)  19. (smpar)  20. (dmpar)  21. (dm+sm)   INTEL (ifort/icc): Xeon (SNB with AVX mods)
 22. (serial)  23. (smpar)  24. (dmpar)  25. (dm+sm)   INTEL (ifort/icc): SGI MPT
 26. (serial)  27. (smpar)  28. (dmpar)  29. (dm+sm)   INTEL (ifort/icc): IBM POE
 30. (serial)               31. (dmpar)                PATHSCALE (pathf90/pathcc)
 32. (serial)  33. (smpar)  34. (dmpar)  35. (dm+sm)   GNU (gfortran/gcc)
 36. (serial)  37. (smpar)  38. (dmpar)  39. (dm+sm)   IBM (xlf90_r/cc_r)
 40. (serial)  41. (smpar)  42. (dmpar)  43. (dm+sm)   PGI (ftn/gcc): Cray XC CLE
 44. (serial)  45. (smpar)  46. (dmpar)  47. (dm+sm)   CRAY CCE (ftn $(NOOMP)/cc): Cray XE and XC
 48. (serial)  49. (smpar)  50. (dmpar)  51. (dm+sm)   INTEL (ftn/icc): Cray XC
 52. (serial)  53. (smpar)  54. (dmpar)  55. (dm+sm)   PGI (pgf90/pgcc)
 56. (serial)  57. (smpar)  58. (dmpar)  59. (dm+sm)   PGI (pgf90/gcc): -f90=pgf90
 60. (serial)  61. (smpar)  62. (dmpar)  63. (dm+sm)   PGI (pgf90/pgcc): -f90=pgf90
 64. (serial)  65. (smpar)  66. (dmpar)  67. (dm+sm)   INTEL (ifort/icc): HSW/BDW
 68. (serial)  69. (smpar)  70. (dmpar)  71. (dm+sm)   INTEL (ifort/icc): KNL MIC
 72. (serial)  73. (smpar)  74. (dmpar)  75. (dm+sm)   AMD (flang/clang) :  AMD ZEN1/ ZEN2/ ZEN3 Architectures
 76. (serial)  77. (smpar)  78. (dmpar)  79. (dm+sm)   INTEL (ifx/icx) : oneAPI LLVM
 80. (serial)  81. (smpar)  82. (dmpar)  83. (dm+sm)   FUJITSU (frtpx/fccpx): FX10/FX100 SPARC64 IXfx/Xlfx

Enter selection [1-83] : 78
------------------------------------------------------------------------
Compile for nesting? (1=basic, 2=preset moves, 3=vortex following) [default 1]: 1
Configuration successful! 
------------------------------------------------------------------------
testing for fseeko and fseeko64
fseeko64 is supported
------------------------------------------------------------------------
Settings listed above are written to configure.wrf.
If you wish to change settings, please edit that file.
If you wish to change the default options, edit the file:
     arch/configure.defaults
NetCDF users note:
 This installation of NetCDF supports large file support.  To DISABLE large file
 support in NetCDF, set the environment variable WRFIO_NCD_NO_LARGE_FILE_SUPPORT
 to 1 and run configure again. Set to any other value to avoid this message.
  

Testing for NetCDF, C and Fortran compiler

This installation of NetCDF is 64-bit
                 C compiler is 64-bit
           Fortran compiler is 64-bit
              It will build in 64-bit

*****************************************************************************
This build of WRF will use classic (non-compressed) NETCDF format
*****************************************************************************
```

Edit configure.wps 
```console
 LIB_EXTERNAL    = \
                      -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf -L/home/danang-eko/.conda/envs/dart/lib -lnetcdff -lnetcdf
```

with 
```console
 LIB_EXTERNAL    = \
                      -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf -L/home/danang-eko/.conda/envs/dart/lib -lnetcdff -lnetcdf -lhdf5_hl -lhdf5
```

For this purpose we are going to compile WRF. Compilation should take about 20-30 minutes. The ongoing compilation can be checked.
```console
./compile em_real &> log &
tail -f log
```

If we see this message, you done it right ;)

```console
==========================================================================
build started:   lun nov 18 21:48:48 -03 2019
build completed: lun nov 18 21:56:41 -03 2019
 
--->                  Executables successfully built                  <---
 
==========================================================================
```

Once the compilation completes, to check whether it was successful, you need to look for executables in the `WRFV4.7.1/main` directory.
```console
ls -las main/*.exe
real.exe 
wrf.exe 
ideal.exe 
tc.exe 
```

These executables are linked to 2 different directories. You can choose to run WRF from either directory.
```console
WRFV4.7.1/run
WRFV4.7.1/test/em_real
```

## Building WPS
Now we need to download and compile WPS
```console
cd $HOME/misc
wget https://github.com/wrf-model/WPS/archive/refs/tags/v4.6.0.tar.gz
tar -xvzf v4.6.0.tar.gz
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

## Building WRFDA
After that, we need to copy the source code from
```console
cd $HOME/misc
cp -r WRFV4.7.1 WRFDA
cd WRFDA
./clean
./compile all_wrfvar &> log &
tail -f log
```
