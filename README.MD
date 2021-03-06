AA_stat
========
An open source software for amino acid residue modification analyses in proteomics

	
		
## Overview

**AA_stat** is a tool for shotgun proteomics that uncovers the unexpected modifications of amino acid residues in the protein sequences, as well as possible artefacts of data acquisition or processing, in the results of proteome analysis. 
AA_stat calculates and visualizes amino acid occurrence frequencies for the identified peptides. AA_stat processes the results of open search and composes a list of identified mass shifts that can be attributed to modifications. The processing involves Gaussian fitting of potential peaks, group-specific FDR filtering, amino acid counting, and appearance frequency normalization for the particular mass shifts.
The algorithm was implemented in Python as an open-source command-line tool available at [https://github.com/SimpleNumber/aa_stat](https://github.com/SimpleNumber/aa_stat)

Project moved from [https://bitbucket.org/J_Bale/aa_stat/](https://bitbucket.org/J_Bale/aa_stat/)

## Installation

Download from Github repository [https://github.com/SimpleNumber/aa_stat](https://github.com/SimpleNumber/aa_stat)
In the directory containing **setup.py** file run the following command:
```
pip install .
```

## Usage

```
AA_stat [-h] --params PARAMS [--dir DIR] [-v] (--pepxml PEPXML [PEPXML ...] | --csv CSV [CSV ...])

optional arguments:
  -h, --help                    Show this help message and exit
  --params PARAMS               CFG file with parameters.
  --dir DIR                     Directory to store the results. Default value is current directory.
  -v, --verbosity               Increase output verbosity
  --pepxml PEPXML [PEPXML ...]  List of input files in pepXML format
  --csv CSV [CSV ...]           List of input files in CSV format
```

### Configuration file


The AA_stat tool needs a **config** file and at least one **data** file with open search results.

The example of **config** file can be found at the BitBucket repository (**example.cfg**).

This file has following parameters:


| Name                             | Default value                            | Description                               |
| ---------------------------------|------------------------------------------|-------------------------------------------|
|                                  |**[data]**                                |                                           |
| decoy prefix                     | DECOY_                                   | Prefix that is used to indicate decoy sequences in database. |
| FDR                              | 0.02                                     | PSM false discovery rate, that will be used for each mass shift interval. |
| labels                           | M D Q C L H S R A W Y P F V N T E G I K  | All amino acid residues. It may be helpful if your data contains unusual amino acid residues. |
| cleavage rule                    | trypsin        | Most common rules taken from pyteomics.parser.expasy_rules at [https://pyteomics.readthedocs.io/en/latest/api/parser.html](https://pyteomics.readthedocs.io/en/latest/api/parser.html) Also, a cleavage rule can be a Python regex. |
|                                  |**[csv input]**     |                              |
| delimiter                        | , (comma)      | Delimiter used in CSV input files. |
| proteins column                  | protein      | Name of column with protein IDs. |
| proteins delimiter               | ;         | The delimiter of proteins in proteins’ column. |
| peptides column                  | peptide  | Name of column with peptide sequences. |
|                                  |**[general]** |                                 |
| width of bin in histogram        |0.001 | Bin width in Da that will be used to make mass shift distribution histogram. |
| open search range                | -500, 500 | Open search range, in Da. |
| shifting window                  |0.03 | Mass window, in Da, that will be used for Gaussian fit. At least one mass shift peak should be in this window. |
| threshold for bins               | 200 | The minimal number of PSMs that should be in the interval to start Gaussian fitting. |
| FDR correction                   | True | Use FDR correction when filtering each mass interval. |
| use specific mass shift window   | no | Focusing on specific mass shift (yes/no). |
| specific mass shift window       | 15.975, 16.005 |Specifying mass range, in Da, which the User wants to focus on. |
| figure size in inches            | 5.0, 3.5 | Specifying the size of the output figures, in inches (L,H). |
|                                  | **[fit]** |                                |
| standard deviation threshold for center of peak | 15 | Threshold value for the standard error of peak center as determined by the Gaussian fit algorithm. This value is expressed in histogram bins (the bin width is configured in **[general]**). |
| standard deviation threshold for sigma | 0.1 | Threshold value for the standard error of sigma, relative to sigma, as determined by the Gaussian fit algorithm. |
| standard deviation threshold for height | 0.15 | Threshold value for the standard error of peak height, relative to peak height, as determined by the Gaussian fit algorithm. |
| shift error                      | 10 | Minimal distance between fitted Gaussian peaks (in histogram bins). Closer peaks will be merged. |
|                                  |**[localization]**|                          |
| spectrum column                  | spectrum      | Name of column with spectrum IDs |
| charge column               | assumed_charge         | Name of column with assumed charges |

### Open search results


AA_stat deals with open search results in pepXML or CSV formats. In **config** file, specify names of CSV columns that AA_stat will further use.

AA_stat is compatible with the search results obtained using most existing search engines. By default, it is recommended to employ MSFragger search engine, available from [Nesvizhskii lab](http://www.nesvilab.org/software.html) website. For details of its operation, see [MSFragger User manual](https://media.nature.com/original/nature-assets/nmeth/journal/v14/n5/extref/nmeth.4256-S2.pdf). 

### Examples

An example of the open search parameters file can be found in the Github [repository](https://github.com/SimpleNumber/aa_stat) of AA_stat. 

Example of MSFragger usage:

```
java -Xmx8G MSFragger.jar open_search.params HeLa_run1.mzML HeLa_run2.mzML 
```


### Output files

AA_stat produces the following files:

**A.** Gaussian fit report (gauss_fit.pdf).

**B.** Summary histogram (summary.png). 

**C.** Charts (PNG and SVG files) of normalized frequencies for each significant mass shift interval. 

**D.** Summary table (aa_statistics_table.csv) of amino acid frequencies for all mass shifts with Unimod.org links for possible modifications.

**E.** Summary table (p_values.csv) with p-values for each amino acid frequencies in each mass shift.

**F.** HTML file (report.html) aggregates and illustrates all results.


**A.** Gaussian fit file shows PSM distributions in intervals that were considered as mass shift peaks. Subplot titles correspond to mass shifts (interval center). Peaks are classified into 3 groups: PASSED - mass shift with good fit, which are considered for  subsequent analysis; NO FIT - mass shifts for which the algorithm could not find Gaussian function; FAILED - mass shift with a fit not passing the configured filtering criteria.

|![img1](manual_pic/gauss.png )|
|------------------------------|
| **Figure 1.** Examples of Gaussian fit results in gauss_fit.pdf.|
	
**B.** Summary histogram shows numbers of filtered PSMs in all mass shift intervals. Number on top of the bin indicates the percentage of all identified PSMs. Each mass shift interval is filtered separately to the user-specified FDR level, using target-decoy approach.

|![img2](manual_pic/summary.png)|
|-------------------------------|
| **Figure 2.** Example of Summary histogram.|

**C.** Charts of normalized frequencies for each significant mass shift. Each chart is named according to the mass shift. Each bar in the chart denotes the normalized occurrence frequency of a specific amino acid residue in the given mass shift interval. The normalized frequency is calculated by:
	
	1) counting all amino acids in all non-redundant peptides identified with the given mass shifts;
	
	2) dividing the count for the given residue by the total amino acid count for the interval to obtain the occurrence frequency of the residue;
	
	3) normalizing the occurrence frequency of the residue by the occurrence residue of the same residue for the zero mass shift interval.

	If the normalized frequency of a residue significantly exceeds one, that means that this residue is “enriched” in the peptides identified with the corresponding mass shift, suggesting that there is a connection between this residue and the cause of the mass shift. In the simplest case, this residue is modified:
	

|![img3](manual_pic/ms.png)|
|--------------------------|
| **Figure 3.** Example of a normalized frequency chart for 15.991 mass shift.|

**D.** Summary table (aa_statistics_table.csv) of amino acid frequencies for all mass shifts with PSMs counts and Unimod.org links for possible modifications. 

**E.** Summary table (p_values.csv) with p-values for all amino acid frequencies in all mass shifts that indicates the significant deviation amino acid frequency from zero mass shift peak.

**F.** HTML file (report.html) aggregates and illustrates all results.
