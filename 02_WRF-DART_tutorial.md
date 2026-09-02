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
datefnl = 2024051912                          # Change to the final assimilation target date. In this example observations are assimilated at time steps 2024051906 and 2024051912.
# gen_retro_icbc.sh
datea = 2024051900                            # Set to the starting time. This is the beginning time of the ensemble spinup, which runs for 6 hours until the first assimilation time step at 2024051906.
datefnl = 2024052000                          # Set to the final time. This is the end of the forecast mode.
paramfile = /full/path/to/param.sh            # Script sources information from param.sh file.
# gen_pert_bank.sh
datea = 2024051900                            # Set to the starting time.
num_ens = 60 (automatically set)              # Total number of perturbation members. Automatically set to 3x model ensemble members (20).
paramfile = /full/path/to/param.sh            # Script sources information from param.sh file.
savedir = ${PERTS_DIR}/work/boundary_perts.   # Location of perturbation bank.
# add_bank_pert.ncl
bank_size = 60 (automatically set)            # Automatically set to 3x model ensemble members (20). If set manually it is recommended to set to the same value as gen_pert_bank.sh num_ens value. Cannot be greater than total perturbations in bank.
```
## Create Initial and Boundary Conditions
The namelist settings included within the 'namelist.input.meso' template. 
```console
cd $BASE_DIR/scripts
./gen_retro_icbc.sh
```
Once the script completes, you should confirm the following files have been created within the '$BASE_DIR/output/2024051900' directory:

```console
wrfbdy_d01_154636_21600_mean
wrfinput_d01_154636_0_mean
wrfinput_d01_154636_21600_mean
wrfinput_d02_154636_0_mean
wrfinput_d02_154636_21600_mean
```
## Generate Perturbation
The spatial pattern and magnitude of the perturbations are controlled through the '&wrfvar7' 'cv_options', 'as1', 'as2', 'as3' and 'as4' namelist settings included within the 'namelist.input.3dvar' template.
```console
cd $BASE_DIR/scripts
./gen_pert_bank.sh
```
The script will generate a batch job for each perturbation (60 total). The rule of thumb is to generate 3-4X as many perturbations as the model ensemble (20). This is done to increase the probability each ensemble member receives a unique perturbation. You should confirm the following files have been created within the '$PERTS_DIR/work/boundary_perts' directory:
```console
pert_bank_mem_01.nc
pert_bank_mem_02.nc
..
..
pert_bank_mem_60.nc
```
## Perform Ensemble Spinup
Next, we generate an initial ensemble of WRF states to prepare for the first assimilation (analysis) step. We run the script init_ensemble_var.sh, which takes two arguments: a date string for the starting time and the path to the param.sh script.
```console
cd $BASE_DIR/scripts
./init_ensemble_var.sh 2024051900 param.sh
```
When the scripts complete for the all ensemble members, you should find 20 new files for each domain (40 total files) in the directory '$BASE_DIR/output/2024051900/PRIORS' named prior_d01.0001, prior_d02.0001, etc.
## Observation Converter
### VAWS Data Processing
```console
cd /scratch/inanwp/dart-work
mkdir VAWS
cd VAWS
ln -sf $DART/observations/obs_converters/NCEP/prep_bufr/work/advance_time  .
ln -sf $DART/observations/obs_converters/NCEP/prep_bufr/work/prepbufr.csh  .
ln -sf $DART/observations/obs_converters/NCEP/prep_bufr/work/preprocess  .
cp $DART/observations/obs_converters/NCEP/prep_bufr/work/run_some_prepbufr.csh .
cp $DART/observations/obs_converters/NCEP/prep_bufr/work/multi_parallel.batch .
cp $DART/observations/obs_converters/NCEP/prep_bufr/work/input.nml .
ls -d /scratch/m2p3_inacawo_ens/inaCAWO_DA_inputs/VAWS/20260901/* > obs_files.txt
./run_some_prepbufr.csh
```
The resulting observation sequence file will be written to “obs_seq.svp”

## Create the First Set of Inflation Files
The initial adaptive inflation files will be used by DART to control how the ensemble is inflated (increases spread) during the first assimilation cycle. Within the '$BASE_DIR/rundir' directory, the 'input.nml' file has settings that control the behavior of 'fill_inflation_restart'. Within this file there is the section:
```console
&fill_inflation_restart_nml
   write_prior_inf = .true.
   prior_inf_mean  = 1.00
   prior_inf_sd    = 0.6

   write_post_inf  = .false.
   post_inf_mean   = 1.00
   post_inf_sd     = 0.6

   input_state_files = 'wrfinput_d01','wrfinput_d02'
   single_file       = .false.
   verbose           = .false.
   /
```
Run the following commands:
```console
datea=2024051900
cd $BASE_DIR/rundir
cp ../output/$datea/wrfinput_d01_154636_0_mean ./wrfinput_d01
cp ../output/$datea/wrfinput_d02_154636_0_mean ./wrfinput_d02
./fill_inflation_restart
mkdir ../output/$datea/Inflation_input
mv input_priorinf_*.nc ../output/$datea/Inflation_input/
```
## Perform the Assimilation
The 'driver.sh' script accomplishes this through a series of scripts that 1) assimilates observations using the DART filter, 2) calculates observation space diagnostics for that assimilation time step, and 3) advances the WRF ensemble members to the next assimilation time step.
```console
cd $BASE_DIR/scripts
./driver.sh 2024051906 param.sh >& run.out &
```
