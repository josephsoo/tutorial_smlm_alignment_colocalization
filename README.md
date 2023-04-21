# Walkthrough
The below steps allow you to run fiduciual alignment on 2/3D point cloud data, then compute colocalization metrics.

## What this will do for you:
- Given any number of directories with 2 CSV files (from Thunderstorm 2D point cloud data)
- Detect fiducials, by looking for peaks in emissions. Fiducials in SMLM continuously emit, so treating the emissions as a distributions we can select the local maxima (set to top 2 now).
- Pair the fiducials across the channels
- If the closest pair is > 400nm apart
  - Abort
- Track their fiducials, if < 400nm apart (center to center)
- Correct temporal drift
- Align the channels
- Compute localization metrics (10)
- Save the output in CSV and image format

Correction is done by a linear translation in 3D using euclidean distance.

### Step 1
Log in to cluster
```bash
ssh you@computecanada.ca
```
You'd see something like this
```
[YOU@cedar5 ~]$
```
Change to `scratch` directory
```bash
cd /scratch/$USER
```
Now it'll show
```bash
[you@cedar5 /scratch/YOU]$
```
Create a new directory, to make sure existing files do not clash:
```
mkdir -p testexperiment
cd testexperiment
```

### Step 2
Copy your data to a folder under /scratch/$USER, preferably using [Globus](https://globus.computecanada.ca/)

### Step 3
Get Compute resources:
Replace `FIXME` with an account ID, which is either `def-yourpiname` or `rrg-yourpiname`. Check ccdb.computecanada.ca, or the output of `groups`.
```bash
salloc --mem=62GB --account=FIXME --cpus-per-task=8 --time=3:00:00
```
Once granted this will look like this:
```bash
salloc --mem=62GB --account=def-hamarneh --cpus-per-task=8 --time=3:00:00
salloc: Pending job allocation 61241941
salloc: job 61241941 queued and waiting for resources
salloc: job 61241941 has been allocated resources
salloc: Granted job allocation 61241941
salloc: Waiting for resource configuration
salloc: Nodes cdr552 are ready for job
[bcardoen@cdr552]$
```
Set the DATASET variable to the name of your dataset
```bash
export DATASET="/scratch/$USER/FIXME"
```

**NOTE** Please make sure your dataset is organized like so:
```
yourdatasetdirectory
  --cell1
    --file1.csv
    --file2.csv
  --cell2
    -- ...
```
You are free to choose the dataset directory naming, as well and the 'cell1' and 'cell2' directories (or even have just 1 subdirectory), but the data is expected to be nested.

The remainder is done by executing a script, to keep things simple for you.
This script assumes you want to process dStorm data in CSV format, output by Thunderstorm.
```bash
wget https://raw.githubusercontent.com/NanoscopyAI/tutorial_smlm_alignment_colocalization/main/script.sh -O script.sh && chmod u+x script.sh
```
For GSD data (bin, ascii).
```bash
wget https://raw.githubusercontent.com/NanoscopyAI/tutorial_smlm_alignment_colocalization/main/script_lydia.sh -O script.sh && chmod u+x script.sh
```
Make it executable
```bash
chmod u+x script.sh
```
### 3.1 -- OPTIONAL
**OPTIONAL NOTE** Processing uses `recipes`, text files that describe in plain language what should be done. 
If you do not have a recipe, it will be downloaded for you. 
However, **if you want to change parameters**, save a `recipe.toml` file in your current directory, and the script will skip downloading a new one.
For example
#### Download the thunderstorm recipe
```bash
  wget https://raw.githubusercontent.com/bencardoen/DataCurator.jl/main/example_recipes/coloc_and_align.toml -O recipe.toml    
```
Change the filtering and fiducial parameters (you can use a text editor such as [Nano, which is preinstalled](https://linuxize.com/post/how-to-use-nano-text-editor/)
Example recipes can be found [here](https://github.com/bencardoen/DataCurator.jl/blob/main/example_recipes/coloc_and_align.toml)
You'd change this line
```toml
actions=[["smlm_alignment",".csv", "is_thunderstorm", 500, 5], ["image_colocalization", 3, "C[1,2].tif", "is_2d_img", "filter", 1]]
```
to now detect up to **10** fiducials, and be a lot more stringent in filtering before colocalization. Also, you want a 5x5 window, not 3x3. You're ok with fiducials being a bit further apart as well (600).
Edit the recipe so that it reads like this:
```toml
actions=[["smlm_alignment",".csv", "is_thunderstorm", 600, 10], ["image_colocalization", 5, "C[1,2].tif", "is_2d_img", "filter", 2]]
```

### 3.2 Execute the script
```bash
./script.sh
```
That's it. Your output is now stored in the same folders as your source data.
At the end you'll see something like
```bash
 Info: 2023-02-27 06:14:21 curator.jl:180: Complete with exit status proceed
+ echo Done
Done
[you@cdrxyz scratch]$ 
```

This includes, but is not limited to
- Aligned.csv files for point cloud data
- Colocalization images for all implemented metrics

For each execution, temporary output is saved in the directory `tmp_{DATE}`.

See below for more docs.

### Troubleshooting
See [DataCurator.jl](https://github.com/NanoscopyAI/DataCurator.jl), [SmlmTools](https://github.com/NanoscopyAI/SmlmTools.jl) and [Colocalization](https://github.com/NanoscopyAI/Colcocalization.jl) repositories for documentation.

#### Possible errors
##### File not found errors
- this occurs if you ask to look for csv files, but the files are of .bin format. Or if there are more than 2 files, or less than two files in the folder.
##### Fiducials too far apart
- You'll see a message "nearest pair = .... nm", if this exceeds 400nm (default center to center distance), the code will refuse to align.

Create an [issue here](https://github.com/NanoscopyAI/tutorial_smlm_alignment_colocalization/issues/new/choose) with
- Exact error (if any)
- Input
- Expected output

