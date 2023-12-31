# Walkthrough
Short description of aim, task to be completed


## Table of Contents
0. [Requirements](#requirements)
   
1. [Installation](#installation)

   1.1 [First time](#first)

   1.2 [Updating](#updating)

2. [Step by step guide](#steps)

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
git clonehttps://github.com/bencardoen/SPECHT.jl.git
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

If you want to count objects per cell, you can do so by `groupby` features in R, aggregating over the columns `replicate, cellnumber, treatment, channel`. 

## FAQ

### Different channel naming pattern

Q: My files are ending with 0.tif and 1.tif, how do I make this work?

A: Change the parameter to modify the pattern `--pattern "*[0,1].tif"`
