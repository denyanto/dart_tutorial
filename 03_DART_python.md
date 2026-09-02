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
import numpy             as np
import pandas            as pd
import cartopy.crs       as ccrs
import cartopy.feature   as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates  as mdates
from pathlib import Path
from IPython.display      import Markdown, display
import cmocean

# Path to DART repo (directory) 
basedir = Path(f"{$rundart}")

# Path to the HF Radar converter
svp_dir = basedir / 'hf' 

# Path to the obs_seq file
obs_seq_file = svp_dir / 'obs_seq.hf'
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

# Total velocity observation types
u_type = "HFRADAR_U_CURRENT_COMPONENT"
v_type = "HFRADAR_V_CURRENT_COMPONENT"

df_u = df[df["type"] == u_type].copy()
df_v = df[df["type"] == v_type].copy()

# Merge U and V observations by time and location
df_totals = pd.merge(
    df_u,
    df_v,
    on=["time", "longitude", "latitude"],
    suffixes=("_u", "_v"),
)

df_totals = df_totals.rename(
    columns={
        "observation_u": "u",
        "observation_v": "v",
        "obs_err_var_u": "u_err_var",
        "obs_err_var_v": "v_err_var",
    }
)

# Compute the speed using both U and V
df_totals["speed"] = np.sqrt(df_totals["u"]**2 + df_totals["v"]**2)

print(f"Number of paired total velocity observations: {len(df_totals)}\n")

display(df_totals.head())

display(
    df_totals.groupby("time")
    .size()
    .rename("Number of total velocity vectors")
    .to_frame()
)

# Select one time to plot
plot_time = sorted(df_totals["time"].unique())[0]

ob_sub = df_totals[df_totals["time"] == plot_time]

proj = ccrs.PlateCarree()

fig = plt.figure(figsize=(9, 7))
ax = plt.axes(projection=proj)

ax.coastlines(resolution="10m")
ax.add_feature(cfeature.OCEAN, facecolor="aliceblue")
ax.add_feature(cfeature.LAND, facecolor="whitesmoke")
ax.add_feature(cfeature.BORDERS, linewidth=0.5)

lon, lat = ob_sub['longitude'], ob_sub['latitude']
Uvel, Vvel, Uall = ob_sub['u'], ob_sub['v'], ob_sub['speed']

q = ax.quiver(lon, lat, Uvel, Vvel, Uall, 
    cmap=cmocean.cm.speed,
    transform=proj,
    scale=3, width=0.006,
)

plt.colorbar(q, ax=ax, label="Current speed (m/s)", shrink=0.65)

lon_min, lon_max = df_totals["longitude"].min(), df_totals["longitude"].max()
lat_min, lat_max = df_totals["latitude"].min() , df_totals["latitude"].max()

ax.set_extent(
    [lon_min - 0.5, lon_max + 0.5, lat_min - 0.5, lat_max + 0.5],
    crs=proj,
)

# gl = ax.gridlines(draw_labels=True, linewidth=0.5, color="gray", alpha=0.3)
# gl.top_labels = False
# gl.right_labels = False

ax.set_title(f"HF Radar Total Velocity Vectors\n{plot_time}", fontsize=15)

plt.show()

# display(Markdown("""
# ### Interpretation of Total Velocity Observations

# HF radar total velocity observations provide both the eastward (U) and northward (V) components of the surface current. 
# <mark>**These observations therefore describe the full horizontal current vector at each observation location**</mark>.
# In the figure above, the arrows represent the observed surface current vectors, while the color indicates the current speed. 
# Total velocity observations are typically derived by combining radial measurements from multiple radar stations and provide 
# a direct estimate of the two-dimensional surface flow field.
# """))

fig.savefig('fig/hf1.png', format='png', dpi=90, bbox_inches='tight')

# Plot observation counts of the total velocity components over time
fig, axes = plt.subplots(1, 2, figsize=(14, 4))

ax = axes[0]
(df_totals.groupby("time")
          .size()
          .rename("Number of vectors")
          .plot(marker="o", ax=ax))
