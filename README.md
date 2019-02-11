# Scripts for performing bulk segregant analysis with *Brassica napus*

These scripts are split into the those actually doing the analysis and plotting (`scripts` folder) and submission scripts for running on a SLURM based HPC system (`submission_scripts`). A lot of variables are hard coded, but for each script I will try to point out which lines these assignments occur on. All of the `submission_scripts`, except `10_plotter_submit.sh`, should be run on SLURM submission nodes as is; they will generate and submit scripts to the system. `10_plotter_submit.sh` should submitted directly with the `sbatch` command. The SLURM submission scripts that are generated in all files in `submission_scripts` will have to be modified to use differently named queues (`-p` variable) and different reporting e-mail addresses (`--mail-user` variable).

## `submission_scripts/01_alignment.sh`

Uses `bowtie` and `samtools` to align paired sequencing files to the *Brassica napus* genome.

Line 3 & 45: These two lines add the `bowtie` directory to the PATH.
Line 4, 6 & 7: The raw reads were contained in two directories, which these lines specify.
Line 5 & 78: The reference genome to align to. `.fa` is appended to this reference on line 78, which may not always be the case.
Lines 16-20: Depending on what the file names of the read files are, these lines may need to be changed so unique identifiers are correctly found.
Lines 46-48: Not all HPC clusters will have this software.

## `submission_scripts/02_what_failed.sh`

Checks jobs to see if they ran correctly.

Line 8-10: The raw reads were contained in two directories, which these lines specify.
Lines 12-16: Depending on what the file names of the read files are, these lines may need to be changed so unique identifiers are correctly found.

## `submission_scripts/03_group_mpileup.sh`

Runs `mpileup` for all BAM files.

Line 3 & 30: The reference genome to align to. `.fa` is appended to this reference on line 78, which may not always be the case.

## `submission_scripts/04_run_delta_snp_calculator.sh`

Runs `pileup.py` for each of the bulk comparisons.

Line 3 & 4: The sequencing samples names of Darmor and Cabriolet, the parental lines.
Line 6: The output directory containing the pileup file to use.
Lines 33, 34, 56, 57, 79 & 80: Contain the sequencing sample names for the three different bulk populations.

## `submission_scripts/05_run_parent_difference.sh`

Runs `pileup_ref_differences.py` for the two parental lines.

Line 3 & 4: The sequencing samples names of Darmor and Cabriolet, the parental lines.
Line 6: The output directory containing the pileup file to use.

## `submission_scripts/06_get_valid_depths.sh`

Runs `pileup_valid_depth.py` for each of the bulk comparisons.

Line 3 & 4: The sequencing samples names of Darmor and Cabriolet, the parental lines.
Line 6: The output directory containing the pileup file to use.
Lines 33, 34, 56, 57, 79 & 80: Contain the sequencing sample names for the three different bulk populations.

## `submission_scripts/07_unique_reads_submit.sh`

Filters all BAM files in the child directories for reads that are uniquely mapping (have a MAPQ score of 42).

