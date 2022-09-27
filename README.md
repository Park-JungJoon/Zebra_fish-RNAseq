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
+ TrimGalore (v0.6.6)를 사용해, read의 adapter trimming과 QC를 진행함.

   | Read | Total Read | Filtered Read | Total basepairs | Filtered basepairs
   | - | - | - | - | -
   | Sample1_1 | 36,675,862 | 36,675,862 | 3,704,262,062 | 3,679,353,873 (99.3%)
   | Sample1_2 | 36,675,862 | 36,675,862 | 3,704,262,062 | 3,672,750,272 (99.1%)
   | Sample2_1 | 37,414,748 | 37,414,748 | 3,778,889,548 | 3,752,045,586 (99.3%)
   | Sample2_2 | 37,414,748 | 37,414,748 | 3,778,889,548 | 3,745,527,653 (99.1%)

## 4. Read alignment
+ NCBI reference genome을 대상으로 QC가 끝난 read를 사용해 HISAT2 (v2.2.1)를 사용해 read alignment를 진행함.
+ Running parameters: --max-intronlen 50000, other parameters default
+ Read alignment 결과 통계는 아래와 같음.

   | Sample | Total reads | Overall alignment rate
   | - | - | -
   | Sample1 | 36,643,192 | 88.78%
   | Sample2 | 37,384,688 | 88.20%

## 5. DEG analysis
+ R의 GenomicAlignments package (v1.30.0)를 사용해 gene 별로 align된 read의 raw count를 구함.
+ Raw count를 RPKM 방식으로 보정한 후, Log2FC (Fold change) 값을 계산함. [sorted_filtered_rpkm_foldchange.tsv](https://github.com/Park-JungJoon/Zebra_fish-RNAseq/blob/main/Supplementary_data/sorted_filtered_rpkm_foldchange.tsv)
+ Protein-coding gene을 제외한, tRNA, rRNA, miRNA, lncRNA 등의 non-coding gene은 모두 제거함.
+ 두 샘플 중 하나라도 raw count가 0인 gene은 DEG 분석에서 제외함.
+ 아래 표는 Sample 1, 2의 유전자 발현량 기본 통계를 나타냄.

   | Statistics | Sample1 raw count | Sample2 raw count | Sample1 RPKM | Sample2 RPKM
   | - | - | - | - | -
   | Mean | 2,222 | 2,193 | 25.92 | 26.51
   | Min | 1 | 1 | 0.003 | 0.001
   | Max | 598,059 | 489,505 | 16,634 | 18,169
   | 1st Quarter | 83 | 94 | 0.86 | 0.96
   | Median | 440 | 460 | 3.78 | 3.97
   | 3rd Quarter | 1,813 | 13.86 | 14.10

+ 유전자 발현량 차이에 따라 유전자를 아래와 같이 분류함.

   | DEG category | LogFC (Treat/Control) | Count
   | - | - | -
   | Total gene | - | 24,952
   | Strongly upregulated | LogFC >= 3 | 79
   | Upregulated | 3 > LogFC >= 2 | 225
   | Slightly upregulated | 2 > LogFC >= 1 | 1,091
   | NA | 1 > LogFC >= -1 | 22,966
   | Slightly downregulated | -1 > LogFC > -2 | 484
   | Downregulated | -2 >= LogFC > -3 | 90
   | Strongly downregulated | -3 >= LogFC | 17

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
