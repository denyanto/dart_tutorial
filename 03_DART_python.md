# DART Python
This tutorial demonstrates how to display processed DART data.
## Install DART python environment
Create an environment.yml file and copy the following modules that will be used in Python: 
```console
name: dart
channels:
  - conda-forge
dependencies:
  - jupyterlab
  - netcdf4
  - cmocean
  - scipy
  - python=3.11
  - dask
  - matplotlib
  - pyyaml
  - numpy
  - h5netcdf
  - xarray
  - cartopy
  - tqdm
  - ipykernel
  - jupyter
  - pip
  - pip:
      - pydartdiags
```
Then run the conda command, assuming conda is already installed.
```console
conda env create -f environment.yml
```
## Visualize SVP observations on a map
```console
import pydartdiags.obs_sequence.obs_sequence as obsq
from pathlib import Path

# Path to DART repo (directory) 
basedir = Path(f"/scratch/inanwp/dart-work")

# Path to the SVP converter
svp_dir = basedir / 'svp' 

# Path to the obs_seq file
obs_seq_file = svp_dir / 'obs_seq.svp'
print(f"obs_seq file: {obs_seq_file}")

# Make sure the obs_Seq file exists
assert obs_seq_file.exists(), 'obs_seq file not found'

# Read the obs seq file into a DataFrame
obs = obsq.ObsSequence(obs_seq_file)

# Uncomment to inspect available methods/attributes
help(obs)
```
