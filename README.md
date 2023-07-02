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

--Alineamiento de las secuencias PE que resultaron de la poda:
```
#!/bin/bash

# Ruta al archivo GRCh38.fasta en el directorio DNAref
ref="./DNAref/GRCh38.fasta"

# Directorio de entrada de los archivos Trimmed_reads
input_dir="./Trimmed_reads/"

# Directorio de salida para los archivos SAM
output_dir="./Aligned_reads/"

# Iterar sobre cada archivo en el directorio Trimmed_reads
for file in "${input_dir}"*.fq.gz; do
    # Obtener el nombre base del archivo sin la extensión
    filename=$(basename "$file" .fq.gz)
    # Crear el nombre de archivo SAM de salida
    output_file="${output_dir}${filename}.bam"
    # Aplicar el comando de alineamiento para el archivo actual
    bwa mem "${ref}" "${file}" > "${output_file}"
done
```

-- Alineamiento, ordenamiento y indexacion en archivos BAM mediante bwa y samtools
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



WIP ZONE!!!!

bwa mem -t 8 ./DNAref/GRCh38.fasta ./Trimmed_reads/p_L1R1.fq.gz ./Trimmed_reads/p_L1R2.fq.gz > ./Aligned_reads/output1.bam
samtools sort -o ./Aligned_reads/output1_sort.bam ./Aligned_reads/output1.bam
samtools index ./Aligned_reads/output1_sort.bam

bwa mem -t 4 ./DNAref/GRCh38.fasta ./Trimmed_reads/p_L2R1.fq.gz ./Trimmed_reads/p_L2R2.fq.gz | samtools sort -o ./Aligned_reads/output2.sorted.bam -

samtools index ./Aligned_reads/output2.sorted.bam



bwa mem -t 8 ./DNAref/GRCh38.fasta ./RawReads/L1R1.fq.gz ./RawReads/L1R2.fq.gz | samtools sort -o ./Aligned_reads/output4.sorted.bam -

samtools index ./Aligned_reads/output3.sorted.bam


bwa mem -t 8 ./DNAref/GRCh38.fasta ./RawReads/Libreria-17-51_S9_L001_R1_001.fastq.gz ./RawReads/Libreria-17-51_S9_L001_R2_001.fastq.gz | samtools sort -o ./Aligned_reads/output4.sorted.bam -

samtools index ./Aligned_reads/output4.sorted.bam



samtools index ./Aligned_reads/p_L1R1_sorted.bam ./Aligned_reads/p_L1R1_sorted.bam.fai







(funciona el trimmed!)
java -jar /usr/share/java/trimmomatic-0.39.jar PE RawReads/L1R1.fq.gz RawReads/L1R2.fq.gz \
Trimmed_reads/p_L1R1.fq.gz Trimmed_reads/u_L1R1.fq.gz Trimmed_reads/p_L1R2.fq.gz Trimmed_reads/u_L1R2.fq.gz \
ILLUMINACLIP:/usr/share/trimmomatic/TruSeq3-SE.fa:2:30:10 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:20
