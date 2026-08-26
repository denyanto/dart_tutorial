# WRF-DART Tutorial
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
## Locate the DART, WRF, WPS and WRFDA executables path
Edit/configure '$BASE_DIR/scripts/param.sh' file with proper paths and variables
```console
# -----------------------------------------------------------
# Directory structure
# IMPORTANT: scripts rely on these relative names
# -----------------------------------------------------------
BASE_DIR=/scratch/
RUN_DIR=${BASE_DIR}/rundir
TEMPLATE_DIR=${BASE_DIR}/template
OBSPROC_DIR=${BASE_DIR}/obsproc
OUTPUT_DIR=${BASE_DIR}/output
ICBC_DIR=${BASE_DIR}/icbc
POST_STAGE_DIR=${BASE_DIR}/post
OBS_DIAG_DIR=${BASE_DIR}/obs_diag
PERTS_DIR=${BASE_DIR}/perts
# -----------------------------------------------------------
# Component paths
# -----------------------------------------------------------
SHELL_SCRIPTS_DIR=${BASE_DIR}/scripts
DART_DIR=/glade/work/bmraczka/DART
WRF_DM_SRC_DIR=/glade/work/bmraczka/WRF/WRFv4.5_git
WPS_SRC_DIR=/glade/work/bmraczka/WRF/WPSv4.5_git
VAR_SRC_DIR=/glade/work/bmraczka/WRF/WRFDAv4.5_git
```
Run the setup.sh script to create the proper directory structure and to move the executables and support files to the proper locations.
```console
cd $BASE_DIR/scripts
./setup.sh param.sh
```
Search for the string 'set this appropriately' for locations that you need to edit.
```console
cd $BASE_DIR/scripts
grep -r 'set this appropriately' .
```
Other than 'param.sh', which was covered above, make the following changes:
```console
# driver.sh
datefnl = 2024051912                # Change to the final assimilation target date. In this example observations are assimilated at time steps 2024051906 and 2024051912.
# gen_retro_icbc.sh
datea = 2024051900                  # Set to the starting time. This is the beginning time of the ensemble spinup, which runs for 6 hours until the first assimilation time step at 2024051906.
datefnl = 2024052000                # Set to the final time. This is the end of the forecast mode.
paramfile = /full/path/to/param.sh  # Script sources information from param.sh file.
# gen_pert_bank.sh
datea = 2024051900                  # Set to the starting time.
num_ens = 60 (automatically set)    # Total number of perturbation members. Automatically set to 3x model ensemble members (20).
paramfile = /full/path/to/param.sh  # Script sources information from param.sh file.
savedir = ${PERTS_DIR}/work/boundary_perts.  # Location of perturbation bank.
# add_bank_pert.ncl
bank_size = 60 (automatically set)  # Automatically set to 3x model ensemble members (20). If set manually it is recommended to set to the same value as gen_pert_bank.sh num_ens value. Cannot be greater than total perturbations in bank.
```
## Create Initial and Boundary Conditions
```console
cd $BASE_DIR/scripts
./gen_retro_icbc.sh
```

