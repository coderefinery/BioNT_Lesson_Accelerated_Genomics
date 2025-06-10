# Basic CPU-native NGS data processing workflow

- Simplest NGS data processing workflows have two main stages
  - Read mapping and alignment refinement
  - Variant calling
- **Test data:** `https://s3.amazonaws.com/parabricks.sample/parabricks_sample.tar.gz`
- **Source of the test data -** `NVIDIA tutorials <https://docs.nvidia.com/clara/parabricks/4.3.0/tutorials/gettingthesampledata.html>`_

## Stage 1: Read mapping and alignment refinement

### Step 1: BWA read mapping and sorting

```bash
#!/bin/bash

SAMPLE_NAME="pb_sample"

FASTA="Ref/Homo_sapiens_assembly38.fasta"
READ1="Data/sample_1.fq.gz"
READ2="Data/sample_2.fq.gz"
BAM_OUT="${SAMPLE_NAME}_CPU.bam"

## Run BWA MEM
### Source: https://github.com/nf-core/sarek/blob/f034b737630972e90aeae851e236f9d4292b9a4f/modules/nf-core/bwa/mem/main.nf#L30
bwa mem \
-t 64 \
-R "@RG\tID:rg1\tLB:lib1\tPL:bar\tSM:SM1\tPU:SM1_rg1" \
${FASTA} \
${READ1} ${READ2} \
| samtools     sort     -@64     -o ${BAM_OUT} -

samtools index -@64 -b ${BAM_OUT} 
```

### Step 2: GATK MarkDuplicates

```bash
#!/bin/bash

SAMPLE_NAME="pb_sample"
BAM_IN="${SAMPLE_NAME}_CPU.bam"
BAM_OUT="${SAMPLE_NAME}_markdup.bam"

## Mark duplicated reads in BAM file
gatk --java-options  "-Xss3m -XX:-UsePerfData" MarkDuplicates \
--VALIDATION_STRINGENCY SILENT \
-I ${BAM_IN} \
-O ${BAM_OUT} \
-M metrics.txt \
--TMP_DIR temp_dir

samtools index ${BAM_OUT} 
```

### Step 3: GATK BaseRecalibrator

```bash
#!/bin/bash

FASTA="Ref/Homo_sapiens_assembly38.fasta"
KNOWN_SITES="Ref/Homo_sapiens_assembly38.known_indels.vcf.gz"

SAMPLE_NAME="pb_sample"
BAM_IN="${SAMPLE_NAME}_markdup.bam"
RECAL_FILE="CPU_recal.txt"

## Generate BQSR Report
gatk --java-options  "-XX:-UsePerfData" BaseRecalibrator \
--input ${BAM_IN} \
--output ${RECAL_OUT} \
--known-sites ${KNOWN_SITES} \
--reference ${FASTA}
```

### Step 4: GATK ApplyBQSR

```bash
#!/bin/bash

FASTA=GRCh38"/"GCA_000001405.15_GRCh38_no_alt_analysis_set.fna
BAM_IN="${SAMPLE_NAME}_markdup.bam"
RECAL_FILE="CPU_recal.txt"
BAM_OUT="${SAMPLE_NAME}_CPU_markdup_BQSR.bam"

## Run ApplyBQSR Step
gatk --java-options  "-XX:-UsePerfData" ApplyBQSR \
-R ${FASTA} \
-I ${BAM_IN} \
--bqsr-recal-file ${RECAL_FILE} \
-O ${BAM_OUT}
```

## Stage 2: Variant calling via HaplotypeCaller

```bash
#!/bin/bash

set -euo pipefail

SAMPLE_NAME="pb_sample"
FASTA="Ref/Homo_sapiens_assembly38.fasta"
BAM_IN="${SAMPLE_NAME}_CPU_markdup_BQSR.bam"

## Variant calling with HaplotypeCaller
gatk --java-options  "-XX:-UsePerfData" HaplotypeCaller \
--input ${BAM_IN} \
--output ${BAM_IN}.vcf \
--reference ${FASTA} \
--native-pair-hmm-threads 64
```
