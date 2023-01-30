# Calcifer: A workflow for circRNA detection and analysis
Author: Andre Brezski, Kathi Zarnack    

---

Important:   
At the moment Calcifer only works with a conda environment created by the environment.yml-file.   
A non-compiled version will be available with the full release.

# 1. Introduction
Calcifer is a workflow for highly automated detection and analysis of circRNAs in RNA-Seq datasets. It allows the evaluation of RNA-Seq read data up to a list of characterized circRNA isoforms, as well as the prediction of possible functions.

# 2. Overview

![Calcifer workflow](/assets/images/calcifer_new_changed_workflow.png)

# 3. CircRNA detection
In a first step the RNA-seq data is aligned with STAR [1] and bwa [2] against the reference genome. The resulting bam-files are used as input for CIRCexplorer2 [3] and CIRI2 [4] respectively. These both tools yield an unfiltered list of putative circRNAs, which is further processed.

# 4. CircRNA filtering
The raw circRNA results from both tools are then further filtered for canonical splice sites, length of the circRNA, encompassing junctions and uniquely mapped back-splice junction reads. At the end only circRNAs which have at least 2 uniquely back-splice junction reads are considered for further analysis.

# 5. Downstream analysis
Based on the filtered circRNA list there is a broad downstream analysis. To shed light on putative biogenesis and function the downstream analysis consists out of detection of putative miRNA binding, RBP binding and open reading frames. 

#### 5.1. Linear and circular count data
In general a count matrix for linear and circular mapped reads is created. If needed these can be utilized by the user for differential expression analysis for multiple condition datasets. A rmd-file for a baseline DESeq2 analysis is included in the major workflow folder.

#### 5.2. miRNA binding site detection
MiRNA binding sites are detected by miRanda [5] on the circular exonic sequence for each circRNA. To enable miRNA binding analysis over the back-splice junction sequence, the linear sequence is extended by 25 bp from the opposite end respectively.

#### 5.3. RBP binding site detection
The RBP binding prediction is performed with FIMO [6] on the same sequence (back-splice junction extended linear exon sequence) and additional on the not included sequence around the back-splice junction. CircRNA biogenesis can be enabled by RBP binding close to the back-splice junction. Also a putative function of circRNAs is the direct binding of RBPs.

#### 5.4. ORF prediction
The ORF prediction is performed on the linear- as well as the pseudo-circular and multi-cycle exonic circRNA sequence. These enables the prediction of longer ORFs, which span over the back-splice junction as well as multiple reading frames.

---

# Example output
CircRNA isoform: 
> chr21:29321220-29329693:+	

Parental gene:
> BACH1	

CircRNA type:
> exon	

Unique back-splice junction reads:
> 21

MiRNA binding site density [binding sites/bp]:
> 0.09532	

Most bound single miRNA (name:binding sites count:proportion to all miRNA binding sites):
> hsa-miR-548am-3p:2:0.01143	

Most binding RBP on circRNA sequence (linear sequence with 25 bp added on both sites to simulate circularization):
> RNCMPT00043:1	

Most binding RBP around the back-splice junction (250 bp up- and downstream of the back-splice junction and 25 bp into the circRNA on both sites):
> no_rbp_binding		

linear ORF:
> complete:597	

pseudo circular ORF:
> complete:597	

multi cycle ORF:
> complete:597	

Uniqueness of putative ORF peptide:
> non_unique	

Circular-to-linear ratio:
> 0.28852
 

---

# Citing Calcifer

> Manuscript in preparation

---

# Usage
Calcifer can be executed from the command line. The dependencies should be installed beforehand in a suitable environment (e.g. miniconda).

Calcifer contains several run modes which can be used to detect and analyse circRNAs.

"calcifer circexplorer2":   
Is the run mode to detect circRNAs with CE2 and filter them for >1 bsj-read. A star index is needed for the mode as well as a gene_pred file (which can be generated by ucsc gtfToGenePred).

"calcifer ciri2":   
Is the run mode to detect circRNAs with ciri2 (these are prefiltered for >1 bsj-reads).

"calcifer downstream":   
Takes the results from the ce2 and ciri2 mode as input (in addition to other inputs). The detected circRNAs are analysed regarding read counts, miRNA binding, RBP binding and ORF as well as resulting unique peptides.

"calcifer fullrun":   
Combines all other modes into one workflow, this is the default mode for calcifer.


A suitable conda environment can be installed with the included environment.yml-file with "conda env create -f environment.yml".   
CIRI2 needs to be installed manually.    
The path to CIRI2 needs to be included as parameters in the respective modes.   

The used gtf-file needs to be from the actual ensemble release.


Usage:


calcifer circexplorer2:   
"calcifer.py circexplorer2 -path [path] -data [name] -star [index] -genome [fasta] -gene_pred [txt] -rt [se/pe]"  
 
>-path		Path to the general working directory with the raw read files   

