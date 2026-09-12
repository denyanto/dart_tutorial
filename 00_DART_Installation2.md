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
cd $HOME/misc
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
cd $HOME/misc
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
cd $HOME/misc
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
ln -sf $INSTALL_DIR/lib64/libnetcdf.* $INSTALL_DIR/lib
cd ../..
```

## Build and Install netcdf-fortran
```console
cd $HOME/misc
wget https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v$NC_F_VERSION.tar.gz
tar xzf v$NC_F_VERSION.tar.gz
cd netcdf-fortran-$NC_F_VERSION
mkdir build && cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_AR=$AR \
  -DCMAKE_RANLIB=$RANLIB \
  -DBUILD_SHARED_LIBS=OFF \
  -DBUILD_STATIC_LIBS=ON \
  -DNETCDF_ROOT=$INSTALL_DIR \
  -DNETCDF_ENABLE_TESTS=OFF \
  -DCMAKE_C_COMPILER=mpiicc \
  -DCMAKE_Fortran_COMPILER=mpiifort
cmake --build . -j$J
cmake --install .
ln -sf $INSTALL_DIR/lib64/libnetcdff.* $INSTALL_DIR/lib
cd ../..
```


## Verify the Netcdf Installation
```console
export PATH=/home/inanwp/.libs/bin:$PATH
# Check the C configuration tool
nc-config --cc

# Check the Fortran configuration tool
nf-config --fc
which nc-config
```

## Downloading DART
```console
cd $HOME/misc
git clone https://github.com/NCAR/DART.git
cd DART
cp build_templates/mkmf.template.intel.linux build_templates/mkmf.template
```

Edit in the build_templates/mkmf.template file
```console
MPIFC=mpiifort
MPILD=mpiifort
FC=ifort
LD=ifort
NETCDF=$INSTALL_DIR
INCS=-I$NETCDF/include
LIBS="-L$NETCDF/lib -lnetcdf -lnetcdff -lhdf5_hl -lhdf5"
FFLAGS="-O2 $INCS"
LDFLAGS="$FFLAGS $LIBS"
```

Build and test DART 
```console
cd $HOME/misc/DART/models/lorenz_63/work
./quickbuild.sh
```

## Build and Install ROMS-DART
### Ocean Observation I - Drifters – Surface Velocity Program (SVP)
```console
cd $HOME/misc/DART/observations/obs_converters/SVP/work
./quickbuild.sh
```
This would generate 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
svp_to_obs
```
### Ocean Observation I - Floats – in-situ profiles of T and S (ARVOR)
```console
cd $HOME/misc/DART/observations/obs_converters/ARVOR/work
./quickbuild.sh
```
This would generate 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
arvor_to_obs
```
### Ocean Observation I - High Frequency radar (HF Radar)
```console
cd $HOME/misc/DART/observations/obs_converters/HFradar/work
./quickbuild.sh
```
This would generate 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
hf_to_obs
```
### Ocean Observation II - Copernicus Marine Environment Monitoring Service (CMEMS) Level-3S blended SST (Satellite SST)
```console
cd $HOME/misc/DART/observations/obs_converters/cmems_sst_l3s/work
./quickbuild.sh
```
This would generate 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
cmems_sst_to_obs
```
### Ocean Observation II - CMEMS Level-3 Near-Real-Time along-track SSH (Satellite SSH)
```console
cd $HOME/misc/DART/observations/obs_converters/cmems_ssh_l3/work
./quickbuild.sh
```
This would generate 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
cmems_ssh_to_obs
```
### ROMS-Rutgers
```console
cd $HOME/misc/DART/models/ROMS_rutgers/work
./quickbuild.sh
```
This would generate several programs:
```console
advance_time              fill_inflation_restart  obs_diag           perfect_model_obs
closest_member_tool       filter                  obs_selection      perturb_single_instance
create_fixed_network_seq  obs_seq_coverage        obs_sequence_tool  wakeup_filter
create_obs_sequence       obs_seq_to_netcdf       obs_common_subset  obs_seq_verify
model_mod_check             
```

## Build and Install WRF-DART
### Observation I - BUFR Format
```console
cd $HOME/misc/DART/observations/obs_converters/NCEP/prep_bufr
./install.sh
```
Confirm the ``exe`` directory contains the executables:
```console
prepbufr.x
prepbufr_03Z.x
grabbufr.x
cword.x
```
Go to the ``$DART_DIR/observations/obs_converters/NCEP/prep_bufr/work/``
   directory and run ``quickbuild.sh`` to build the ``advance_time`` executable:
