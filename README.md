# Zebra_fish-RNAseq
## 1. Purpose
+ Zebra fish에 1,4-Naphthoquinone을 처리하고, Differentially Expressed Genes (DEGs)를 확인한다.

## 2. Data Collecting
+ 충남대학교 생명시스템 과학대학 김철희 교수님 연구실 Ph.d student PUSPANJALI SWAIN의 RNA를 [Marcrogen, Inc.](https://www.macrogen.com/ko/main)에서 시퀀싱한 데이터를 사용했다.
+ 총 데이터의 sample은 2개로, sample 1은 Control로, 10hpf (bud stage)의 untreated embryo에서 추출한 RNA이고, sample 2는 0.5uM의 1-4-Naphthoquinone을 6hpf에 처리한 후, 10hpf (bud stage)의 RNA이다.

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
+ Indexing이 끝난후, trimming이 끝난 데이터를 대상으로 HISAT2를 통한 alignment를 했다. 아래는 running option.
<pre>
<code>
hisat2 --max-intronlen 50000 -p 24 -x index -1 1_1_val_1.fq -2 1_2_val_2.fq 2> sample1.log | samtools view -@ 24 -bSF4 - | samtools sort -@ 24 - -o sample1.bam
</code>
</pre>
+ HISAT2 alignment의 통계는 아래와 같다.


||Total reads|Non alignment reads(%)|Once aligned reads(%)|Multiple aligned reads(%)|Overall alignment rate|
|-|-|-|-|-|-|
|Sample1|36,643,192|7,470,036 (20.39%)|23,123,139 (63.10%)|6,050,017 (16.51%)|88.78%|
|Sample2|37,384,688|7,589,304 (20.30%)|23,244,199 (62.18%)|6,551,185 (17.52%)|88.20%|

## 5. Data Handling
+ 화학적 처리를 한 Sample의 repeat이 없어서 edgeR 패키지를 통한 통계 분석이 불가능해, R 및 Python의 코드를 통해 대략적인 분포를 나타냈다. 
+ R 코드와 Python 코드를 통해, bam 파일로 부터 각 유전자의 발현량, RPKM, LogFC값을 표기하는 [sorted_filtered_rpkm_foldchange.tsv](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/sorted_filtered_rpkm_foldchange.tsv) 파일을 만들었다. Protein coding gene을 제외한, tRNA,rRNA,miRNA,lncRNA 등의 gene은 모두 제거되었다. Sample 1,2 두 샘플 중 하나라도 발현량이 0인 gene 또한 제외하였다.
+ 아래 표는 sample 1,2에서의 유전자 발현량의 기본적인 통계를 나타냈다.

||Mean|Min|Max|Median|1st Qu.|3rd Qu.|
|-|-|-|-|-|-|-|
|Sample1 gene expression count|2,222|1|598,059|440|83|1,813|
|Sample2 gene expression count|2,193|1|489,505|460|94|1,808|
|Sample1 RPKM|25.92|0.003|16,634|3.78|0.86|13.86|
|Sample2 RPKM|26.51|0.001|18,169|3.97|0.96|14.10|

## 6. Data Statistics
+ R을 이용한 통계를 내었다. 
+ Strongly upregulate는 sample 2 RPKM 값이 sample 1에 비해 8배 이상 큰 경우를 나타내고, Upregulate는 4배 - 8배, Slightly upregulate는 2배 - 4배이다.   
+ Strongly downregulate는 sample 2 RPKM 값이 sample 1에 비해 1/8 이하인 경우를 나타내고, Downregulate는 1/8-1/4,  Slightly downregulate는 1/4 - 1/2이다.
+ NA는 뚜렷한 발현량 차이가 없는(sample간 발현량 차이가 0.5-2인 경우를 나타낸다.)

   |DEG level|count|
   |-|-|
   |Total gene|24,952|
   |Strongly upregulated|79|
   |Upregulated|225|
   |Slightly upregulated|1,091|
   |Slightly downregulated|484|
   |Downregulated|90|
   |Strongly downregulated|17|
   |NA|22,966|


### 6-1. Distribution of RPKM
![Rplot](https://user-images.githubusercontent.com/97942772/191928772-e3fbff45-a651-46bc-a650-5a92ef28a7ed.png)


   + Sample1의 RPKM값을 X축 Sample2의 RPKM값을 Y축으로 설정하고, Scatter plot을 통해 나타냈다. 
   + Sample 1 RPCM의 최소값은 0.003이고, Sample 2 RPKM의 최소값은 0.001이다.
   + Sample 1 RPCM의 최대값은 16,634이고, Sample 2 RPKM의 최대값은 18,169이다. 
   + 데이터의 범위는 큰 반면, median은 sample1, sample2 각각 3.78, 3.97으로 낮았다. 이에 3rd Quater값인 14.10와 가까운 15까지 나타냈다.
   + Strongly upregulate는 sample 2 RPKM 값이 sample 1에 비해 8배 이상 큰 경우를 나타내고, Upregulate는 4배 - 8배, Slightly upregulate는 2배 - 4배이다.   
   + Strongly downregulate는 sample 2 RPKM 값이 sample 1에 비해 1/8 이하인 경우를 나타내고, Downregulate는 1/8-1/4,  Slightly downregulate는 1/4 - 1/2이다.
 
### 6-2. Distribution of Log RPKM
![logrpkm](https://user-images.githubusercontent.com/97942772/191929564-0dab38de-474f-4c27-b4d7-c04292f2bde7.png)

   + Sample1, Sampe2의 RPKM값의 로그 값을 취한 그래프이다. Pseudocount를 적용했다.


### 6-3. MA plot
![maplot](https://user-images.githubusercontent.com/97942772/191929727-6f54d87e-8a9c-4ba7-9d2d-da106e042469.png)

   + MA plot으로, X축에 Sample 1과 2의 평균값의 로그(Log10), Y축에 LogFC(Log2)으로 설정했다. Pseudocount를 적용했다.