>-data		Names of the datasets which are analyzed (if >1 then separate with a comma)   

>-star		Path to the folder with a STAR index, the index needs to be generated before using Calcifer   

>-genome		Path to the reference genome in fasta-format   

>-gene_pred		Path to a gene prediction file, needes for CE2, can be generated using ucsc gtfToGenePred   

>-rt			Read type of the input files (se [single-end] or pe [paired-end])   



calcifer ciri2:   
"calcifer.py ciri2 -path [path] -data [name] -bwa [index] -genome [fasta] -ref [gtf] -cpath [path] -rt [se/pe]"  
 
>-path		Path to the general working directory with the raw read files   

>-data		Names of the datasets which are analyzed (if >1 then separate with a comma)   

>-bwa			Path to the folder with a bwa index, the index needs to be generated before using Calcifer   

>-genome		Path to the reference genome in fasta-format   

>-ref			Path to the ensemble gtf-file   

>-cpath		Path to ciri2.pl (the CIRI2 installation), the installation needs to be done manually as there is no conda version  
 
>-rt			Read type of the input files (se [single-end] or pe [paired-end])   



calcifer downstream:   
"calcifer.py -path [path] -data [name] -con [list] -con_names [list] -genome [fasta] -rt [se/pe] -gtf [gtf] -mirna [path] -pep [path] -rbp [path]"   

>-path		Path to the general working directory with the raw read files   

>-data		Names of the datasets which are analyzed (if >1 then separate with a comma)   

>-con			List with count of datasets for each condition   

>-con_names		Names of the condition in the same order as the count from -con   

>-genome		Path to the reference genome in fasta-format   

>-rt			Read type of the input files (se [single-end] or pe [paired-end])   

>-gtf			Path to the ensemble gtf-file   

>-mirna		Path to a miRNA database [can be downloaded from miRBase.org]   

>-pep			Path to a peptide fasta file [can be downloaded from the actual ensembl release]   

>-rbp			Path to a RBP database [can be downloaded from the MEME motif database, Ray et al. 2013]   
 


calcifer full_run:   
"calcifer full_run -path [path] -data [name] -star [index] -genome [fasta] -gene_pred [txt] -rt [se/pe] -gtf [gtf] -con [list] -con_names [list] -cpath [path] -bwa [index] -mirna [path] -pep [path] -rbp [path]" 
  
>-path		Path to the general working directory with the raw read files   

>-data		Names of the datasets which are analyzed (if >1 then separate with a comma)   

>-star		Path to the folder with a STAR index, the index needs to be generated before using Calcifer   

>-genome		Path to the reference genome in fasta-format   

>-gene_pred		Path to a gene prediction file, needes for CE2, can be generated using ucsc gtfToGenePred   

>-rt			Read type of the input files (se [single-end] or pe [paired-end])   

>-bwa			Path to the folder with a bwa index, the index needs to be generated before using Calcifer   

>-cpath		Path to ciri2.pl (the CIRI2 installation), the installation needs to be done manually as there is no conda version   

>-gtf			Path to the ensemble gtf-file   

>-mirna		Path to a miRNA database [can be downloaded from miRBase.org]   

>-pep			Path to a peptide database [can be downloaded from the actual ensembl release]   

>-rbp			Path to a RBP database [can be downloaded from the MEME motif database, Ray et al. 2013]    

>-con			List with count of datasets for each condition   

>-con_names		Names of the condition in the same order as the count from -con   



Downstream/Fullrun Output:   
The main output it the calcifer_output.tab file.   
The file contains an overview of all detected circRNAs and each of the respective downstream analyses.   
The following information is provided for each circRNA:   
>circID	parental_gene	type	unique_bsr	con_clr_header	    
>mirna_binding_site_density	most_mirna	   
>rbp_circ_binding	rbp_bsj_binding	   
>linear_seq_orf	pseudo_circular_seq_orf	multi_cycle_seq_orf	unique_region   



---

# Literature
[1] Dobin, Alexander, et al. "STAR: ultrafast universal RNA-seq aligner." Bioinformatics 29.1 (2013): 15-21.

[2] Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows–Wheeler transform. Bioinformatics, 25(14), 1754–1760.

[3] Zhang, Xiao-Ou, et al. "Diverse alternative back-splicing and alternative splicing landscape of circular RNAs." Genome research 26.9 (2016): 1277-1287.

[4] Gao, Yuan, Jinyang Zhang, and Fangqing Zhao. "Circular RNA identification based on multiple seed matching." Briefings in bioinformatics 19.5 (2018): 803-810.

[5] John, Bino, et al. "Human microRNA targets." PLoS biology 2.11 (2004): e363.

[6] Grant, Charles E., Timothy L. Bailey, and William Stafford Noble. "FIMO: scanning for occurrences of a given motif." Bioinformatics 27.7 (2011): 1017-1018.
