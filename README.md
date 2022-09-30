# Zebrafish RNA-seq analysis
## 1. Purpose
+ Zebrafish embryo에 1,4-Naphthoquinone (1,4-NQ)를 처리한 후 control group과 비교해 Differentially Expressed Genes (DEGs)를 확인함.
+ 기존 연구 결과를 확인해 1,4-NQ 관련 biomarker 유전자들이 DEG에 포함되어 있는지 탐색함. 

## 2. Data collecting
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
+ TrimGalore (v0.6.6)를 사용해, read의 adapter trimming과 QC를 진행함.

   | Read | Total Read | Filtered Read | Total basepairs | Filtered basepairs
   | - | - | - | - | -
   | Sample1_1 | 36,675,862 | 36,675,862 | 3,704,262,062 | 3,679,353,873 (99.3%)
   | Sample1_2 | 36,675,862 | 36,675,862 | 3,704,262,062 | 3,672,750,272 (99.1%)
   | Sample2_1 | 37,414,748 | 37,414,748 | 3,778,889,548 | 3,752,045,586 (99.3%)
   | Sample2_2 | 37,414,748 | 37,414,748 | 3,778,889,548 | 3,745,527,653 (99.1%)

+ QC가 끝난 sample에 대해 sample1, sample2의 read를 무작위하게 3개로 나눠 각 샘플당 3개의 replicate를 만듦.


## 4. Read alignment
+ NCBI reference genome을 대상으로 QC가 끝난 read를 사용해 HISAT2 (v2.2.1)를 사용해 read alignment를 진행함.
+ Used parameters: --max-intronlen 50000, other parameters default value
+ Read alignment 결과 통계는 아래와 같음.

   | Sample | Total reads | Overall alignment rate
   | - | - | -
   | Sample1_a | 12,216,314 | 88.77%
   | Sample1_b | 12,211,778 | 88.76%
   | Sample1_c | 12,215,100 | 88.76%
   | Sample2_a | 12,461,574 | 88.16%
   | Sample2_b | 12,457,613 | 88.18%
   | Sample2_c | 12,465,501 | 88.18%

## 5. DEG analysis
+ R의 GenomicAlignments package (v1.30.0)를 사용해 gene 별로 align된 read의 raw count를 구함.
+ Raw count를 R의 edgeR package (v3.36.0)를 사용해 RPKM, Log2FC (Fold change), P-value 등의 값을 계산함. [2022_zebrafish_triplate_deg_table_20210405.txt](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/2022_zebrafish_triplate_deg_table_20210405.txt)
+ Protein-coding gene을 제외한, tRNA, rRNA, miRNA, lncRNA 등의 non-coding gene은 모두 제거함

### 5.1 edge R
+ 아래 표는 Sample 1, 2의 유전자 발현량 기본 통계를 나타냄.
  * Raw count 
  
   | Statistics | Sample1_a raw count | Sample1_b raw count | Sample1_c raw count | Sample2_a raw count | Sample2_b raw count | Sample2_c raw count | Sample 1 AVG raw count | Sample 2 AVG raw count|
   | - | - | - | - | - | - | - | - | - 
   | Mean | 652.8 | 652.5 | 652.7 | 643.7 | 643.5 | 644.3 | 652.69 | 643.9
   | Min | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0
   | Max | 197,521 | 196,954 | 197,521 | 160,595 | 162,025 | 161,897 | 197,332 | 161,506 |
   | 1st Quarter | 13 | 13 | 14 | 16 | 16 | 16 | 13.67 | 16
   | Median | 107 | 106 | 106 | 113 | 112 | 112 | 106.33 | 113
   | 3rd Quarter | 502 | 503 | 502 | 505 | 505 | 506 | 501.67 | 506.4
   
   * RPKM

   | Statistics | Sample1_a RPKM | Sample1_b RPKM | Sample1_c RPKM | Sample2_a RPKM | Sample2_b RPKM | Sample2_c RPKM | Sample 1 AVG RPKM | Sample 2 AVG RPKM |
   | - | - | - | - | - | - | - | - | - 
   | Mean | 2.394 | 2.391 | 2.391 | 2.416 | 2.413 | 2.415 | 2.392 | 2.4145
   | Min | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0
   | Max | 1736.59 | 1718.83 | 1791.81 | 1853.94 | 1865.12 | 1904.55 | 1749.08 | 1874.54 |
   | 1st Quarter | 0.056 | 0.056 | 0.056 | 0.063 | 0.063 | 0.064 | 0.0575 | 0.065
   | Median | 0.298 | 0.298 | 0.297 | 0.305 | 0.309 | 0.309 | 0.298 | 0.308
   | 3rd Quarter | 1.266 | 1.266 | 1.270 | 1.274 | 1.271 | 1.268 | 1.272 | 1.273
   
