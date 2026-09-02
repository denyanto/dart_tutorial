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
conda activate dart
```
## Visualize SVP observations on a map
Create a plotting folder to store the visualization scripts. Misalnya svp_ploting.py:
```console
import pydartdiags.obs_sequence.obs_sequence as obsq
import cartopy.crs       as ccrs
import cartopy.feature   as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates  as mdates
from pathlib import Path
from IPython.display      import Markdown, display
import cmocean

# Path to DART repo (directory) 
basedir = Path(f"{$rundart}")

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
# help(obs)

# Examine the file
print(f"DataFrame shape: {obs.df.shape}")
print('\n')

display(obs.df)

# To view everything
# obs_seq.df
# obs_seq.all_obs

print("*" * 16)
print("obs_seq SUMMARY:")
print("*" * 16)

print(f"\nNumber of observations : {len(obs.df)}")
print(f"Number of obs types    : {len(obs.types)}")

# Available observation types in the obs_seq file 
# Each type is associated with a DART idenitifier number
print("\nObservation types:")
for kind, name in obs.types.items():
    print(f"  {kind:3d} : {name}")

# Number of copies in the obs_seq 
# observation, QC, .. could be more especially after assimilation 
print("\nObservation copies:")
for i, name in enumerate(obs.copie_names):
    print(f"  {i:2d} : {name}")

# Number of QCs
print("\nQC copies:")
for i, name in enumerate(obs.qc_copie_names):
    print(f"  {i:2d} : {name}")

display(
    obs.df["type"]
    .value_counts()
    .rename_axis("Observation Type")
    .to_frame("Count")
)

df = obs.df.copy()

# Define obs info as a list of tuples to use for plotting
plot_specs = [
    ("DRIFTER_TEMPERATURE"        , "Temperature", "C"  , cmocean.cm.thermal),
    ("DRIFTER_U_CURRENT_COMPONENT", "U current"  , "m/s", cmocean.cm.thermal),
    ("DRIFTER_V_CURRENT_COMPONENT", "V current"  , "m/s", cmocean.cm.thermal),
]

# Define the cartopy projection: standard. un-projected Equidistant 
# Cylindrical coordinate system (lons, lats mapped to 2D cartesian grid)
proj = ccrs.PlateCarree()
# fig = plt.figure(figsize=(16,7))
# # We will plot a figure containing 3 different plots (maps)
# axes = fig.add_subplot(1, 3, sharey=True, projection=proj)
fig, axes = plt.subplots(
    3, 1,
    figsize=(10, 7),
    sharey=True,
    subplot_kw={"projection": proj}
)

# Loop over figure specs:
# ax: current axis; obs_type: T, U, V; cmap: colormap style
for ax, (obs_type, title, label, cmap) in zip(axes, plot_specs):

    # subset dataframe by observation type
    # i.e., only get data for each obs type
    this_ob = df[df["type"] == obs_type]

    ax.coastlines(resolution="10m")
    ax.add_feature(cfeature.OCEAN, facecolor="aliceblue")
    ax.add_feature(cfeature.LAKES, facecolor="lightsteelblue")
    ax.add_feature(cfeature.LAND, zorder=0, linewidth=0.5, facecolor="whitesmoke")
    ax.add_feature(cfeature.BORDERS, linewidth=0.5)

    sc = ax.scatter(
        this_ob["longitude"],
        this_ob["latitude"],
        c=this_ob["observation"],
        s=50,
        cmap=cmap,
        transform=proj,
        zorder=5,
    )

    plt.colorbar(sc, ax=ax, label=label, shrink=1)

    ax.set_extent([110, 140, -10, 0], crs=proj)

    # gl = ax.gridlines(draw_labels=True, linewidth=0.5, color="gray", alpha=0.2)
    # gl.top_labels = False
    # gl.right_labels = False

    ax.set_title(f"{title} (N={len(this_ob)})")

# plt.suptitle("SVP Observations by Type", fontsize=18)
plt.tight_layout()

# display(Markdown("""
# The maps below show the locations and values of the different observation types in the `obs_seq` file. 
# """))

plt.show()
plt.savefig('fig/svp1.png', format='png', dpi=90, bbox_inches='tight')

