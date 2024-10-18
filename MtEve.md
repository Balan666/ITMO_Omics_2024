**Before the start of the work all the archive files were dowloaded and unzipped with `unzip archive.zip`, conda environment was created and all the needed tools were downloaded with bioconda.**

## Building the first tree
Let's start from some human mtDNA data
```
# unit all the fasta reads in one file
cat Human/* > human.fasta

# do the global alignment
mafft human.fasta > aligned_human.fasta

# remove uninformative alignment pieces
trimal -in aligned_human.fasta -out human_aligned_trimmed.fasta -keepheader -automated1
```
Now let's choose a substitution model. IqTree can do this autometically, however, I will do this in order to get more control over the process.
```
modeltest-ng -i human_aligned_trimmed.fasta > human_modeltest.txt
```
In the txt document we can find our results. According to one of the analysis, Bayesian information criterion (BIC), TIM2+I+G4 is the best model here:
```
BIC       model              K            lnL          score          delta    weight
--------------------------------------------------------------------------------
       1  TIM2+I+G4          8    -27910.9153     56744.7831         0.0000    0.5052
       2  TrN+I+G4           7    -27915.8904     56745.0180         0.2349    0.4492
       3  HKY+I+G4           6    -27923.4154     56750.3528         5.5696    0.0312
       4  TIM1+I+G4          8    -27914.9341     56752.8207         8.0376    0.0091
       5  TIM3+I+G4          8    -27915.6067     56754.1659         9.3828    0.0046
       6  TPM2uf+I+G4        7    -27922.4909     56758.2190        13.4358    0.0006
       7  GTR+I+G4          10    -27910.7430     56763.8690        19.0859    0.0000
       8  TPM3uf+I+G4        7    -27925.6821     56764.6014        19.8183    0.0000
       9  TVM+I+G4           9    -27920.0350     56772.7376        27.9545    0.0000
      10  TPM1uf+I+G4        7    -27931.7915     56776.8201        32.0369    0.0000
--------------------------------------------------------------------------------
Best model according to BIC
---------------------------
Model:              TIM2+I+G4
lnL:                -27910.9153
Frequencies:        0.3094 0.3143 0.1305 0.2458
Subst. Rates:       2.2963 100.0000 2.2963 1.0000 66.3532 1.0000 
Inv. sites prop:    0.8956
Gamma shape:        1.2648
Score:              56744.7831
Weight:             0.5052
---------------------------
```
Let's use this model while building the tree:
```
iqtree2 -m TIM2+I+G4 -s human_aligned_trimmed.fasta -B 1000 -alrt 1000 -redo --prefix human_iqtree
```
![The resulting tree](https://github.com/Balan666/MtEve/blob/main/pics/human_figtree.png?raw=true)
## Adding other Homo species
```
cat Human/* ./Denisova/* Neanderthal/* Pan/* > all_hum_genomes.fasta

mafft all_hum_genomes.fasta > aligned_all_hum_genomes.fasta

trimal -in aligned_all_hum_genomes.fasta -out aligned_all_hum_genomes_trimmed.fasta -keepheader -automated1
```
The best model has changed, our previous one appeared to be the second best variant. However, this time I'll use another model.
```
Best model according to BIC
---------------------------
Model:              TrN+I+G4
lnL:                -38407.0738
Frequencies:        0.3144 0.3127 0.1254 0.2475
Subst. Rates:       1.0000 54.3132 1.0000 1.0000 37.6647 1.0000 
Inv. sites prop:    0.6793
Gamma shape:        0.9029
Score:              77921.6285
Weight:             0.6729
---------------------------
```
```
iqtree2 -m TrN+I+G4 -s aligned_all_hum_genomes_trimmed.fasta -B 1000 -alrt 1000 --prefix all_human_iqtree
```
A few files were acquired and the tree was visualized in figtree
![The tree of all the sequences drawn by FigTree](https://github.com/Balan666/MtEve/blob/main/pics/all-human_figtree.png?raw=true "A pretty ugly tree")

It's pretty hard to see the branch length though, and the figtree interface is pretty restricted, so iTOL was used to make it a bit prettier:
![The tree of all the sequences drawn by iTOL](https://github.com/Balan666/MtEve/blob/main/pics/all-human_iTol.png?raw=true "A beautiful tree")

Tools like IQ-TREE allow you to estimate divergence times based on either fixed rates (where you specify a rate of mutation/changes per year) or relaxed clock models, which account for rate variability among lineages. When you run analyses using these settings, the software provides estimated ages for nodes based on the accumulated molecular data.
