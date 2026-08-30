# DART Configure on HPC TUNA-MMS
Configuration of DART on HPC

These are some installation notes taken in the process of installing DART on HPC TUNA-MMS using ROME. This tutorial is for **free**


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