# Keep only drifters in the Indonesia domain
df_svp = df[
    (df["longitude"] >= 95) & (df["longitude"] <= 145) &
    (df["latitude"]  >= -15) & (df["latitude"]  <= 15)
].copy()

# Create a drifter ID from location
this_ob = df_svp[df_svp["type"] == obs_type].copy()

# Use rounded (approximate) location to identify drifters
this_ob["drifter_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

# Sort IDs so numbering is reproducible
ids = sorted(this_ob["drifter_id"].unique())

id_map = {id_: f"SVP Drifter {i+1}" for i, id_ in enumerate(ids)}

fig, axes = plt.subplots(
    nrows=len(plot_specs),
    ncols=1,
    figsize=(16, 8),
    sharex=True,
)

for ax, (obs_type, title, ylabel, cmap) in zip(axes, plot_specs):

    this_ob = df_svp[df_svp["type"] == obs_type].copy()

    this_ob["drifter_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

    ids = sorted(this_ob["drifter_id"].unique())
    id_map = {id_: f"SVP Drifter {i+1}" for i, id_ in enumerate(ids)}

    for drifter_id in ids:

        dr = this_ob[this_ob["drifter_id"] == drifter_id].sort_values("time")

        lon0 = dr["longitude"].iloc[0]
        lat0 = dr["latitude"].iloc[0]

        ax.plot(dr["time"], dr["observation"], "-o", ms=5,
                label=f"{id_map[drifter_id]} ({lon0:.2f}E, {lat0:.2f}N)")

    ax.set_ylabel(ylabel)
    ax.set_title(f"{title} (N={len(this_ob)})", fontsize=14)
    ax.grid(True, alpha=0.1)
    ax.legend(fontsize=8)

axes[-1].set_xlabel("Time") 

# Format time axis 
axes[-1].xaxis.set_major_formatter(mdates.DateFormatter("%H:%M")) 
axes[-1].xaxis.set_major_locator(mdates.HourLocator(interval=1))

plt.tight_layout()

# display(Markdown("""
# At each analysis time, there are observations from <mark>**several (4) drifters**</mark>. 
# There is one other SVP drifter that is already outside our InaCAWO domain; we've ignored it here. 
# """))

plt.show()
plt.savefig('fig/svp2.png', format='png', dpi=90, bbox_inches='tight')
```
<img width="607" height="621" alt="svp1" src="https://github.com/user-attachments/assets/4787f3f2-abe1-4e7d-ba8a-9922b5a825ab" />
<img width="1421" height="711" alt="svp2" src="https://github.com/user-attachments/assets/d8afdcdd-2769-42fd-ad40-f08d53a4d55a" />

## Visualize ARVOR observations on a map
Create a plotting folder to store the visualization scripts. Misalnya arvor_ploting.py:
```console
import pydartdiags.obs_sequence.obs_sequence as obsq
import cartopy.crs       as ccrs
import cartopy.feature   as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates  as mdates
from pathlib import Path
from IPython.display      import Markdown, display
import cmocean

# Path to DART repo (directory) 
basedir = Path(f"{$rundart}")

# Path to the ARVOR converter
svp_dir = basedir / 'arvor' 

# Path to the obs_seq file
obs_seq_file = svp_dir / 'obs_seq.arvor'
print(f"obs_seq file: {obs_seq_file}")

# Make sure the obs_Seq file exists
assert obs_seq_file.exists(), 'obs_seq file not found'

# Read the obs seq file into a DataFrame
obs = obsq.ObsSequence(obs_seq_file)

# Uncomment to inspect available methods/attributes
# help(obs)

# Examine the file
print(f"DataFrame shape: {obs.df.shape}")
print('\n')

display(obs.df)

# To view everything
# obs_seq.df
# obs_seq.all_obs

print("*" * 16)
print("obs_seq SUMMARY:")
print("*" * 16)

print(f"\nNumber of observations : {len(obs.df)}")
print(f"Number of obs types    : {len(obs.types)}")

# Available observation types in the obs_seq file 
# Each type is associated with a DART idenitifier number
print("\nObservation types:")
for kind, name in obs.types.items():
    print(f"  {kind:3d} : {name}")

# Number of copies in the obs_seq 
# observation, QC, .. could be more especially after assimilation 
print("\nObservation copies:")
for i, name in enumerate(obs.copie_names):
    print(f"  {i:2d} : {name}")

# Number of QCs
print("\nQC copies:")
for i, name in enumerate(obs.qc_copie_names):
    print(f"  {i:2d} : {name}")

display(
    obs.df["type"]
    .value_counts()
    .rename_axis("Observation Type")
    .to_frame("Count")
)

df = obs.df.copy()

# Define obs info as a list of tuples to use for plotting
plot_specs = [
    ("FLOAT_TEMPERATURE", "Temperature", "C"  , cmocean.cm.thermal),
    ("FLOAT_SALINITY"   , "Salinity"   , "psu", cmocean.cm.haline),
]

# Define the cartopy projection: standard. un-projected Equidistant 
# Cylindrical coordinate system (lons, lats mapped to 2D cartesian grid)
proj = ccrs.PlateCarree()
# fig = plt.figure(figsize=(16,7))
# # We will plot a figure containing 3 different plots (maps)
# axes = fig.add_subplot(1, 3, sharey=True, projection=proj)
fig, axes = plt.subplots(
    2, 1,
    figsize=(10, 7),
    sharex=True,
    subplot_kw={"projection": proj}
)

# Loop over figure specs:
# ax: current axis; obs_type: T, U, V; cmap: colormap style
for ax, (obs_type, title, label, cmap) in zip(axes, plot_specs):

    # subset dataframe by observation type
    # i.e., only get data for each obs type
    this_ob = df[df["type"] == obs_type]

    ax.coastlines(resolution="10m")
    ax.add_feature(cfeature.OCEAN, facecolor="aliceblue")
    ax.add_feature(cfeature.LAKES, facecolor="lightsteelblue")
    ax.add_feature(cfeature.LAND, zorder=0, linewidth=0.5, facecolor="whitesmoke")
    ax.add_feature(cfeature.BORDERS, linewidth=0.5)

    sc = ax.scatter(
        this_ob["longitude"],
        this_ob["latitude"],
        c=this_ob["observation"],
        s=80,
        cmap=cmap,
        transform=proj,
        zorder=5,
    )

    plt.colorbar(sc, ax=ax, label=label, shrink=1)

    ax.set_extent([110, 140, -10, 0], crs=proj)

    # gl = ax.gridlines(draw_labels=True, linewidth=0.5, color="gray", alpha=0.2)
    # gl.top_labels = False
    # gl.right_labels = False

    ax.set_title(f"{title} (N={len(this_ob)})")

plt.suptitle("ARVOR Observations by Type", fontsize=18)
plt.tight_layout()

# display(Markdown("""
# The maps below show the locations and values of the different observation types in the `obs_seq` file. 
# """))

plt.show()
plt.savefig('fig/arvor1.png', format='png', dpi=90, bbox_inches='tight')

# Keep only drifters in the Indonesia domain
df = df[
    (df["longitude"] >= 95) & (df["longitude"] <= 145) &
    (df["latitude"]  >= -15) & (df["latitude"]  <= 15)
].copy()

# Create a drifter ID from location
this_ob = df[df["type"] == obs_type].copy()

# Use rounded (approximate) location to identify drifters
this_ob["arvor_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

# Sort IDs so numbering is reproducible
ids = sorted(this_ob["arvor_id"].unique())

id_map = {id_: f"ARVOR Float {i+1}" for i, id_ in enumerate(ids)}

fig, axes = plt.subplots(
    nrows=len(plot_specs),
    ncols=1,
    figsize=(16, 8),
    sharex=True,
)

for ax, (obs_type, title, ylabel, cmap) in zip(axes, plot_specs):

    this_ob = df[df["type"] == obs_type].copy()

    this_ob["arvor_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

    ids = sorted(this_ob["arvor_id"].unique())
    id_map = {id_: f"ARVOR Float {i+1}" for i, id_ in enumerate(ids)}

    for arvor_id in ids:

        prof = this_ob[this_ob["arvor_id"] == arvor_id].sort_values("vertical")

        lon0 = prof["longitude"].iloc[0]
        lat0 = prof["latitude"].iloc[0]

        ax.plot(prof["observation"], prof["vertical"], "-o", ms=4, lw=1.8,
                label=f"{id_map[arvor_id]} ({lon0:.2f}°E, {lat0:.2f}°N)")

        ax.invert_yaxis()
        ax.set_ylim(10000, 1)
        ax.set_yscale('log')
        
        ax.set_xlabel(ylabel, fontsize= 14)
        ax.set_ylabel('Depth (m)', fontsize= 14)
        ax.set_title(f"{title} (N={len(this_ob)})", fontsize=16)
        
        ax.grid(True, alpha=0.25)
        ax.legend(fontsize=12)

plt.tight_layout()

# display(Markdown("""
# At each analysis time, there are observations from <mark>**several (4) drifters**</mark>. 
# There is one other SVP drifter that is already outside our InaCAWO domain; we've ignored it here. 
# """))

plt.show()
plt.savefig('fig/arvor2.png', format='png', dpi=90, bbox_inches='tight')
```
<img width="828" height="621" alt="arvor1" src="https://github.com/user-attachments/assets/f484a563-85bc-46ed-ad65-6b27b77c14c2" />
<img width="1431" height="710" alt="arvor2" src="https://github.com/user-attachments/assets/6428c587-7704-4724-834e-883072f5adff" />

## Visualize HF Radar observations on a map
Create a plotting folder to store the visualization scripts. Misalnya hf_ploting.py:
```console
import pydartdiags.obs_sequence.obs_sequence as obsq
import cartopy.crs       as ccrs
import cartopy.feature   as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates  as mdates
from pathlib import Path
from IPython.display      import Markdown, display
import cmocean

# Path to DART repo (directory) 
basedir = Path(f"{$rundart}")

# Path to the ARVOR converter
svp_dir = basedir / 'arvor' 

# Path to the obs_seq file
obs_seq_file = svp_dir / 'obs_seq.arvor'
print(f"obs_seq file: {obs_seq_file}")

# Make sure the obs_Seq file exists
assert obs_seq_file.exists(), 'obs_seq file not found'

# Read the obs seq file into a DataFrame
obs = obsq.ObsSequence(obs_seq_file)

# Uncomment to inspect available methods/attributes
# help(obs)

# Examine the file
print(f"DataFrame shape: {obs.df.shape}")
print('\n')

display(obs.df)

# To view everything
# obs_seq.df
# obs_seq.all_obs

print("*" * 16)
print("obs_seq SUMMARY:")
print("*" * 16)

print(f"\nNumber of observations : {len(obs.df)}")
print(f"Number of obs types    : {len(obs.types)}")

# Available observation types in the obs_seq file 
# Each type is associated with a DART idenitifier number
print("\nObservation types:")
for kind, name in obs.types.items():
    print(f"  {kind:3d} : {name}")

# Number of copies in the obs_seq 
# observation, QC, .. could be more especially after assimilation 
print("\nObservation copies:")
for i, name in enumerate(obs.copie_names):
    print(f"  {i:2d} : {name}")

# Number of QCs
print("\nQC copies:")
for i, name in enumerate(obs.qc_copie_names):
    print(f"  {i:2d} : {name}")

display(
    obs.df["type"]
    .value_counts()
    .rename_axis("Observation Type")
    .to_frame("Count")
)

df = obs.df.copy()

# Define obs info as a list of tuples to use for plotting
plot_specs = [
    ("FLOAT_TEMPERATURE", "Temperature", "C"  , cmocean.cm.thermal),
    ("FLOAT_SALINITY"   , "Salinity"   , "psu", cmocean.cm.haline),
]

# Define the cartopy projection: standard. un-projected Equidistant 
# Cylindrical coordinate system (lons, lats mapped to 2D cartesian grid)
proj = ccrs.PlateCarree()
# fig = plt.figure(figsize=(16,7))
# # We will plot a figure containing 3 different plots (maps)
# axes = fig.add_subplot(1, 3, sharey=True, projection=proj)
fig, axes = plt.subplots(
    2, 1,
    figsize=(10, 7),
    sharex=True,
    subplot_kw={"projection": proj}
)

# Loop over figure specs:
# ax: current axis; obs_type: T, U, V; cmap: colormap style
for ax, (obs_type, title, label, cmap) in zip(axes, plot_specs):

    # subset dataframe by observation type
    # i.e., only get data for each obs type
    this_ob = df[df["type"] == obs_type]

    ax.coastlines(resolution="10m")
    ax.add_feature(cfeature.OCEAN, facecolor="aliceblue")
    ax.add_feature(cfeature.LAKES, facecolor="lightsteelblue")
    ax.add_feature(cfeature.LAND, zorder=0, linewidth=0.5, facecolor="whitesmoke")
    ax.add_feature(cfeature.BORDERS, linewidth=0.5)

    sc = ax.scatter(
        this_ob["longitude"],
        this_ob["latitude"],
        c=this_ob["observation"],
        s=80,
        cmap=cmap,
        transform=proj,
        zorder=5,
    )

    plt.colorbar(sc, ax=ax, label=label, shrink=1)

    ax.set_extent([110, 140, -10, 0], crs=proj)

    # gl = ax.gridlines(draw_labels=True, linewidth=0.5, color="gray", alpha=0.2)
    # gl.top_labels = False
    # gl.right_labels = False

    ax.set_title(f"{title} (N={len(this_ob)})")

plt.suptitle("ARVOR Observations by Type", fontsize=18)
plt.tight_layout()

# display(Markdown("""
# The maps below show the locations and values of the different observation types in the `obs_seq` file. 
# """))

plt.show()
plt.savefig('fig/arvor1.png', format='png', dpi=90, bbox_inches='tight')

# Keep only drifters in the Indonesia domain
df = df[
    (df["longitude"] >= 95) & (df["longitude"] <= 145) &
    (df["latitude"]  >= -15) & (df["latitude"]  <= 15)
].copy()

# Create a drifter ID from location
this_ob = df[df["type"] == obs_type].copy()

# Use rounded (approximate) location to identify drifters
this_ob["arvor_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

# Sort IDs so numbering is reproducible
ids = sorted(this_ob["arvor_id"].unique())

id_map = {id_: f"ARVOR Float {i+1}" for i, id_ in enumerate(ids)}

fig, axes = plt.subplots(
    nrows=len(plot_specs),
    ncols=1,
    figsize=(16, 8),
    sharex=True,
)

for ax, (obs_type, title, ylabel, cmap) in zip(axes, plot_specs):

    this_ob = df[df["type"] == obs_type].copy()

    this_ob["arvor_id"] = (this_ob["longitude"].round(2).astype(str) + "_" + this_ob["latitude"].round(2).astype(str))

    ids = sorted(this_ob["arvor_id"].unique())
    id_map = {id_: f"ARVOR Float {i+1}" for i, id_ in enumerate(ids)}

    for arvor_id in ids:

        prof = this_ob[this_ob["arvor_id"] == arvor_id].sort_values("vertical")

        lon0 = prof["longitude"].iloc[0]
        lat0 = prof["latitude"].iloc[0]

        ax.plot(prof["observation"], prof["vertical"], "-o", ms=4, lw=1.8,
                label=f"{id_map[arvor_id]} ({lon0:.2f}°E, {lat0:.2f}°N)")

        ax.invert_yaxis()
        ax.set_ylim(10000, 1)
        ax.set_yscale('log')
        
        ax.set_xlabel(ylabel, fontsize= 14)
        ax.set_ylabel('Depth (m)', fontsize= 14)
        ax.set_title(f"{title} (N={len(this_ob)})", fontsize=16)
        
        ax.grid(True, alpha=0.25)
        ax.legend(fontsize=12)

plt.tight_layout()

# display(Markdown("""
# At each analysis time, there are observations from <mark>**several (4) drifters**</mark>. 
# There is one other SVP drifter that is already outside our InaCAWO domain; we've ignored it here. 
# """))

plt.show()
plt.savefig('fig/arvor2.png', format='png', dpi=90, bbox_inches='tight')
```
