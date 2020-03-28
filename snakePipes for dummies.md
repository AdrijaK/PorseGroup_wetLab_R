# snakePipes 2.0.1 setup 
## for GRCm38, gencode M24 basic annotation, using mm10-blacklist.v2


create conda environment
```bash
conda create --yes -n snakePipes -c mpi-ie -c bioconda  -c conda-forge snakePipes
```
activate the newly created environment
```bash
conda activate snakePipes
```

add conda environment to snakePipes configuration
```bash
snakePipes config --snakemakeOptions " --use-conda --conda-prefix /localhome/inst/usr123/analysis/software/anaconda3/envs/snakePipes/"
```

get list of chromosome names to ignore for normalisation (I will ignore sex chromosomes, non-canonical scaffolds and mtDNA)
```bash
wget -qO-  ftp://ftp.ncbi.nlm.nih.gov/genomes/genbank/vertebrate_mammalian/Mus_musculus/all_assembly_versions/GCA_000001635.8_GRCm38.p6/GCA_000001635.8_GRCm38.p6_assembly_report.txt | awk '{print $NF}' | grep 'random\|chrUn\|chrM\|chrX\|chrY' > ignoreForNormalization.txt
```

(for manipulating blacklist you need bedtools. If you havent installed bedtools, install them via conda:)
```bash
conda create --yes -n bedtools -c bioconda bedtools
conda activate bedtools
```

get a list of chromosome sizes for blacklist expansion with slopBed
```bash
wget -qO- ftp://ftp.ncbi.nlm.nih.gov/genomes/genbank/vertebrate_mammalian/Mus_musculus/all_assembly_versions/GCA_000001635.8_GRCm38.p6/GCA_000001635.8_GRCm38.p6_assembly_report.txt | grep -v '^#' |  awk -v RS='\r\n' '{print $10"\t"$9}' | grep -v '^na'  > GRCm38.genome
```
>`wget -qO-` will read the file from url and print it to stdout instead of a file
`grep -v '^#' ` ignores the lines that starts with a comment(`#`)
`awk -v RS='\r\n' '{print $10"\t"$9}'` prints 10th column (chromosome name) and 9th column (chromosome size)
`grep -v '^na'` drops all chromosome/scaffold names that contain missing values (`na`)

download the blacklist from https://github.com/Boyle-Lab/Blacklist/tree/master/lists
edit the regions so they dont overlap with each other and are expanded by 50 bp both ways
```bash
wget -qO- https://github.com/Boyle-Lab/Blacklist/blob/master/lists/mm10-blacklist.v2.bed.gz?raw=true | gunzip -c | sort -k1,1 -k2,2n | slopBed -i stdin -g GRCm38.genome -b 50 > mm10-blacklist.v2.bed
```

>`wget -qO-` will read the file from url and print it to stdout instead of a file
`gunzip -c` will unzip the file
`sort -k1,1 -k2,2n` sort bed file by region before using bedtools
`slopBed -i stdin -g GRCm38.genome -b 50` expands each blacklist peak by 50 bases, but takes into account chromosome sizes from `GRCm38.genome`

activate snakePipes again to continue snakePiping
```bash
conda activate snakePipes
```

create snakePipe indices with **basic** GENCODE annotation (change this to something else if you prefer):
```bash
createIndices \
--genomeURL ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M24/GRCm38.primary_assembly.genome.fa.gz \
--gtfURL ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M24/gencode.vM24.basic.annotation.gtf.gz \
--blacklist mm10-blacklist.v2.bed \
--ignoreForNormalization ignoreForNormalization.txt \
--local \
-o /scratch/genomes/snakepipes/GRCm38_M24_basic GRCm38_M24_basic
```

make a directory for storing the temporary files
```bash
mkdir /localhome/inst/usr123/analysis/snakepipes_tmp/
```

find where the defaults.yaml is located by entering
```bash
snakePipes info
```

open the `defaults.yaml` with any test editor. 
change `tmpDir: /data/extended/` to `tmpDir: /localhome/inst/usr123/analysis/snakepipes_tmp/` or whatever your chosen tmpDir is. 

Tada!
