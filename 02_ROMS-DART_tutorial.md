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
ln -sf $DART/observations/obs_converters/SVP/work/svp_ssh_to_obs .
ls -d /scratch/m2p3_inacawo_ens/inaCAWO_DA_inputs/SVP/20260829/* > obs_files.txt
```