```console
cd $HOME/misc/DART/observations/obs_converters/NCEP/prep_bufr/work
./quickbuild.sh
```
Confirm the executable file:
```console
advance_time
```
### Observation II - ASCII Format
```console
cd $HOME/misc/DART/observations/obs_converters/NCEP/ascii_to_obs/work
./quickbuild.sh
```
Confirm the executable files:
```console
create_real_obs
prepbufr_to_obs
```
### Observation III - NETCDF Format
```console
cd $HOME/misc/DART/observations/obs_converters/NCEP/netcdf/work
./quickbuild.sh
```
Confirm the executable file:
```console
convert_pb_netcdf
```
### Observation IV - HIMAWARI-9 AHI
Download and Install RTTOV ver 13.2 from https://nwp-saf.eumetsat.int
```console
mkdir rttov13
cd rttov13
tar xf rttov132.tar.xz
```
Edit build/Makefile.local
```console
HDF5_PREFIX = $HOME/.libs
NETCDF_PREFIX = $HOME/.libs

FFLAGS_HDF5 = -I$(HDF5)/include
LDFLAGS_HDF5 = -L$(HDF5)/lib -lhdf5_fortran -lhdf5

FFLAGS_NETCDF = -I$(NETCDF)/include
LDFLAGS_NETCDF = -L$(NETCDF)/lib -lnetcdff -lnetcdf

```
```console
cd $HOME/misc/DART/build_template
cp mkmf.template.rttov.ifort mkmf.template
```
Edit mkmf.template:
```console
MPIFC = mpif90
MPILD = mpif90
FC = ifort
LD = ifort
NETCDF = $(INSTALL_DIR)
HDF5 = $(NETCDF)
RTTOV = $(DART)/../rttov13/
```
Adding library path:
```console
export RTTOV_ROOT=$HOME/misc/rttov13
export LD_LIBRARY_PATH=$RTTOV_ROOT/lib:$LD_LIBRARY_PATH
```
Edit input.nml DART
```console&preprocess_nml
&preprocess_nml
   input_files = '../../../observations/forward_operators/obs_def_rttov_mod.f90',
quantity_files = '../../../assimilation_code/modules/observations/atmosphere_quantities_mod.f90',
                  '../../../assimilation_code/modules/observations/ocean_quantities_mod.f90',
                  '../../../assimilation_code/modules/observations/chemistry_quantities_mod.f90',
                  '../../../assimilation_code/modules/observations/land_quantities_mod.f90'
   /
&obs_kind_nml
   assimilate_these_obs_types = 'HIMAWARI_9_AHI_RADIANCE',
                                 'RADIOSONDE_TEMPERATURE',
                                 'RADIOSONDE_U_WIND_COMPONENT',
                                 'RADIOSONDE_V_WIND_COMPONENT',
   /
&obs_def_rttov_nml
   use_tskin = .true.
   addsolar = .false.
   cfrac_data = .false.
   clw_data = .true.
   ciw_data = .true.
   rain_data = .false.
   snow_data = .false.
   graupel_data = .false.
/
&model_nml

   default_state_variables = .false.

   wrf_state_variables =

      'U',
      'QTY_U_WIND_COMPONENT',
      'TYPE_U',
      'UPDATE',
      '999',

      'V',
      'QTY_V_WIND_COMPONENT',
      'TYPE_V',
      'UPDATE',
      '999',

      'W',
      'QTY_VERTICAL_VELOCITY',
      'TYPE_W',
      'UPDATE',
      '999',

      'THM',
      'QTY_POTENTIAL_TEMPERATURE',
      'TYPE_T',
      'UPDATE',
      '999',

      'PH',
      'QTY_GEOPOTENTIAL_HEIGHT',
      'TYPE_GZ',
      'UPDATE',
      '999',

      'MU',
      'QTY_PRESSURE',
      'TYPE_MU',
      'UPDATE',
      '999',

      'QVAPOR',
      'QTY_VAPOR_MIXING_RATIO',
      'TYPE_QV',
      'UPDATE',
      '999',

      'T2',
      'QTY_2M_TEMPERATURE',
      'TYPE_T2',
      'UPDATE',
      '999',

      'PSFC',
      'QTY_SURFACE_PRESSURE',
      'TYPE_PS',
      'UPDATE',
      '999',

      'TSK',
      'QTY_SKIN_TEMPERATURE',
      'TYPE_TSK',
      'UPDATE',
      '999',

      'HGT',
      'QTY_SURFACE_ELEVATION',
      'TYPE_HGT',
      'UPDATE',
      '999',
/

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
