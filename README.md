# AfterQC
Automatic Filtering, Trimming, Error Removing and Quality Control for fastq data   
`AfterQC` can simply go through all fastq files in a folder and then output three folders: <b>good</b>, <b>bad</b> and <b>QC</b> folders, which contains good reads, bad reads and the QC results of each fastq file/pair.   
Currently it supports processing data from HiSeq 2000/2500/3000/4000, Nextseq 500/550, MiniSeq...and other [Illumina 1.8 or newer formats](http://support.illumina.com/help/SequencingAnalysisWorkflow/Content/Vault/Informatics/Sequencing_Analysis/CASAVA/swSEQ_mCA_FASTQFiles.htm)   

# Features:
`AfterQC` does following tasks automatically:  
* Filters reads with too low quality, too short length or too many N
* Filters reads with abnormal PolyA/PolyT/PolyC/PolyG sequences
* Does per-base quality control and plots the figures
* Trims reads at front and tail, according to QC results
* For pair-end sequencing data, `AfterQC` automatically corrects low quality wrong bases in overlapped area of read1/read2
* Detects and eliminates bubble artifact caused by sequencer due to fluid dynamics issues
* Single molecule barcode sequencing support: if all reads have a single molecule barcode (see duplex sequencing), `AfterQC` shifts the barcodes from the reads to the fastq query names
* Support both single-end sequencing and pair-end sequencing data

# Get AfterQC
* latest: `git clone https://github.com/OpenGene/AfterQC.git`
* stable: [0.4.0 (released on 2016-05-17)](https://github.com/OpenGene/AfterQC/archive/Release-0.4.0.zip)

# Dependency:
`AfterQC` uses `editdistance` module, run following before using `AfterQC`:
```shell
pip install editdistance
```
***WARNING: If you haven't installed `editdistance` module, `AfterQC` will use a python implementation of editdistance, but it will be extremely slow.***

# Simple usage:
* Prepare your fastq files in a folder
* For single-end sequencing, the filenames in the folder should be `*R1*`, otherwise you should specify `--read1_flag`
* For pair-end sequencing, the filenames in the folder should be `*R1*` and `*R2*`, otherwise you should specify `--read1_flag` and `--read2_flag`
```shell
cd /path/to/fastq/folder
python path/to/AfterQC/after.py
```
* two folders will be automatically generated, a folder 'good' stores the good reads and a folder 'bad' stores the bad reads
* `AfterQC` will print some statistical information after it is done, such how many good reads, how many bad reads, and how many reads are corrected.
* if you want to run `AfterQC` only with a single file/pair:
```shell
# with a single file
python after.py -1 R1.fq

# with a single pair
python after.py -1 R1.fq -2 R2.fq
```

# Quality Control only
If you only want to get quality control statistics, run:  
```shell
python after.py --qc_only
```

# Full options:
***Common options***
```shell
  --version             show program's version number and exit
  -h, --help            show this help message and exit
```
***File (name) options***
```

  -1 READ1_FILE, --read1_file=READ1_FILE
                        file name of read1, required. If input_dir is
                        specified, then this arg is ignored.
  -2 READ2_FILE, --read2_file=READ2_FILE
                        file name of read2, if paired. If input_dir is
                        specified, then this arg is ignored.
  -7 INDEX1_FILE, --index1_file=INDEX1_FILE
                        file name of 7' index. If input_dir is specified, then
                        this arg is ignored.
  -5 INDEX2_FILE, --index2_file=INDEX2_FILE
                        file name of 5' index. If input_dir is specified, then
                        this arg is ignored.
  -d INPUT_DIR, --input_dir=INPUT_DIR
                        the input dir to process automatically. If read1_file
                        are input_dir are not specified, then current dir (.)
                        is specified to input_dir
  -g GOOD_OUTPUT_FOLDER, --good_output_folder=GOOD_OUTPUT_FOLDER
                        the folder to store good reads, by default it is the
                        same folder contains read1
  -b BAD_OUTPUT_FOLDER, --bad_output_folder=BAD_OUTPUT_FOLDER
                        the folder to store bad reads, by default it is same
                        as good_output_folder
  --read1_flag=READ1_FLAG
                        specify the name flag of read1, default is R1, which
                        means a file with name *R1* is read1 file
  --read2_flag=READ2_FLAG
                        specify the name flag of read2, default is R2, which
                        means a file with name *R2* is read2 file
  --index1_flag=INDEX1_FLAG
                        specify the name flag of index1, default is I1,
                        which means a file with name *I1* is index2 file
  --index2_flag=INDEX2_FLAG
                        specify the name flag of index2, default is I2,
                        which means a file with name *I2* is index2 file
```
***Filter options***
```
  -f TRIM_FRONT, --trim_front=TRIM_FRONT
                        number of bases to be trimmed in the head of read. -1
                        means auto detect
  -t TRIM_TAIL, --trim_tail=TRIM_TAIL
                        number of bases to be trimmed in the tail of read. -1
                        means auto detect
  --trim_pair_same=TRIM_PAIR_SAME
                        use same trimming configuration for read1 and read2 to
                        keep their sequence length identical, default is true
                        lots of dedup algorithms require this feature
  -q QUALIFIED_QUALITY_PHRED, --qualified_quality_phred=QUALIFIED_QUALITY_PHRED
                        the quality value that a base is qualifyed. Default 20
                        means base quality >=Q20 is qualified.
  -u UNQUALIFIED_BASE_LIMIT, --unqualified_base_limit=UNQUALIFIED_BASE_LIMIT
                        if exists more than unqualified_base_limit bases that
                        quality is lower than qualified quality, then this
                        read/pair is bad. Default 0 means do not filter reads
                        by low quality base count
  -p POLY_SIZE_LIMIT, --poly_size_limit=POLY_SIZE_LIMIT
                        if exists one polyX(polyG means GGGGGGGGG...), and its
                        length is >= poly_size_limit, then this read/pair is
                        bad. Default is 35
  -a ALLOW_MISMATCH_IN_POLY, --allow_mismatch_in_poly=ALLOW_MISMATCH_IN_POLY
                        the count of allowed mismatches when evaluating
                        poly_X. Default 5 means disallow any mismatches
  -n N_BASE_LIMIT, --n_base_limit=N_BASE_LIMIT
                        if exists more than maxn bases have N, then this
                        read/pair is bad. Default is 5
  -s SEQ_LEN_REQ, --seq_len_req=SEQ_LEN_REQ
                        if the trimmed read is shorter than seq_len_req, then
                        this read/pair is bad. Default is 35
```
***Debubble options***   
If you want to eliminate bubble artifact, turn debubble option on (this is slow, usually you don't need to do this): 
```
  --debubble            enable debubble algorithm to remove the
                        reads in the bubbles. Default is False
  --debubble_dir=DEBUBBLE_DIR
                        specify the folder to store output of debubble
                        algorithm, default is debubble
  --draw=DRAW           specify whether draw the pictures or not, when use
                        debubble or QC. Default is on
```
***Barcoded sequencing options***
```
  --barcode=BARCODE     specify whether deal with barcode sequencing files, default is on
  --barcode_length=BARCODE_LENGTH
                        specify the designed length of barcode
  --barcode_flag=BARCODE_FLAG
                        specify the name flag of a barcoded file, default is
                        barcode, which means a file with name *barcode* is a
                        barcoded file
  --barcode=BARCODE     specify whether deal with barcode sequencing files,
                        default is on, which means all files with barcode_flag
                        in filename will be treated as barcode sequencing
                        files
```
***QC options***
```shell
  --qc_only             enable this option, only QC result will be output, this
                        can be much faster
  --qc_sample=QC_SAMPLE
                        sample up to qc_sample when do QC, default is 1000,000
  --qc_kmer=QC_KMER     specify the kmer length for KMER statistics for QC,
                        default is 8
```
                        
# Understand the report
* `AfterQC` will generate a QC folder, which contains lots of figures. 
* For pair-end sequencing data, both read1 and read2 figures will be in the same folder with the folder name of read1's filename. `R1` means `read1`, `R2` means `read2`.
* For single-end sequencing data, it will still have `R1`.
* `prefilter` means `before filtering`, `postfilter` means `after filtering`
* For pair-end sequencing data, `After` will do an `overlap analysis`. read1 and read2 will be overlapped when `read1_length + read2_length > DNA_template_length`. 

### Filtering statistics
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/filter-stat.png)

### Overlapped length distribution
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/overlap.png)

### Per-cycle quality
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/R1-prefilter-quality.png)

### Per-cycle base content
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/R1-prefilter-content.png)

### GC content ratio distribution
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/R1-prefilter-gc-curve.png)

### Per-cycle discontinuity
`discontinuity` means the summution of the difference neighbor cycles, in a KMER window. For example, `discontinuity` of `AAAAA` is `0`, `discontinuity` of `ATCGA` is `4`. This value reflects the `signal-noise-ratio` of each sequencing cycle. Idealy it should be nearly a hrizontal straight line.
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/R1-prefilter-discontinuity.png)

### KMER-based strand bias
Theoretically in a sequencing run, the repeat count of a KMER should be close to the count of this KMER's reverse complement. If these two counts are not close, it reflects there is amplificaion bias (like PCR bias) or sequencing bias. If the bias is very low, the points of `(KMER-count, KMER-reverse-complement-count)` should be near the line `y=x`. The compacter the points are, the lower the bias is.  
![image](https://github.com/OpenGene/AfterQC/raw/master/report_sample/R1-prefilter-strand-bias.png)
