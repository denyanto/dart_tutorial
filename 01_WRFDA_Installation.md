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
make distclean 2>/dev/null || true
rm -f config.status config.cache
autoreconf -fiv
./configure --prefix=/home/danang-eko/misc/WPS-4.6.0/grib2

```