Sincere thanks to this [blog post](http://biofinysics.blogspot.com/2014/05/how-does-bowtie2-assign-mapq-scores.html).

## `submission_scripts/08_group_mpileup_unique.sh`

Runs `mpileup` for all BAM files generated by `07_unique_reads_submit.sh`.

Line 3 & 30: The reference genome to align to. `.fa` is appended to this reference on line 78, which may not always be the case.

## `submission_scripts/09_get_position_counts.sh`

Runs `pileup_snp_filtering.py` for each of the bulk comparisons.

Line 3 & 4: The sequencing samples names of Darmor and Cabriolet, the parental lines.
Line 6: The output directory containing the pileup file to use.
Lines 33, 34, 56, 57, 79 & 80: Contain the sequencing sample names for the three different bulk populations.

## `submission_scripts/10_plotter_submit.sh`

Runs `plotter.R`.

Line 12: Not all HPC clusters will have this specific version of R.

## `scripts/pileup.py`

Performs the bulk segregant analysis. The script takes five command line parameters:

    1. Output file.
    2. Column number of parent 1 in the mpileup file.
    3. Column number of parent 2 in the mpileup file.
    4. Column number of bulk 1 in the mpileup file.
    5. Column number of bulk 2 in the mpileup file.

Line 6 & 7: The pileup file to use.
Line 65: The minimum depth to use.
Line 66: The minimum SNP proportion to use.
Line 67: The minimum SNP Index to use.

Returns a file with the column headers:

    1. chromosome: The chromosome of the SNP.
    2. position: Chromosomal position of the SNP.
    3. reference_parent_allele: The allele of the 'reference' parent (parent 1).
    4. other_parent_allele: The allele of the 'other' parent (parent 2).
    5. reference_parent_depth: Number of reads with the allele from parent 1.
    6. other_parent_depth: Number of reads with the allele from parent 2.
    7. bulk1_depth: Read depth of the position in bulk 1.
    8. bulk2_depth: Read depth of the position in bulk 2.
    9. bulk1_snp_index: SNP Index of the position in bulk 1.
    10. bulk2_snp_index: SNP Index of the position in bulk 2.
    11. delta_snp_index: Delta SNP Index of the position.

## `scripts/pileup_ref_differences.py`

Reports differences between the genome reference and the reference parent, as well as differences between the reference parent and the other parent used to make the test population. In the context of this experiment, it reports differences between the published *Brassica napus* genome (Darmor-*bzh*) and the Darmor sequenced in this experiment and Darmor and Cabriolet both sequenced in this experiment. The script takes four command line parameters:

    1. Output file for the genome reference - reference parent comparison.
    2. Output file for the parental comparison.
    3. Column number of parent 1 in the mpileup file.
    4. Column number of parent 2 in the mpileup file.

Line 7 & 8: The pileup file to use.
Line 56: The minimum depth to use.
Line 57: The minimum SNP proportion to use.
Line 58: The minimum SNP Index to use.

The genome reference - reference parent comparison output has the following column headers:

    1. chromosome: The chromosome of the SNP.
    2. position: Chromosomal position of the SNP.
    3. published_reference: The allele in the reference genome.
    4. reference_parent_allele: The allele of the 'reference' parent (parent 1).
    5. reference_parent_depth: Number of reads with the allele from parent 1.
    6. type: The type of difference, either snp, deletion, insertion, or replacement.

The parental comparison output has the following column headers:

    1. chromosome: The chromosome of the SNP.
    2. position: Chromosomal position of the SNP.
    3. reference_parent_allele: The allele of the 'reference' parent (parent 1).
    4. other_parent_allele: The allele of the 'other' parent (parent 2).
    5. reference_parent_depth: Number of reads with the allele from parent 1.
    6. other_parent_depth: Number of reads with the allele from parent 2.
    7. type: The type of difference, either snp, deletion, insertion, or replacement.

## `scripts/pileup_valid_depth.py`

Outputs regions of the genome which have the specified minimum depth in both parents and both bulks. The script takes five command line parameters:

    1. Output file.
    2. Column number of parent 1 in the mpileup file.
    3. Column number of parent 2 in the mpileup file.
    4. Column number of bulk 1 in the mpileup file.
    5. Column number of bulk 2 in the mpileup file.

Line 6: The pileup file to use.
Line 64: The minimum depth to use.
Line 65: The minimum SNP proportion to use.
Line 66: The minimum SNP Index to use.

Returns a file with the column headers:

    1. chromosome: Chromosome of the region.
    2. region_no: A unique identifier for that region.
    3. start: Start position of the region.
    4. end: End position of the region.

## `scripts/pileup_snp_filtering.py`

Returns summary statistics about the number of valid genomic positions and the number of positions that are invalid for various reasons. The script takes five command line parameters:

    1. Output file.
    2. Column number of parent 1 in the mpileup file.
    3. Column number of parent 2 in the mpileup file.
    4. Column number of bulk 1 in the mpileup file.
    5. Column number of bulk 2 in the mpileup file.

Line 6 & 7: The pileup file to use.
Line 65: The minimum depth to use.
Line 66: The minimum SNP proportion to use.
Line 67: The minimum SNP Index to use.

Returns a file with the number of positions that are:

    1. valid: Meet all filtering parameters.
    2. invalid_snp_index: Either bulk 1 or bulk 2 has a SNP Index less than the minimum SNP Index parameter.
    3. parental_alleles_not_different: The parental alleles do not differ at this position, so no comparison can be made.
    4. invalid_parental_alleles: The SNP frequency of one of the parental alleles is below the minimum SNP proportion to use.
    5. invalid_depth: Either one of the parents or one of the bulks has read depth below the read depth threshold.

## `scripts/plotter.R`

Generates plots showing SNP indices across the genome. Also generates quality assurance plots, showing which parts of the genome have suitable depth to make comparisons, the distribution of parental differences across a genome, and the distributions of differences between the reference genome sequence and the reference parent.

Lines 8-30: Requires input files generated by the scripts `04_run_delta_snp_calculator.sh` (`input_folder`), `05_run_parent_difference.sh` (`dar_cab_diffs` and `dar_dar_diffs`), and `06_get_valid_depths.sh` (`depth_folder`).
Line 50: The proportion of extreme SNP indices to highlight can be changed on this line. At the moment, the value of `0.99` indicates that the 1% most extreme SNP Index values will be highlighted in the plots.
Line 77: The sizes of the SNP Index averaging window to use.

In addition to the plots the script also outputs a `gct` file, which can be imported into IGV to view the positions of the most extreme 1% of SNP indices.

## `scripts/checker.py`

Used to compare MD5 hashes of the downloaded sequencing files to the hashes reported by the sequencing company, which in this case were given in the file `MD5.txt`.

## `scripts/gtf_filterer.py`

Used for filtering GTF files. In this case, a GTF file of *Brassica napus* genes is filtered for those genes which are homologues of Arabidopsis flowering time genes.
