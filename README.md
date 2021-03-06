# crest
Classification of RelationShip Types

## Introduction
CREST (**C**lassification of **R**elation**S**hip **T**ypes) is a tool that uses identity-by-descent (IBD) segments to classify second-degree relatives as avuncular, half-siblings, or grandparent/grandchild.

## Quick Start
Follow these steps to get CREST results quickly and easily. All file names and directories in brackets should be replaced with names and directories of your choosing.
### Your Data
CREST, and IBIS, if you choose to use it, requires genotype data in a PLINK binary file format. Note that CREST currently uses autosomal IBD only, so if you have non-autosomal data, be sure to exclude it later on.
### Getting IBD Segments
We recommend using [IBIS](https://github.com/williamslab/ibis) to extract IBD information. It tends to infer contiguous segments, which is especially important for sex inference. 
Before running IBIS, we advise adding a genetic map to your .bim file. See the IBIS documentation [here](https://github.com/williamslab/ibis#Steps-for-running-IBIS):
```
./add-map-plink.pl [your data].bim [map directory]/genetic_map_GRCh37_chr{1..22}.txt > [your new data].bim
```

Then run IBIS itself:
```
ibis [your data].bed [your new data].bim [your data].fam -f [your IBIS data]
```
or if you rename the new .bim, you can supply all three at once:
```
ibis -b [your data] -f [your IBIS data]
```

...


For sex-inference, you will need to convert sex-specific genetic maps of your choosing to a .simmap format file.
Information on how to do this can be found [here](https://github.com/williamslab/ped-sim#map-file):
```
bash
wget https://github.com/cbherer/Bherer_etal_SexualDimorphismRecombination/raw/master/Refined_genetic_map_b37.tar.gz
tar xvzf Refined_genetic_map_b37.tar.gz
printf "#chr\tpos\tmale_cM\tfemale_cM\n" > [your map].simmap
for chr in {1..22}; do
  paste Refined_genetic_map_b37/male_chr$chr.txt Refined_genetic_map_b37/female_chr$chr.txt \
    | awk -v OFS="\t" 'NR > 1 && $2 == $6 {print $1,$2,$4,$8}' \
    | sed 's/^chr//' >> [your map].simmap;
done
```
Now you are ready to run the sex-inference script:
```
CREST_sex_inference.py -i [your IBIS data].seg -m [your map].simmap -b [your (new) data].bim -o [sex inference output]
```
## Thorough Start
### Pre-CREST Data Generation and Curation
### Relationship Type Inference
### Parental Sex Inference
#### Command line arguments:

* `-i` or `--input`: name of the input file (required)

* `-o` or `--output`: name of the output file (required)

* `-m` or `--map`: name of a genetic map in .simmap format (required)

* `-b` or `--bim`: name of the .bim file from your IBD data (required)

* `-w` or `--window`: window size in kilobases (optional, integer, defaults to 500 kb)
    * Windows are symmetric about the IBD segment ends, so a 500 kb window extends 250 kb in both directions.

* `-k` or `--keep`: list of sample pairs to keep for sex-inference analysis (optional, defaults to `None`)


#### Output file format
The format of the output file is
```
id1 id2 segment_number GP_lod HS_lod
```
The last two columns are quasi-LOD scores under the HS and GP models, where LOD = log10 (p(maternal) / p(paternal)). In other words,  positive scores indicate a pair is more probably maternal, while negative scores indicate a pair is more probably paternal.
