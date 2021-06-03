## Software version 
* bwa 0.7.17-r1188
* biobambam 2.0.87
* samtools 1.9
* varscan v2.4.4
* bcftools 1.9
* tabix 1.9

---
## SNP calling pipeline with NGS reads
* align illumina reads to reference genome
```shell script
bwa mem -M -R '@RG\\tID:sample_id\\tSM:mysample\\tLB:sample_lib\\tPL:ILM' \ 
  -t 24 ref.fa reads.R1.fq.gz reads.R2.fq.gz | \
  samtools view -Shb -@ 24 | \
  samtools sort -@ 24 -m 2G  -o mysample.bam -O BAM 
```

* marked duplicates with biobambam2

```shell script
bammarkduplicates2 I=mysample.bam O=mysample.markedd_dup.bam M=mysample.markedd_up.metrics \
  markthreads=2  tmpfile=/tmp
```

* samtools mpileup

```shell script
samtools mpileup -B -f  ref.fa -o mysample.markedd_dup.mpileup mysample.markedd_dup.bam
```

* call snp with varscan
```shell script
varscan mpileup2snp mysample.markedd_dup.mpileup --p-value 0.05 --mysample-vcf 1 | \
  bcftools view -Oz -o mysample.markedd_dup.varscan.vcf.gz
```

* build index for variants 

```shell script
tabix mysample.markedd_dup.varscan.vcf.gz
```
