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
# help(obs)
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

# We will plot a figure containing 3 different plots (maps)
fig, axes = plt.subplots(
    1, 3,
    figsize=(16, 7),
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

    plt.colorbar(sc, ax=ax, label=label, shrink=0.25)

    ax.set_extent([110, 140, -10, 0], crs=proj)

    gl = ax.gridlines(draw_labels=True, linewidth=0.5, color="gray", alpha=0.2)
    gl.top_labels = False
    gl.right_labels = False

    ax.set_title(f"{title} (N={len(this_ob)})")

# plt.suptitle("SVP Observations by Type", fontsize=18)
plt.tight_layout()

display(Markdown("""
The maps below show the locations and values of the different observation types in the `obs_seq` file. 
"""))

plt.show()
```
