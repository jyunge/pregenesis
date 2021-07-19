# Pre-genesis stage: Observation framework using polar-orbiting satellites

Author: Jimmy Yunge (University of Miami RSMAS)

Email: `jyg3@miami.edu`

Last updated: 19 July 2021

Scripts (detailed documentation below):
- `pregenesis.py`
- `runmirs.py`
- `runairs.py`
- `example_post_process.py`

Please inform me of any necessary clarification, grammatical or spelling errors, etc. I appreciate any comments on my coding.

## Microwave Integrated Retrieval System (MIRS)
This section of the project file describes work done using Microwave Integrated Retrieval System (MIRS) data. Swath files were retrieved from the NOAA Comprehensive Large Array-Data Stewardship System (CLASS) [https://www.class.noaa.gov/ > "MIRS Orbital Data (MIRS_ORB)"].

MIRS data was offered beginning 29 August 2007. Of the eight platforms to choose from on CLASS, two of them; DMSP F17 and GPM; did not offer any data in the time interval analyzed so far (01 Jan 2002 - 31 Dec 2015).

CLASS offered two formats for the data: HDF and NetCDF. For NOAA-18, METOP-A, and DMSP F16, substantial amounts of data were available only as HDF (see Table 1.1). **However, the analysis was only performed on the NetCDF files.** This is due to swath "overlapping" or "concatenation" issues that will be described in detail below.

| Satellite          | HDF                           | NetCDF                    |
|--------------------|-------------------------------|---------------------------|
| NOAA-18 (N18; NN)  | ~~29 Aug 2007 - 30 Mar 2009~~ | 30 Mar 2009 - 31 Dec 2015 |
| NOAA-19 (N19; NP)  | None                          | 06 Jan 2009 - 31 Dec 2015 |
| METOP-A (M2)       | ~~29 Aug 2007 - 30 Mar 2009~~ | 30 Mar 2009 - 31 Dec 2015 |
| METOP-B (M1)       | None                          | 23 Apr 2013 - 31 Dec 2015 |
| DMSP F16 (SA)      | ~~27 Oct 2008 - 30 Mar 2009~~ | 30 Mar 2009 - 24 Feb 2014 |
| DMSP F17           | None                          | None                      |
| DMSP F18 (SC)      | None                          | 17 Sep 2012 - 31 Dec 2015 |
| GPM                | None                          | None                      |

**Table 1.1.** _MIRS data availability on CLASS by formats and date. HDF files may be available after the end date indicated above, but this was not verified since only NetCDF files were considered for analysis. Note that METOP-A and METOP-B are M2 and M1, respectively._

### MIRS code documentation

#### pregenesis.py
This is a module that stores functions used in other scripts. This module also contains `invests_dict`, a nested Python dictionary which contains, for each year, a list of the Invests for developing and non-developing disturbances.

:bangbang: &nbsp; **This file must be imported for** `runmirs.py` **to work.** To import `pregenesis.py`, make sure that it is in the Python path.

---

#### runmirs.py
The main script, this code searches MIRS swath files for storm overpasses. Performance could be improved.

Program flow:
- Import packages
- Define local functions
- Set parameters
- Data pre-processing
- Main loop

**Part 1: Import packages.** Install packages as needed. Again, `pregenesis.py` must be imported.

**Part 2: Define local functions.** Place any user-defined functions here.

**Part 3: Set parameters.**

The user inputs all program settings here. :warning: &nbsp; **All user control over the program is in this block.**

Path variables (all type `str`):
- `path_to_mirs`: the parent directory containing all satellite directories.
- `path_to_best_tracks`: the parent directory for TC-PMW best tracks.
- `path_to_output`: the parent directory for outputted "pixel" files.

> :warning: &nbsp; **IMPORTANT:** The program will only run if the raw data is structured in the following way (See Figure 1.1):
> 
> `/home/<user>/.../mirs/<satellite>/<year>/*.nc`
> 
> (in this case, `path_to_mirs = '/home/<user>/.../mirs/'`), where *.nc are all MIRS NetCDF files with the same file names as downloaded from NOAA CLASS, except partitioned into folders by year and satellite. For example, for the NOAA-18 satellite, once downloaded, the data should be organized into:
> ```
> /home/<user>/.../mirs/n18/2009/*.nc
> /home/<user>/.../mirs/n18/2010/*.nc
> /home/<user>/.../mirs/n18/2011/*.nc
> /home/<user>/.../mirs/n18/2012/*.nc
> /home/<user>/.../mirs/n18/2013/*.nc
> /home/<user>/.../mirs/n18/2014/*.nc
> /home/<user>/.../mirs/n18/2015/*.nc
> ```
> In addition, the program will only run if the best-track files from TC-PMW have maintained their original structures from INVESTS.zip. _Any future adjustments made to INVESTS.zip or best-track files should be noted here._ Currently, the format is
> 
> `/home/<user>/.../INVESTS/<year>/<dev>/*`
> 
> where * are SHIPS, track (NetCDF), DAT files, etc.

[Figure 1.1.]

> :warning: &nbsp; **IMPORTANT:** The program will output data in a similar structure inside a directory specified by a user (Figure 1.2). For example, if `path_to_output = '/home/<user>/.../output/'`, the program will output all pixel files to
> 
> `/home/<user>/.../output/<satellite>/<year>/<dev>/<invest>/*.nc`
> 
> where *.nc are all subsetted pixel files (NetCDF) for that satellite, year, dev/non-dev, and Invest. For example, for al_ana2009 using NOAA-18 data, the program will output
> ```
> /home/<user>/.../output/n18/2009/dev/al_ana2009/al_ana2009_20090809_0240_pixels.nc
> /home/<user>/.../output/n18/2009/dev/al_ana2009/al_ana2009_20090809_1533_pixels.nc
> /home/<user>/.../output/n18/2009/dev/al_ana2009/al_ana2009_20090810_1522_pixels.nc
> ...
> ```
> and as another example, for al20090517INV90,
> ```
> /home/<user>/.../output/n18/2009/nondev/al20090517INV90/al20090517INV90_20090517_1839_pixels.nc
> /home/<user>/.../output/n18/2009/nondev/al20090517INV90/al20090517INV90_20090518_0713_pixels.nc
> /home/<user>/.../output/n18/2009/nondev/al20090517INV90/al20090517INV90_20090518_1829_pixels.nc
> ...
> ```
> (note that the satellite is _not_ included in the pixel file name, by choice).

[Figure 1.2.]

Run variables:
- `sat_opt`: Integer specifying which satellite to be run (see code for input options).
- `years`: Integer or range of integers specifying years to run through.
- `dev`: Set to `True` (run developing storms only) or `False` (run non-developing storms only).
- `basin_opt`: Integer specifying which basin to be run. Set to `None` to run all basins for that year(s).
- `invest_opt`: String specifying which Invest to run. Set to `None` to run all invests for the year(s) and basin specified, if any.

> **NOTE:** If `invest_opt` is specified, then `years`, `dev`, and `basin_opt` are ignored. For example, if `years = range(2009, 2016)`, `dev = False`, `basin_opt = 3` (E. Pacific), but `invest_opt = 'al_ana2009'`, then the program will still run and only search for swath files in the time range of Ana (2009) (based on the TC-PMW best track).

Other variables:
- `concat_lim_deg`: Concatenation threshold, in radial degrees.
- `make_plots`: `True` or `False` to plot each successful overpass (time consuming, but recommended for verification).
- `save_plots`: `True` or `False` to save plots as PNG.

---

**Part 4: Data pre-processing.**

This part of the script takes the parameters specified in the previous section and converts ("standardizes") them into strings, etc. for processing in the main loop. For example, if `sat_opt = 1`, then the program assigns `sat = 'n18'` to look in the NOAA-18 data folder named `n18/`, assuming the paths are structured as described in Part 3. As another example, if `basin_opt = 1`, then the program defines `basin = 'al'`. These strings are used in the paths to search for data and in the main loop.[_nb_ 1] (Of course, if changes are made to the Invest name formatting, etc., then those changes will need to be reflected in this section.)

> [_nb_ 1] Arguably, one could make the user enter in these strings directly in Part 3, although the strings must be formatted exactly as is indicated (e.g., for the Atlantic basin, it must be 'al', not 'AL' or 'natl'). Although the strings are short, it might be simpler to just have the user input the parameters as integers and have those be converted to strings by the code. Again, if changes are made to the invest name format, etc., then those changes will need to be reflected in this section.

Also in this section, we generate a sorted list of Invests to loop through based on the specified run parameters. For example, if:
- `dev = True`
- `years = range(2009, 2016)`
- `sat_opt = 1`
- `basin_opt = None`
- `invest_opt = None`

then a `list` of length 632 will be generated:

`['al_ana2009', 'al_bill2009', ..., 'wp_ts122015', 'wp_vamco2015']`

Lastly in this section, `concat_lim_deg` is converted into grid points (how close to the grid boundary the center needs to be to trigger concatenation). This varies by satellite and the conversion values are given (hard-coded) in the script.

---

**Part 5: Main loop (summary).**

For each invest in the list generated in Part 4:

- Determine whether to use IDL0 coordinates (see Appendix [[link](https://github.com/jyunge/pregenesis/new/main?readme=1#appendix)]).
- Get paths to input and output files.
- Open TC-PMW best-track file.
  - Load best-track time, longitude, latitude, BThrgen.
  - Convert best-track time data to `datetime` objects and UNIX timestamps.
  - :exclamation: **[Check 1]** If the best-track file does not have data after 2009 (beginning of MIRS NetCDF dataset), skip to the next Invest.
  - Set missing best-track values to NaN.
  - Create function to interpolate best-track coordinates for later.
- Get MIRS swath files between the best-track time range for the Invest.
- For each MIRS swath file:
  - Perform checks. If a check fails, skip to the next swath file.
    - :exclamation: **[Check 2]** Check if the swath time falls within the best-track time range for the Invest (redundancy for Check 1). If not, skip to the next swath file.
    - :exclamation: **[Check ?]** Check for duplicate or truncated files (will elaborate at some point).
    - Load swath file into Python.
    - Overlap handling. (See below for full explanation. [[link](https://github.com/jyunge/pregenesis/new/main?readme=1#overlap-issues-concatenation-and-hdf-files)])
    - Time handling. Convert swath time data to `datetime` obbjects and timestamps.
    - :exclamation: **[Check 4]** Check if times in the first dataset are later than the start of the second dataset. (This should not happen except in bad files, which should have been rejected by [Check ?].)
    - Load coordinate arrays.
    - :exclamation: **[Check 5]** Check if the entire swath is NaN (missing data due to instrument error, etc.).
    - :exclamation: **[Check 6]** Check that scan times are within the best-track time range (in between first and last times in the best-track file). Set all out-of-range times to NaN (for interpolation to work).
    - Interpolate 3-hourly best-track coordinates to finer-resolution scan times.
      - For MIRS data, each grid row has a corresponding scan time, e.g., `ScanTime_UTC`. The interpolation calculates the storm location to the scan time in _every row_ to get the closest possible grid point to the storm center.
    - Calculate the closest grid point in the swath to the storm center.
    - :exclamation: **[Check ?]** Redundancy for Check 5.
    - :exclamation: **[Check 8]** Check if any swath data exists within 0.5 degrees of the storm center (i.e., if the swath is near the storm center).
    - :exclamation: **[Check 9]** Check if the center is in the swath interior.
  - Checks have been completed; load in all data arrays from the swath file.
  - If the storm center is too close to either along-track edge of the swath, perform concatenation (_see below_ [[link](https://github.com/jyunge/pregenesis/new/main?readme=1#overlap-issues-concatenation-and-hdf-files)]).
    - Overlap handling.
  - Convert data to numpy arrays to store as NetCDF; convert missing values (-999) to NaN.
  - Write pixel file to output directory.
  - Plot if designated earlier.

---

##### Overlap issues, concatenation, and HDF files

For whatever reason, MIRS files have small overlapping portions of data at the beginning and end of each swath. That is, for some swath (from a sorted list of files), the first few rows will be identical to the last few rows of the previous swath, and the last few rows will be identical to the first few rows of the next swath. This creates rather obnoxious issues for concatenation, accidentally getting double positives for center-finding, etc. Below we describe the method used for handling this and why this was prohibitive for analyzing HDF files.

**Overlap handling technique (see Figs. 1.3 and 1.4):** As we load swath files from the sorted list, the current swath file is loaded in as `ds1` ("Dataset 1"). To deal with the overlap issue, we load in the variable `ds1.ScanTime_UTC` (or `ds1['ScanTime_UTC']`). This is a 1-D array (length = Scanline) that records, for each scanned row, the seconds elapsed since 00:00 UTC of the day in which the swath began.

We then load in the next swath file in the sorted list as `ds2` ("Dataset 2") and `ds2.ScanTime_UTC` and find the last index, denoted `a`, of `ds1.ScanTime_UTC` that occurs before the first time given in `ds2.ScanTime_UTC`. Hence, we only consider the _unique_ data in the range `0:a+1` for swath `ds1`. (Since Python excludes the last index in a range, we use `a+1` to include row `a`.) Thus, the swath `ds1` has been slightly truncated[_nb_ 2], and if we must concatenate from `ds2` (Edge 2; Fig. 1.3), we do not concatenate duplicate rows; we only concatenate unique data, forming a continuous transition (Fig. 1.4).

> [_nb_ 2] The truncation is not an issue because once ds1 has been processed, ds2 is loaded in as the current array ("ds2" becomes "ds1"), and the first few (non-truncated) rows of the ds2 are identical to the truncated rows of ds1. This process continues for the duration of the loop.

If the storm center is too close to the "near" edge (Edge 1; Fig. 1.3), then we want to concatenate data from the previous swath. The previous swath is loaded as `ds0` and must also be processed for overlapping. We find the last index, denoted `b`, of `ds0.ScanTime_UTC` that occurs before the first time in `ds1.ScanTime_UTC`, and truncate the swath `ds0` before concatenating to get only the unique, non-overlapping data in `ds0`.

Since the HDF files do not have the variable `ScanTime_UTC` (or any other time variables), it is much more difficult to determine the extent of overlap between swaths. We have found that the swath start and end times in the file name of NetCDF files do not match with the first and last values of `ScanTime_UTC`, so creating our own interpolated time variable would not guarantee that all overlapping data will be correctly identified. For future work, another method that, for example, checks for equality of values in rows of other data arrays (temperature, water vapor, etc.), could be implemented.

[Figures 1.3, 1.4.]

---

#### example_post_process.py
A short script that shows how to do further subsetting of pixel files after running the main code, for example, to partition the pixel files into those with complete data 3 radial degrees around the storm center.

### References and useful links
NOAA/OSPO website on the Operational Microwave Integrated Retrieval System, general overview: https://www.ospo.noaa.gov/Products/atmosphere/mirs/index.html

NOAA/STAR MIRS page with instrument details, algorithm description: https://www.star.nesdis.noaa.gov/mirs/index.php

NESDIS/STAR/JPSS website on MIRS with detailed documentation: https://www.star.nesdis.noaa.gov/jpss/mirs.php

NCEI page for dataset: https://doi.org/10.7289/V5X34VH3

---

## Atmospheric Infrared Sounder (AIRS)

### Data access and availability
AIRS/Aqua L2 Standard Physical Retrieval (AIRS+AMSU) V006 (AIRX2RET_V006) data was retrieved from https://disc.gsfc.nasa.gov/datasets/AIRX2RET_006/summary?keywords=airs.

A NASA EarthData account is needed to download data using the following steps:

> **Using wget for Mac/Linux** (from https://disc.gsfc.nasa.gov/data-access#mac_linux_wget)
> 
> 1. Make sure you have set up your Earthdata account. (https://disc.gsfc.nasa.gov/#top)
> 
> 2. Install wget if necessary. A version of wget 1.18 compiled with gnuTLS 3.3.3 or OpenSSL 1.0.2 or LibreSSL 2.0.2 or later is recommended.
> 
> 3. Create a .netrc file in your home directory.
> 
> ```
> cd ~ or cd $HOME
> touch .netrc
> echo "machine urs.earthdata.nasa.gov login <uid> password <password>" >> .netrc
> ```
> (where \<uid\> is your user name and \<password\> is your Earthdata Login password without the brackets);
> 
> `chmod 0600 .netrc`
> 
> (so only you can access it).
> 
> 4. Create a cookie file. This file will be used to persist sessions across calls to `wget` or `curl`.
> 
> `cd ~` or `cd $HOME`
> 
> `touch .urs_cookies`
> 
> Note: you may need to re-create `.urs_cookies` in case you have already executed wget without valid authentication.
> 
> 5. Download your data (single data file or directory only) using wget:
> ```
> wget --load-cookies ~/.urs_cookies --save-cookies ~/.urs_cookies --auth-no-challenge=on --keep-session-cookies --content-disposition <url>
> ```
> - `--auth-no-challenge` may not be needed depending on your version of wget.
> - \<url\> is the link that points to a file you wish to download or to an OPeNDAP resource.
> - Your Earthdata password might be requested on the first download.
> - If you wish to download an entire directory, such as this [example URL](https://acdisc.gesdisc.eosdis.nasa.gov/data//Aqua_AIRS_Level3/AIRX3STD.006/2006/), use the following command:
> ```wget --load-cookies ~/.urs_cookies --save-cookies ~/.urs_cookies --auth-no-challenge=on --keep-session-cookies -np -r --content-disposition <url>```
> - :bangbang: &nbsp; To download multiple data files at once, create a plain-text \<url.txt\> file with each line containing a GES DISC data file URL. Then, enter the following command:
> ```
> wget --load-cookies ~/.urs_cookies --save-cookies ~/.urs_cookies --auth-no-challenge=on --keep-session-cookies --content-disposition -i <url.txt>
> ```

Since 2015 was the latest year in the TC-PMW dataset, the current AIRS analysis is only for swaths up to 31 Dec. 2015. The full time range for AIRS L2 V6 is 30 Aug. 2002 to 24 Sep. 2016.

Maps of "granules" (swaths) for the entire AIRS time range can be found at https://airsl1.gesdisc.eosdis.nasa.gov/data/Aqua_AIRS_Level1/AIRXAMAP.005/. The numbers in each map represent the granule number in the file name ("ggg" in `<AIRS.yyyy.mm.dd.ggg.Lev.productType.vm.m.r.b.GproductionTimeStamp.hdf>`); see README.AIRS_V6.pdf, p.17; and 20070301_L2_ATBD_signed.pdf, p.17, for information on how time (UTC) is obtained from granule number.

AIRS appeared to suffer major issues on 2003-08-12 and 2003-10-07 and possibly other dates; see
https://airsl1.gesdisc.eosdis.nasa.gov/data/Aqua_AIRS_Level1/AIRXAMAP.005/2003/224/AIRS.2003.08.12.GranuleMap.pdf;
https://airsl1.gesdisc.eosdis.nasa.gov/data/Aqua_AIRS_Level1/AIRXAMAP.005/2003/280/AIRS.2003.10.07.GranuleMap.pdf.

Lastly, note that V7 retrieval is available as of 2020(?); however, analysis was performed in 2019 when only V6 was available. The following link is for the V7 algorithm, if desired: https://disc.gsfc.nasa.gov/datasets/AIRX2RET_7.0/summary?keywords=airs.

### Code summary and workflow

#### pregenesis.py
See MIRS section. **This module must be imported for** `runairs.py` **to work.**

#### runairs.py
The workflow/philosophy is nearly identical to that for MIRS (`runmirs.py`), i.e., parameters, then main loop through invests; so a full step-by-step description will not be given. We note some key differences:

- AIRS analysis was performed first and so the code is not as optimized, or as well commented, as for the MIRS code. However, the code indeed _works_. The motivated programmer may feel free to clean, add comments to, or speed up the code.

**Time handling:**
- The AIRS raw files use "TAI" time, defined as floating-point elapsed seconds since Jan 1, 1993. The timestamps for calculations and saved in the pixel file are TAI, not UTC time.
- The time data (variable `Time` in the HDF) associates a scan time to ALL pixels in the swath, so `Time` has increasing values as the scan proceeds from left (of satellite motion) to right until the end of the row, along the track. This is opposed to MIRS where only each along-track row of pixels is assigned a time.

**Interpolation:**
- The time data just described above allows for even finer interpolation of storm position within the granule. However, since each granule only encompasses 6 minutes of data, it is arguable whether such fine interpolation is useful at all, given the additional (but small) computation time. I would estimate that the finer interpolation may catch 1-10 fringe overpass cases throughout the entire analysis time period.

**Concatenation:**
- The AIRS data do not overlap as in the MIRS data, which greatly simplifies loading and concatenating swaths.
- The concatenation scheme is somewhat less efficient than for MIRS (which was developed after). Instead of the MIRS concatenation method which uses "near" and "far" edges ("Edge 1" and "Edge 2") and bypasses the need to determine "ascending" vs. "descending" nodes, the AIRS code uses an admittedly crude method to determine "north" or "south" edge (Fig. 2.1). However, it absolutely works.

[Fig. 2.1.]

**Data/coordinates:**
- There are two vertical dimensions in the AIRS data, `pressStd` and `pressH2O`, where `pressH2O` is simply the first fifteen levels of `pressStd`.
- The AIRS data files have many variables, but (currently) only water vapor mixing ratio, air temperature, and relative humidity, as well as associated QC arrays, are saved to the pixel files. See Table 2.1, and V6_L2_Product_User_Guide.pdf for descriptions of other AIRS variables. Note that relative humidity is already a variable and does not need to be computed separately as in MIRS.

[Table 2.1.]

### References and useful links
- Data retrieval:
  - https://disc.gsfc.nasa.gov/datasets/AIRX2RET_006/summary?keywords=airs
  - https://disc.gsfc.nasa.gov/data-access#mac_linux_wget
- Granule maps: https://airsl1.gesdisc.eosdis.nasa.gov/data/Aqua_AIRS_Level1/AIRXAMAP.005/
- https://docserver.gesdisc.eosdis.nasa.gov/repository/Mission/AIRS/3.3_ScienceDataProductDocumentation/3.3.4_ProductGenerationAlgorithms/README.AIRS_V6.pdf
- https://eospso.gsfc.nasa.gov/sites/default/files/atbd/20070301_L2_ATBD_signed.pdf
- https://docserver.gesdisc.eosdis.nasa.gov/repository/Mission/AIRS/3.3_ScienceDataProductDocumentation/3.3.4_ProductGenerationAlgorithms/V6_L2_Product_User_Guide.pdf

## Appendix

### IDL0 coordinates

### Pressure levels










