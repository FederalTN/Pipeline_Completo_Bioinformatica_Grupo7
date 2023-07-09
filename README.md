--Archivos grupo 7:
```
Libreria-17-51_S9_L001_R1_001.fastq.gz
Libreria-17-51_S9_L001_R2_001.fastq.gz
Libreria-17-68-R_S7_L001_R1_001.fastq.gz
Libreria-17-68-R_S7_L001_R2_001.fastq.gz
```

----ORGANIZACION DE DIRECTORIOS:
```
● DNAref, para almacenar el DNA de referencia
● RawReads, para almacenar los FASTQ obtenidos de la secuenciación (datos crudos)
● FastQC_rawReads, para almacenar los reportes de FastQC de los datos crudos
● Trimmed_reads, para almacenar los FASTQ podados
● FastQC_trimmedReads, para almacenar los reportes de FastQC de los datos podados
● Aligned_reads, para almacenar los archivos de alineamiento
● Variant_calling, para almacenar las variantes identificadas
```

Glosario:
A -> B: El input del comando es "A" y el output es "B"

--Creacion de enlace simbolico :

(EJ: RawReads/Libreria-17-51_S9_L001_R1_001.fastq.gz -> RawReads/L1R1.fq.gz)
```
ln -s <ruta_del_objetivo> <ruta_del_enlace>
```

----CONTROL DE CALIDAD

--FastQC, comando utilizado para la creacion de FastQC_rawReads desde RawReads :

(EJ: RawReads/L1R1.fq.gz -> FastQC_rawReads/L1R1_fastqc.html)
```
fastqc RawReads/*.fq.gz -o FastQC_rawReads
```

--Trimmomatic, comando utilizado para la creacion de Trimmed_reads :

(EJ: RawReads/L1R1.fq.gz -> Trimmed_reads/p_L1R1.fq.gz)

Para L1
```
java -jar /usr/share/java/trimmomatic-0.39.jar PE RawReads/L1R1.fq.gz RawReads/L1R2.fq.gz \
Trimmed_reads/p_L1R1.fq.gz Trimmed_reads/u_L1R1.fq.gz Trimmed_reads/p_L1R2.fq.gz Trimmed_reads/u_L1R2.fq.gz \
ILLUMINACLIP:/usr/share/trimmomatic/NexteraPE-PE.fa:2:30:10:8 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:20

```
Para L2
```
java -jar /usr/share/java/trimmomatic-0.39.jar PE RawReads/L2R1.fq.gz RawReads/L2R2.fq.gz \
Trimmed_reads/p_L2R1.fq.gz Trimmed_reads/u_L2R1.fq.gz Trimmed_reads/p_L2R2.fq.gz Trimmed_reads/u_L2R2.fq.gz \
ILLUMINACLIP:/usr/share/trimmomatic/NexteraPE-PE.fa:2:30:10:8 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:20

```

--FastQC, comando utilizado para la creacion de FastQC_trimmedReads desde Trimmed_reads:

(EJ: Trimmed_reads/p_L1R1.fq.gz -> FastQC_trimmedReads/p_L1R1.html)
```
fastqc Trimmed_reads/p_*.fq.gz -o FastQC_trimmedReads
```

----ALINEAMIENTO

--Indexación del genoma de referencia:
```
bwa index DNAref/GRCh38.fasta
```

-- Alineamiento, ordenamiento y indexacion en archivos BAM mediante bwa y samtools (Crea los ALigned_reads)
Para L1:
```
bwa mem -t 8 ./DNAref/GRCh38.fasta ./Trimmed_reads/p_L1R1.fq.gz ./Trimmed_reads/p_L1R2.fq.gz > ./Aligned_reads/L1Aligned.bam
samtools sort -o ./Aligned_reads/L1Aligned_sort.bam ./Aligned_reads/L1Aligned.bam
samtools index ./Aligned_reads/L1Aligned_sort.bam
```
Para L2
```
bwa mem -t 8 ./DNAref/GRCh38.fasta ./Trimmed_reads/p_L2R1.fq.gz ./Trimmed_reads/p_L2R2.fq.gz > ./Aligned_reads/L2Aligned.bam
samtools sort -o ./Aligned_reads/L2Aligned_sort.bam ./Aligned_reads/L2Aligned.bam
samtools index ./Aligned_reads/L2Aligned_sort.bam
```

----VARIANT CALLING

--creación de archivo BCF intermedio (pileup)
```
samtools mpileup ./Aligned_reads/L1Aligned_sort.bam ./Aligned_reads/L2Aligned_sort.bam -g -f ./DNAref/GRCh38.fasta > ./Variant_calling/pileup.bcf

```
--calling
```

bcftools call -O b -vc ./Variant_calling/pileup.bcf > ./Variant_calling/varcall.bcf
```

--conversion bcf -> vcf
```
bcftools view ./Variant_calling/varcall.bcf > ./Variant_calling/varcall.vcf
```