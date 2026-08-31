# ROMS-DART Tutorial
## Check the Observation Converter
### Check in the SVP converter folder:
```console
ls $DART_DIR/observations/obs_converters/SVP/work
```
It should contain 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
svp_to_obs
```
### Check in the ARVOR converter folder:
```console
ls $DART_DIR/observations/obs_converters/ARVOR/work
```
It should contain 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
arvor_to_obs
```
### Check in the HF Radar converter folder:
```console
ls $DART_DIR/observations/obs_converters/HFradar/work
```
It should contain 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
hf_to_obs
```
### Check in the Satelite SST converter folder:
```console
ls $DART_DIR/observations/obs_converters/cmems_sst_l3s/work
```
It should contain 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
cmems_sst_to_obs
```
### Check in the Satelite SSH converter folder:
```console
ls $DART_DIR/observations/obs_converters/cmems_ssh_l3/work
```
It should contain 4 programs:
```console
advance_time
obs_seq_to_netcdf
obs_sequence_tool
cmems_ssh_to_obs
```

## Preparing the experiment directory and converting the observation data
```console
cd /scratch/inanwp
mkdir dart-work
```
### SVP Data Processing
```console
cd /scratch/inanwp/dart-work
mkdir svp
cd svp
ln -sf $DART/observations/obs_converters/SVP/work/advance_time  .
ln -sf $DART/observations/obs_converters/SVP/work/obs_seq_to_netcdf  .
ln -sf $DART/observations/obs_converters/SVP/work/obs_sequence_tool  .
ln -sf $DART/observations/obs_converters/SVP/work/svp_to_obs .
ln -sf $DART/observations/obs_converters/SVP/work/preprocess .
cp $DART/observations/obs_converters/SVP/work/input.nml .
ls -d /scratch/m2p3_inacawo_ens/inaCAWO_DA_inputs/SVP/20260829/* > obs_files.txt
./svp_to_obs
```
The resulting observation sequence file will be written to “obs_seq.svp”
### ARVOR Data Processing
```console
cd /scratch/inanwp/dart-work
mkdir arvor
cd arvor
ln -sf $DART/observations/obs_converters/ARVOR/work/advance_time  .
ln -sf $DART/observations/obs_converters/ARVOR/work/obs_seq_to_netcdf  .
ln -sf $DART/observations/obs_converters/ARVOR/work/obs_sequence_tool  .
ln -sf $DART/observations/obs_converters/ARVOR/work/arvor_to_obs .
ln -sf $DART/observations/obs_converters/ARVOR/work/preprocess .
cp $DART/observations/obs_converters/ARVOR/work/input.nml .
ls -d /scratch/m2p3_inacawo_ens/inaCAWO_DA_inputs/ARVOR-I/20260829/* > obs_files.txt
./arvor_to_obs
```
The resulting observation sequence file will be written to “obs_seq.arvor”
### HF Radar Data Processing
```console
export LD_LIBRARY_PATH=/usr/lib64:$LD_LIBRARY_PATH
cd /scratch/inanwp/dart-work
mkdir hf
cd hf
ln -sf $DART/observations/obs_converters/HFradar/work/advance_time  .
ln -sf $DART/observations/obs_converters/HFradar/work/obs_seq_to_netcdf  .
ln -sf $DART/observations/obs_converters/HFradar/work/obs_sequence_tool  .
ln -sf $DART/observations/obs_converters/HFradar/work/hf_to_obs .
ln -sf $DART/observations/obs_converters/HFradar/work/preprocess .
cp $DART/observations/obs_converters/HFradar/work/input.nml .
ls -d /scratch/m2p3_inacawo_ens/inaCAWO_DA_inputs/HF_RADAR/CODAR/20260829/* > obs_files.txt
./hf_to_obs
```
The resulting observation sequence file will be written to “obs_seq.hf”


