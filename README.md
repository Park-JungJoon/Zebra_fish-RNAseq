# Zebrafish-RNAseq
## 1. Purpose
+ Zebrafish embryo에 1,4-Naphthoquinone (1,4-NQ)를 처리한 후 control group과 비교해 Differentially Expressed Genes (DEGs)를 확인함.
+ 기존 연구 결과를 확인해 1,4-NQ 관련 biomarker 유전자들이 DEG에 포함되어 있는지 탐색함. 

## 2. Data Collecting
+ 실험 및 RNA 추출: 충남대학교 생명시스템과학대학 김철희 교수님 연구실 박사과정 PUSPANJALI SWAIN 연구원
+ Sequencing: [Marcrogen, Inc.](https://www.macrogen.com/ko/main)
+ 전체 sample은 2개로 sample 1은 아무 물질도 처리하지 않은 control group이고, sample 2는 0.5μM의 1,4-NQ를 6hpf 시점에서 처리한 group이다.

   | Sample | Category | Treat | Extract point
   | - | - | - | -
   | Sample1 | Control | Untreated | 10hpf (bud stage)
   | Sample2 | Treated | 1,4-NQ 0.5μM at 6hpf | 10hpf (bud stage)

+ Zebrafish reference genome으로는 NCBI의 GRCz11 (RefSeq accession: GCF_000002035.6) genome을 사용함.

   Feature | Statistics
   | - | -
   Genome size (bp) | 1,373,454,788
   Number of chromosomes | 25
   Number of scaffolds | 1,917
   Scaffold N50 (bp)| 7,379,053
   Scaffold L50 | 44
   GC ratio (%) | 36.5
   Number of genes | 40,031
   Number of protein-coding genes | 26,448

## 3. Read adapter trimming & QC
+ TrimGalore (v0.6.6)를 이용해, read adapter trimming과 QC를 진행함.

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
+ R 코드와 Python 코드를 통해, bam 파일로 부터 각 유전자의 발현량, RPKM, LogFC값을 표기하는 [sorted_filtered_rpkm_foldchange.tsv](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/sorted_filtered_rpkm_foldchange.tsv) 파일을 만들었다. Protein coding gene을 제외한, tRNA,rRNA,miRNA,lncRNA 등의 non-coding gene은 모두 제거되었다. 
+ Sample 1,2 두 샘플 중 하나라도 발현량이 0인 gene 제외하였다.
+ 아래 표는 sample 1,2에서의 유전자 발현량의 기본적인 통계를 나타냈다.

   ||Mean|Min|Max|Median|1st Qu.|3rd Qu.|
   |-|-|-|-|-|-|-|
   |Sample1 gene expression count|2,222|1|598,059|440|83|1,813|
   |Sample2 gene expression count|2,193|1|489,505|460|94|1,808|
   |Sample1 RPKM|25.92|0.003|16,634|3.78|0.86|13.86|
   |Sample2 RPKM|26.51|0.001|18,169|3.97|0.96|14.10|

## 6. Data Statistics
+ R을 이용한 통계를 내었다. 

   |DEG level|Treat/Control|count|
   |-|-|-|
   |Total gene|-|24,952|
   |Strongly upregulated|8배 이상|79|
   |Upregulated|4배 이상, 8배 이하|225|
   |Slightly upregulated|2배 이상, 4배이하|1,091|
   |Slightly downregulated|1/4배 이상, 1/2배 이하|484|
   |Downregulated|1/8배 이상, 1/4배 이하|90|
   |Strongly downregulated|1/8배 이하|17|
   |NA|1/2배 이상, 2배 이하|22,966|


### 6-1. Distribution of RPKM
![Rplot](https://user-images.githubusercontent.com/97942772/191928772-e3fbff45-a651-46bc-a650-5a92ef28a7ed.png)

   + Sample1의 RPKM값을 X축 Sample2의 RPKM값을 Y축으로 설정하고, Scatter plot을 통해 나타냈다. 
   + Sample 1 RPCM의 최소값은 0.003이고, Sample 2 RPKM의 최소값은 0.001이다.
   + Sample 1 RPCM의 최대값은 16,634이고, Sample 2 RPKM의 최대값은 18,169이다. 
   + 데이터의 범위는 큰 반면, median은 sample1, sample2 각각 3.78, 3.97으로 낮았다. 이에 sample 2의 3rd Quater값인 X,Y축을 14.10와 가장 가까운 정수, 14까지 나타냈다.
   
 
### 6-2. Distribution of Log RPKM
![logrpkm](https://user-images.githubusercontent.com/97942772/191929564-0dab38de-474f-4c27-b4d7-c04292f2bde7.png)

   + Sample1, Sampe2의 RPKM값의 로그 값을 취한 그래프이다. 
   + Pseudocount를 적용했다.
      * RPKM값이 1 이하일 경우, Log RPKM 값이 음수가 나오며, 그래프를 통한 시각적 인식의 어려움이 있어, sample 1, sample 2의 RPKM값에 1을 더한 후 Log 값을 계산했다. 


### 6-3. MA plot
![maplot](https://user-images.githubusercontent.com/97942772/191929727-6f54d87e-8a9c-4ba7-9d2d-da106e042469.png)

   + MA plot으로, X축에 Sample 1과 2의 평균값의 로그(Log10), Y축에 LogFC(Log2)으로 설정했다. Pseudocount를 적용했다.

## 7. Result
