# Zebra_fish-RNAseq
## 1. 목적
+ Zebra fish에 1,4-Naphthoquinone을 처리하고, Differentially Expressed Genes (DEGs)를 확인한다.

## 2. 데이터 수집
+ /eevee/val/jjpark/220916_marcrogene 하위의 1_1.fastq, 1_2.fastq, 2_1.fastq, 2_2.fastq를 사용했다.
+ NCBI의 [RefSeq genomes FTP](https://ftp.ncbi.nlm.nih.gov/genomes/refseq/)에서 GCF_000002035.6_GRCz11에 대한 모든 정보를 다운 받고, genomic.gff파일을 사용했다.

## 3. TrimGalore
+ TrimGalore 0.6.6 version을 이용해, adapter trimming을 진행하였다. 

## 4. HISAT2
+ GCF_000002035.6_GRCz11.genomic.gff을 대상으로 hisat-build를 통해서 indexing을 했다. 
+ Indexing이 끝난후, trimming이 끝난 데이터를 대상으로 HISAT2를 통한 allignment를 했다. 아래는 running option.
<pre>
<code>
hisat2 --max-intronlen 50000 -p 24 -x index -1 1_1_val_1.fq -2 1_2_val_2.fq 2> sample1.log | samtools view -@ 24 -bSF4 - | samtools sort -@ 24 - -o sample1.bam
</code>
</pre>
+ 결과물을 bam파일로 저장했다. 

## 5. Data Handling
+ 화학적 처리를 한 Sample의 copy가 없어서 edgeR 패키지를 통한 분석이 불가능해, R 및 Python의 코드를 통해 대략적인 분포를 나타냈다. 
+ R 코드와 Python 코드를 통해, bam 파일로 부터 각 유전자의 발현량, RPKM, LogFC값을 표기하는 
