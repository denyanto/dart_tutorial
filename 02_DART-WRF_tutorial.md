# DART-WRF Tutorial
This tutorial based on the https://docs.dart.ucar.edu/en/latest/models/wrf/tutorial/README.html.
## Build the WRF-DART executables
```console
cd $DART_DIR/models/wrf/work
./quickbuild.sh
```
Many executables are built, the following executables are needed for the tutorial and will be copied to the right place by the setup.sh script in a subsequent step:
```console
advance_time
fill_inflation_restart
filter
obs_diag
obs_seq_to_netcdf
obs_sequence_tool
pert_wrf_bc
wrf_dart_obs_preprocess
```
## Preparing the experiment directory.
Create a “work” directory that can accomodate approximately 40 GB of space to run the tutorial. The rest of the instructions assume you have an environment variable called BASE_DIR that points to this directory.
```console
cd $BASE_DIR
wget data.dart.ucar.edu/WRF/wrf_dart_nested_tutorial_15Jan2026.tar.gz
tar -xzvf wrf_dart_nested_tutorial_15Jan2026.tar.gz

cp $DART_DIR/models/wrf/tutorial/template_nest/*.*   $BASE_DIR/template/.
mkdir $BASE_DIR/scripts
cp -R $DART_DIR/models/wrf/shell_scripts/* $BASE_DIR/scripts
```
## Locate the WRF, WPS and WRFDA executables
Edit/configure $BASE_DIR/scripts/param.sh with proper paths and variables
```console
cd $DART_DIR/models/wrf/work
./quickbuild.sh
```
