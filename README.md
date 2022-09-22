# Zebra_fish-RNAseq
## 1. Purpose
+ Zebra fish에 1,4-Naphthoquinone을 처리하고, Differentially Expressed Genes (DEGs)를 확인한다.

## 2. Data Collecting
+ 충남대학교 생명시스템 과학대학 김철희 교수님 연구실 Ph.d student PUSPANJALI SWAIN의 RNA데이터를 [Marcrogen, Inc.]에서 시퀀싱한 데이터를 사용했다.
+ 총 데이터의 sample은 2개로, sample 1은 Control로, 10hpf, bud stage의 untreated embryo에서 추출한 RNA이고, sample 2는 0.5uM의 1-4-Naphthoquinone을 6hpf에 처리한 후, 10hpf, bud stage의 RNA이다.

|Sample|Category|Treat|Extract point|
|-|-|-|-|
|Sample1|Control|NA|10hpf (bud stage)|
|Sample2|Treated|1-4-Naphthoquinone 0.5uM at 6hpf|10hpf (bud stage)|

+ NCBI의 [RefSeq genomes FTP](https://ftp.ncbi.nlm.nih.gov/genomes/refseq/)에서 GCF_000002035.6_GRCz11에 대한 모든 정보를 다운 받고, genomic.gff파일을 주로 사용했다.
+ GCF_000002035.6_GRCz11의 gene 개수는 47,394이고, gene_biotype이 protein_coding인 gene은 32,730개이다. 

## 3. TrimGalore
+ TrimGalore 0.6.6 version을 이용해, adapter trimming을 진행하였다.

||Total Read|Filtered Read|Total basepairs|Filtered basepairs|
|-|-|-|-|-|
|sample1_1|36,675,862|36,675,862|3,704,262,062|3,679,353,873(99.3%)|
|sample1_2|36,675,862|36,675,862|3,704,262,062|3,672,750,272(99.1%)|
|sample2_1|37,414,748|37,414,748|3,778,889,548|3,752,045,586(99.3%)|
|sample2_2|37,414,748|37,414,748|3,778,889,548|3,745,527,653(99.1%)|

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
+ 화학적 처리를 한 Sample의 repeat이 없어서 edgeR 패키지를 통한 통계 분석이 불가능해, R 및 Python의 코드를 통해 대략적인 분포를 나타냈다. 
+ R 코드와 Python 코드를 통해, bam 파일로 부터 각 유전자의 발현량, RPKM, LogFC값을 표기하는 [sorted_filtered_rpkm_foldchange.tsv](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/sorted_filtered_rpkm_foldchange.tsv) 파일을 만들었다. Protein coding gene을 제외한, tRNA,rRNA,miRNA,lncRNA 등의 gene은 모두 제거되었다.

||Mean|Min|Max|Median|1st Qu.|3rd Qu.|
|-|-|-|-|-|-|-|
|Sample1|2,222|1|598,059|440|83|1,813|
|Sample2|2,193|1|489,505|460|94|1,808|
|Sample1_RPKM|25.92|0.003|16,634|3.78|0.86|13.86|
|Sample2_RPKM|26.51|0.001|18,169|3.97|0.96|14.10|

## 6. Data Statistics
+ R을 이용한 통계를 내었다. 

![plot_zoom_png](https://user-images.githubusercontent.com/97942772/191666472-dec46c5f-510f-4efa-b40a-48e6df9eb2b3.png)

   + Sample1의 RPKM값을 X축 Sample2의 RPKM값을 Y축으로 설정하고, Scatter plot을 통해 나타냈다. Sample 1,2 모두 RPCM의 최소값은 10^-3의 값이고, 최대값은 10^4의 값으로, 데이터의 범위는 큰 반면, median은 sample1, sample2 각각 3.78, 3.97으로 낮았다. 이에 3rd Quater값인 14.10와 가까운 15까지 나타냈다.

![plot (1)](https://user-images.githubusercontent.com/97942772/191673140-fd88bfda-8c53-4901-8f80-edec5e3739a8.png)

   + Sample1, Sampe2의 RPKM값의 로그 값을 취한 그래프이다.

![plot](https://user-images.githubusercontent.com/97942772/191673215-769d81a5-7791-42e5-8b4b-cfab80eb65d6.png)

   + MA plot으로, X축에 Sample 1과 2의 평균값의 로그(Log10), Y축에 LogFC(Log2)으로 설정했다. 

