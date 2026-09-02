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
If the following error occurs:
```console
Creating "obs_seq.hf" file.
forrtl: severe (41): insufficient virtual memory
Image              PC                Routine            Line        Source
hf_to_obs          000000000077EE85  Unknown               Unknown  Unknown
hf_to_obs          000000000057168D  Unknown               Unknown  Unknown
hf_to_obs          00000000004BD4F4  Unknown               Unknown  Unknown
hf_to_obs          000000000040A622  Unknown               Unknown  Unknown
libc-2.28.so       000014A36338B865  __libc_start_main     Unknown  Unknown
hf_to_obs          000000000040A52E  Unknown               Unknown  Unknown
```
Reduce the number of files to be read. Execute the following command:
```console
head -1000 obs_files.txt > obs_files.txt
./hf_to_obs
```
The resulting observation sequence file will be written to “obs_seq.hf”
### Satellite SST Data Processing
To begin working on converting our SST data, using Data source: https://data.marine.copernicus.eu/product/SST_GLO_PHY_L3S_MY_010_039.

```console
cd /scratch/inanwp/dart-work
mkdir sst
cd sst
ln -sf $DART/observations/obs_converters/cmems_sst_l3s/work/advance_time  .
ln -sf $DART/observations/obs_converters/cmems_sst_l3s/work/obs_seq_to_netcdf  .
ln -sf $DART/observations/obs_converters/cmems_sst_l3s/work/obs_sequence_tool  .
ln -sf $DART/observations/obs_converters/cmems_sst_l3s/work/cmems_sst_to_obs .
ln -sf $DART/observations/obs_converters/cmems_sst_l3s/work/preprocess .
cp $DART/observations/obs_converters/cmems_sst_l3s/work/input.nml .
ls -d /scratch/inanwp/test3/dart-work/sst/cmems_obs-sst_glo_phy_my_l3s_P1D-m_1780584695815.nc > obs_files.txt
./cmems_sst_to_obs
```
The resulting observation sequence file will be written to “obs_seq.sst”
### Satellite SSH Data Processing
To begin working on converting our SSH data, using Data source: https://data.marine.copernicus.eu/product/SEALEVEL_GLO_PHY_L3_NRT_008_044.

```console
cd /scratch/inanwp/dart-work
mkdir ssh
cd ssh
ln -sf $DART/observations/obs_converters/cmems_ssh_l3/work/advance_time  .
ln -sf $DART/observations/obs_converters/cmems_ssh_l3/work/obs_seq_to_netcdf  .
ln -sf $DART/observations/obs_converters/cmems_ssh_l3/work/obs_sequence_tool  .
ln -sf $DART/observations/obs_converters/cmems_ssh_l3/work/cmems_ssh_to_obs .
ln -sf $DART/observations/obs_converters/cmems_ssh_l3/work/preprocess .
cp $DART/observations/obs_converters/cmems_ssh_l3/work/input.nml .
ls -d /scratch/inanwp/test3/dart-work/ssh/cmems_obs-sl_glo_phy-ssh_nrt_c2n-l3-duacs_PT1S_1780585148858.csv > obs_files.txt
./cmems_ssh_to_obs
```
The resulting observation sequence file will be written to “obs_seq.ssh”.
Adjusting the SSH Data and Errors
```console
cd /scratch/inanwp/dart-work/roms_rutgers
cp /home/inanwp/misc/DART/models/ROMS_rutgers/preprocess_ocean_obs.py .
obs_seq_all=/scratch/inanwp/dart-work/ssh/obs_seq.ssh
obs_seq_trim=/scratch/inanwp/dart-work/ssh/obs_seq.ssh_trim
roms_file=/scratch/inanwp/dart-work/roms_rutgers/roms.nc
python preprocess_ocean_obs.py $obs_seq_all $obs_seq_trim --roms-file $roms_file --obs-type SATELLITE_SSH
```

### Merge all obs_seq files
Now that we’ve created all individual observation sequence files, and in preparation for a data assimilation run in DART, we want to merge all obs_seq files (i.e., stitch them together). For that, we will use a DART utility called “obs_sequence_tool”. Edit the input.nml:
```console
&obs_sequence_tool_nml
   filename_seq      = 'obs_seq.ssh', '../sst/obs_seq.sst', 
                       '../hf/obs_seq.hf', '../arvor/obs_seq.arvor', 
                       '../svp/obs_seq.svp'
   filename_out      = '../roms_rugers/obs_seq_all.out'
   print_only        = .false.
   gregorian_cal     = .true.
   first_obs_days    = -1
   first_obs_seconds = -1
   last_obs_days     = -1
   last_obs_seconds  = -1
   /
```
Run the program:
```console
./obs_sequence_tool
```
Inspect the output messages 
```console
   Processing sequence file ../roms_rutgers/obs_seq_all.out
   Data Metadata: observation
     QC Metadata: QC
  First obs time: day=155141, sec=0
   Gregorian day: 2025 Oct 06 00:00:00
   Last obs time: day=155468, sec=82800
   Gregorian day: 2026 Aug 29 23:00:00
   Number of obs processed  :                601439
   ---------------------------------------------------------
                    FLOAT_SALINITY     212 obs
                 FLOAT_TEMPERATURE     212 obs
       DRIFTER_U_CURRENT_COMPONENT     120 obs
       DRIFTER_V_CURRENT_COMPONENT     120 obs
               DRIFTER_TEMPERATURE     120 obs
             SATELLITE_BLENDED_SST   36638 obs
                     SATELLITE_SSH    1449 obs
       HFRADAR_U_CURRENT_COMPONENT  281284 obs
       HFRADAR_V_CURRENT_COMPONENT  281284 obs

  write_obs_seq  opening formatted observation sequence file "../roms_rutgers/ob
 s_seq_all.out"
  write_obs_seq  closed observation sequence file "../roms_rutgers/obs_seq_all.o
 ut"
  obs_sequence_tool Finished successfully.
```
