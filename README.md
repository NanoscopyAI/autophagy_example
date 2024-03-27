# Walkthrough
Short description of aim, task to be completed


## Table of Contents
0. [Requirements](#requirements)
   
1. [Installation](#installation)

   1.1 [First time](#first)

   1.2 [Updating](#updating)

2. [Step by step guide](#steps)

3. [Troubleshooting](#faq)

<a name="requirements"></a>
## 0. Requirements
Internet access, [VSCode](https://code.visualstudio.com/download) installed, [Julia](https://julialang.org/downloads/) installed. Optionally in VSCode [install the Julia plugins](https://code.visualstudio.com/docs/languages/julia). 


<a name="installlation"></a>
## 1. Installation

<a name="first"></a>
### 1.1 First time installation
- Open VSCode
- Open a terminal File -> Terminal -> New Terminal
- In the terminal, we will clone SPECHT.jl:
```bash
git clone https://github.com/bencardoen/SPECHT.jl.git
```

The output will look similar to this:
```
Cloning into 'SPECHT.jl'...
remote: Enumerating objects: 438, done.
remote: Counting objects: 100% (192/192), done.
remote: Compressing objects: 100% (107/107), done.
remote: Total 438 (delta 102), reused 140 (delta 63), pack-reused 246
Receiving objects: 100% (438/438), 10.57 MiB | 1.44 MiB/s, done.
Resolving deltas: 100% (222/222), done.
```
- Change directory into `SPECHT.jl` (in the file explorer on your left hand side in VSCode, you should now see the SPECHT.jl directory)
```bash
cd SPECHT.jl
```

Let's make sure everything works, we will run the self-tests.

```bash
julia --project=. -e 'using Pkg; Pkg.test()'
```
The output should look like this
```bash
Test Summary: | Pass  Total     Time
SPECHT.jl     | 9053   9053  1m33.1s
     Testing SPECHT tests passed
```

<a name="updating"></a>
### 1.2 Updating

If you already cloned/installed SPECHT, then the following steps are needed:
- Open VSCode
- Open New Window
- Select Open Folder
- Navigate until you find SPECHT.jl
- File -> Terminal -> New Terminal (or open existing Terminal window)
- Now type

```bash
git pull origin main
```

If there are no updates, this will look like

```bash
* branch            main       -> FETCH_HEAD
Already up to date.
```

<a name="steps"></a>
## 2. Step-by-step guide

### 2.1 Preparing the data
Ensure your data is segmented (only 1 cell, no background, in view, in tif files). 
The structure should be like so
```
- datadirectory
  - replicate (1, 2, ...)
    - treatment (e.g. HT-BaF)
      - cellnumber (1, 2, ...)
        - ...1.tif  (channel 1)
        - ...2.tif  (channel 2)
```

### 2.2 Running the analysis
In the terminal window, type
```bash
 julia --project=. scripts/2ch.jl --inpath datadirectory --outpath outputdirectory
```

The script will verbosely tell you what data it discovers:
```bash
[ Info: 2023-12-22 16:17:30 2ch.jl:113: Replicate 1 Treatment HT CCCP -BaF siAMF Cell 001
```

When completed, you'll see something like this
```bash
[ Info: 2023-12-22 16:20:01 2ch.jl:130: Saving tabular results in /outputdirectory/table_spots.csv
```
--pattern "*[0,1].tif"
(note on Linux paths are separated by '/', on Windows it will be '\'.)

At the end you will have 
- For each cell
  - 1 cell mask
  - 1 spot mask per channel (2 total)
- 1 CSV table with columns `distance_to_other,area,channel,replicate,cellnumber,treatment`
  - 1 row corresponds with 1 detected object.
    - The `distance_to_other` column denotes the euclidean distance in pixels to the nearest object in the other channel.
    - The area is the number of non zero pixels of this object (mask size)
    - The channel, replicate, cellnumber and treatment describe what data this object belongs to.

A 2nd csv is generated for you, contained counts of spots per cell, per channel. Spots that overlap are counted as well, the channel column is then set to '12', rather than '1' or '2'.

You can filter the minimum size of spots to be counted, as well as the minimum number of pixels of overlap to count before counting colocalizing.

For example, to ignore (do not count) spots that have size < 5 pixels, and overlap > 1 pixel, you would pass these arguments:
```bash
 julia --project=. scripts/2ch.jl --inpath datadirectory --outpath outputdirectory --min_overlap 1 --filterleq 5
```

### 2.3 Enabling autotuning

If you prefer the code to use its adaptive mode, you can enabling autotuning, by adding these two parameters
```bash
 --auto-tune --prc 1
```

* 'auto-tune' enables the autotuning mode, so -z will now be computed for you. 
* 'prc x' sets the precision recall balance, > 1 is more recall (more objects), < 1 is fewer objects

**IMPORTANT**
The detection relies on the idea that there is heavy background noise. If you have images where there is almost no noise, increase PRC to say 2-4 instead, otherwise you may not pick up the objects you intend to pick up. 
In images with heavy noise, a PRC of +- 1 is a good starting point.

<a name="faq"></a>
## 3. FAQ

### 3.1 Different channel naming pattern

Q: My files are ending with 0.tif and 1.tif, how do I make this work?

A: Change the parameter to modify the pattern `--pattern "*[0,1].tif"`

### 3.2 Directory/file not found errors

Q: I get directory not found, but it's right there?!

A: On Windows, use the \ path separator
Windows:
```bash
julia --project=. scripts\2ch.jl --inpath data\ND --outpath data\Output --pattern "*[0,1].tif"
```

Linux/Mac:
```bash
julia --project=. scripts/2ch.jl --inpath data/ND --outpath data/Output --pattern "*[0,1].tif"
```

When in doubt, check where the current directory is in the script
```bash
julia -e '@info pwd()'
```
This will print the full path of the current directory, the script looks in relative directories below this path (unless you provide a full or absolute path).

Q: (Mac) I get .DS_Store is not a directory ?!

A: .DS_Store is hidden metadata used by Mac, you can remove it from the data to prevent the script from getting confused
Assuming you want to use the `data` folder, this command recursively deletes it from the data folder
```bash
find ./data -name ".DS_Store" -print -delete
```
### I want more complex grouping, how do I do this?

Suppose you want to exclude spots with mean intensity < 0.25, before counting them. 
Let's assume the csv table for all spots is saved in the directory `output/table_spots.csv`:

Start a Julia session in the top directory of the repository
```bash
julia --project=.
```
In Julia, then execute the following
```julia
using CSV, DataFrames, SPECHT
# Load the table
df = CSV.read("output/table_spots.csv", DataFrame)
# Drop all spots with mean intensity < 0.25
df = filter(row -> row.mean_intensity >= 0.25, df)
# Reuse the counting function
grouped_df = group_data(df, 0, 0) # Any overlap, all sizes
# Save to CSV
CSV.write("intensityfiltered.csv", grouped_df)
```
You can also save this snippet as a script (for example `intensity.jl`), and execute it like so
```bash
julia --project=. intensity.jl
```
More complex rules can be executed in similar fashion. Alternatively, both R and Python can work with the CSV file for custom postprocessing.

### NOTE: If there are no spots for a channel in a cell, there will not be any lines in the CSV file for that cell and channel.

### Lowering or increasing the detection of spots
By default the adaptive threshold to pick up spots is set at 1.75. If you lower this, more candidate spots can be detected, at the risk of false positives. 
If you increase it, you risk more false negatives (no spots where there should be). 

Changing it is done as follows, for example lowering from 1.75 to 1.5:
```bash
julia --project=. scripts\2ch.jl --inpath data\ND --outpath data\Output --pattern "*[0,1].tif" --zval 1.5
```