+ 유전자 발현량 차이에 따라 유전자를 아래와 같이 분류함.
  * P-value 0.005이하는 통계적으로 DEG가 아님. Log FC 값에 상관 없이 Non DEG에 분류함.   

   | DEG category | LogFC (Treat/Control) | Count
   | - | - | -
   | Total genes | - | 32,843
   | Strongly upregulated | LogFC >= 3 | 176
   | Upregulated | 3 > LogFC >= 2 | 197
   | Slightly upregulated | 2 > LogFC >= 1 | 759
   | NA | 1 > LogFC >= -1 | 31,402
   | Slightly downregulated | -1 > LogFC > -2 | 215
   | Downregulated | -2 >= LogFC > -3 | 36
   | Strongly downregulated | -3 >= LogFC | 58

### 5-1. Distribution of RPKM
![Rplot](https://user-images.githubusercontent.com/97942772/191928772-e3fbff45-a651-46bc-a650-5a92ef28a7ed.png)

   + Sample1의 RPKM 값을 X축, sample2의 RPKM 값을 Y축으로 설정하고 scatterplot을 통해 분포를 확인함.
   + RPKM 최대값(15,000 이상)에 비해, median은 sample1, sample2 각각 3.78, 3.97으로 낮음.
   + 따라서 X, Y축의 범위를 3rd Quarter 값인 14까지만 제한하여 나타냄.
 
### 5-2. Distribution of Log RPKM
![logrpkm](https://user-images.githubusercontent.com/97942772/191929564-0dab38de-474f-4c27-b4d7-c04292f2bde7.png)

   + Sample1, sample2 RPKM의 로그 값 분포를 확인함. 
   + RPKM 값이 1 이하일 경우 Log RPKM 값은 음수가 되어 그래프를 통한 시각적 인식에 어려움이 있어 RPKM값에 1의 pseudocount를 더한 후 로그 값을 계산함.

### 5-3. MA plot
![maplot](https://user-images.githubusercontent.com/97942772/191929727-6f54d87e-8a9c-4ba7-9d2d-da106e042469.png)

   + X축을 Sample 1과 2의 RPKM 평균의 로그 값(Log10, pseudocount 적용)으로, Y축을 LogFC(Log2)로 설정함.

## 6. Result
+ DEG 분석에서 LogFC의 절댓값이 2 이상이고, 두 샘플의 RPKM 값 모두 1이 넘는 유전자만 필터링함. [rnaseq_zebrafish_filtered.tsv.txt](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/rnaseq_zebrafish_filtered.tsv.txt)
+ 기준을 통과한 유전자들의 통계는 아래와 같음.

   | DEG category | Count
   | - | -
   | Total filtered genes | 36
   | Strongly upregulated | 9
   | Upregulated | 25
   | Downregulated | 2

+ Strongly upregulated group에 포함된 hsp70.3, hsp70.1, hsp90aa1.2, dnaja (hsp40) 유전자가 human carcinoma A431 cell에서 1,4-NQ 처리 시 발현이 증가했다는 [연구 결과](https://www.sciencedirect.com/science/article/pii/S0891584916311492)가 있음.
+ Strongly upregulated group의 나머지 유전자인 [LOC103909982](https://www.ncbi.nlm.nih.gov/gene/103909982), pdcd4b-2, pdcd4b, [si:dkey-204l11.1](https://www.ncbi.nlm.nih.gov/gene/100006301), bbc3는 programmed cell death 또는 apoptosis 관련 유전자로 확인되었고, 1,4-NQ는 electron-accepting capability으로 인해 ROS (reactive oxygen species) 생산을 증가시켜 cell apoptosis를 유발한다는 [연구 결과](https://www.spandidos-publications.com/10.3892/mmr.2019.10500)가 있음.
+ 또한 upregulated group에서도 heat shock protein (HSP) 관련 유전자들인 dnajb1b, ahsa1a, dnajb2, hspa4a 등이 확인됨.
+ 따라서 이번 실험에서 확인한 1,4-NQ 처리 후 발견된 DEG들이 기존의 연구들과 유사한 경향성을 보이고 있다고 할 수 있음.