ax.set_ylabel("Count")
ax.set_title("HF Radar Total Velocity Observations Through Time")
ax.grid(alpha=0.3)

# Speed Histogram 
ax = axes[1]

ax.hist(df_totals["speed"], bins=20)
ax.set_xlabel("Current speed (m/s)")
ax.set_ylabel("Count")
ax.set_title("Distribution of HF Radar Current Speeds")
ax.grid(alpha=0.3)

plt.tight_layout()
plt.show()
fig.savefig('fig/hf2.png', format='png', dpi=90, bbox_inches='tight')

# Observation Statistics
print("Observation time range:")
print(f"{df['time'].min()} to {df['time'].max()}")

print("\nLongitude range:")
print(f"{df['longitude'].min():.2f}° to {df['longitude'].max():.2f}°")

print("\nLatitude range:")
print(f"{df['latitude'].min():.2f}° to {df['latitude'].max():.2f}°")

summary = []

for obs_type in sorted(df["type"].unique()):

    this = df[df["type"] == obs_type]

    summary.append({
        "Type"          : obs_type,
        "Count"         : len(this),
        "Min"           : this["observation"].min().round(3),
        "Max"           : this["observation"].max().round(3),
        "Mean"          : this["observation"].mean().round(3),
        "Std"           : this["observation"].std().round(3),
        "Error SD" : np.sqrt(this["obs_err_var"]).mean().round(3),
    })

summary = pd.DataFrame(summary)

display(summary)
```
<img width="615" height="548" alt="hf1" src="https://github.com/user-attachments/assets/4f7732b8-9ff0-4946-bf23-bd561df05285" />
<img width="710" height="621" alt="hf2" src="https://github.com/user-attachments/assets/307a570f-52d6-4725-bd95-6d1533a44beb" />

## Visualize Satellite SST observations on a map
Create a plotting folder to store the visualization scripts. Misalnya hf_ploting.py:
<details>
<summary>📜 Expand Python script</summary>
```python
import pydartdiags.obs_sequence.obs_sequence as obsq
import numpy             as np
import pandas            as pd
import cartopy.crs       as ccrs
import cartopy.feature   as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates  as mdates
from pathlib import Path
from IPython.display      import Markdown, display
import cmocean

# Path to DART repo (directory) 
basedir = Path(f"{$rundart}")

# Path to the SST converter
svp_dir = basedir / 'sst' 

# Path to the obs_seq file
obs_seq_file = svp_dir / 'obs_seq.sst'
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

# Extract data from the obs_seq file
df   = obs.df.copy()

data = df['observation'] 
lon  = df['longitude']
lat  = df['latitude']
err  = np.sqrt(df['obs_err_var'])
time = sorted(df['time'].unique())[0]

proj = ccrs.PlateCarree()

fig = plt.figure(figsize=(10, 6))
ax  = plt.axes(projection=proj)

ax.set_facecolor('aliceblue') 
ax.add_feature(cfeature.LAND, facecolor='whitesmoke', zorder=1) 
ax.add_feature(cfeature.NaturalEarthFeature('physical', 'coastline', '10m'), 
               edgecolor='black', facecolor='none', zorder=2)
ax.add_feature(cfeature.LAKES, facecolor='lightsteelblue', zorder=2)
ax.add_feature(cfeature.BORDERS, linewidth=0.5, zorder=2)


# Observations
sc = ax.scatter(lon, lat,
                c=data, s = 10,
                cmap=cmocean.cm.thermal, linewidth=0.5,
                zorder=3,
                transform=proj)

cb = plt.colorbar(sc, ax=ax, label='SST Obs Value (C)', shrink=0.6)

ax.set_extent([95, 145, -15, 15], crs=proj)

# gl = ax.gridlines(draw_labels=True, linewidth=0.5, color='gray', alpha=0.1)
# gl.top_labels   = False
# gl.right_labels = False

plt.title(f"Satellite L3 SST Observations\n{time}", fontsize=18)
plt.show()

# display(Markdown("""
# ### Interpretation of Satellite SST Observations

# Satellite SST observations provide estimates of sea surface temperature over large spatial areas. 
# Unlike in-situ observations, satellite measurements offer broad coverage but may contain spatially varying uncertainties. 
# The colors in the figure above represent the observed SST values.
# """))
fig.savefig('fig/sst1.png', format='png', dpi=90, bbox_inches='tight')

# Histogram 
fig=plt.figure(figsize=(8,5))

plt.hist(data, bins=30)

plt.xlabel("SST (C)", fontsize=14)
plt.ylabel("Count", fontsize=14)
plt.title("Distribution of Satellite SST Observations", fontsize=18)
plt.grid(alpha=0.3)
fig.savefig('fig/sst2.png', format='png', dpi=90, bbox_inches='tight')

# Display the errors
fig = plt.figure(figsize=(10, 6))
ax  = plt.axes(projection=proj)

ax.set_facecolor('aliceblue') 
ax.add_feature(cfeature.LAND, facecolor='whitesmoke', zorder=1) 
ax.add_feature(cfeature.NaturalEarthFeature('physical', 'coastline', '10m'), 
               edgecolor='black', facecolor='none', zorder=2)
ax.add_feature(cfeature.LAKES, facecolor='lightsteelblue', zorder=2)
ax.add_feature(cfeature.BORDERS, linewidth=0.5, zorder=2)


# Observations
sc = ax.scatter(lon, lat,
                c=err, s = 10,
                cmap='plasma', linewidth=0.5,
                zorder=3,
                transform=proj)

cb = plt.colorbar(sc, ax=ax, label='Obs Error SD (C)', shrink=0.6)

ax.set_extent([95, 145, -15, 15], crs=proj)

# gl = ax.gridlines(draw_labels=True, linewidth=0.5, color='gray', alpha=0.1)
# gl.top_labels   = False
# gl.right_labels = False

plt.title(f"Satellite L3 SST Observations\n{time}", fontsize=18)
plt.show()

# display(Markdown("""
# ### Interpretation of SST Observation Errors

# The colors in the figure above represent the observation error standard deviation assigned to each satellite SST observation. 
# Smaller values indicate observations that are expected to be more accurate and therefore receive greater weight during data assimilation. 
# The spatial variability in the assigned errors originates from the uncertainty estimates provided with the SST product. 
# Regions with lower uncertainties (dark colors) generally correspond to areas where the satellite retrievals are considered more reliable, 
# while regions with higher uncertainties (bright colors) indicate reduced confidence in the observations.
# These uncertainties are carried into the DART observation sequence file and are used during 
# assimilation to determine how strongly each observation influences the model analysis.
# """))
fig.savefig('fig/sst3.png', format='png', dpi=90, bbox_inches='tight')


print(f"Longitude range : {df['longitude'].min():.2f}° to {df['longitude'].max():.2f}°")
print(f"Latitude range  : {df['latitude'].min():.2f}° to {df['latitude'].max():.2f}°")
print(f"Time range      : {df['time'].min()} to {df['time'].max()}\n")

summary = pd.DataFrame({
    "Count"        : [len(df)],
    "Min SST"      : [df["observation"].min()],
    "Max SST"      : [df["observation"].max()],
    "Mean SST"     : [df["observation"].mean()],
    "Std SST"      : [df["observation"].std()],
    "Mean Error"   : [np.sqrt(df["obs_err_var"]).mean()],
})

display(summary.round(3).style.hide(axis="index"))
```
</details>
<img width="667" height="406" alt="sst1" src="https://github.com/user-attachments/assets/abfcecdc-96a3-4b03-8a6f-0d5174fbf2c3" />
<img width="639" height="433" alt="sst2" src="https://github.com/user-attachments/assets/6c636468-400f-4b44-ba16-72348f753821" />
<img width="679" height="406" alt="sst3" src="https://github.com/user-attachments/assets/3ecf9629-eaab-42e4-bfd8-664d8a73fd3b" />

