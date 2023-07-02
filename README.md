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
```
input_dir="RawReads"
output_dir="Trimmed_reads"
prefix="p_"

mkdir -p "$output_dir"

for file in "$input_dir"/*.fq.gz; do
  filename=$(basename "$file")
  output_file="$output_dir/$prefix$filename"

  java -jar /usr/share/java/trimmomatic-0.39.jar SE "$file" "$output_file" ILLUMINACLIP:/usr/share/trimmomatic/TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:20
done

```

--FastQC, comando utilizado para la creacion de FastQC_trimmedReads desde Trimmed_reads:

(EJ: Trimmed_reads/p_L1R1.fq.gz -> FastQC_trimmedReads/p_L1R1.html)
```
fastqc Trimmed_reads/*.fq.gz -o FastQC_trimmedReads
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
    output_file="${output_dir}${filename}.sam"
    # Aplicar el comando de alineamiento para el archivo actual
    bwa mem "${ref}" "${file}" > "${output_file}"
done
```

--Cambiamos los archivos sam a bam:
```
#!/bin/bash

# Directorio de entrada con archivos SAM
input_dir="./Aligned_reads/"

# Directorio de salida para los archivos BAM
output_dir="./Aligned_reads/"

# Iterar sobre cada archivo SAM en el directorio Aligned_reads
for file in "${input_dir}"*.sam; do
    # Obtener el nombre base del archivo sin la extensión
    filename=$(basename "$file" .sam)
    # Crear el nombre de archivo BAM de salida
    output_file="${output_dir}${filename}.bam"
    # Convertir el archivo SAM a BAM
    samtools view -bS "${file}" > "${output_file}"
done
```

-- Ordenamiento y indexacion de los archivos bam
```
#!/bin/bash

# Directorio de entrada con archivos BAM
input_dir="./Aligned_reads/"

# Directorio de salida para los archivos ordenados y indexados
output_dir="./Aligned_reads/"

# Iterar sobre cada archivo BAM en el directorio Aligned_reads
for file in "${input_dir}"*.bam; do
    # Obtener el nombre base del archivo sin la extensión
    filename=$(basename "$file" .bam)
    # Crear el nombre de archivo para el archivo ordenado
    sorted_file="${output_dir}${filename}_sorted.bam"
    # Crear el nombre de archivo para el archivo de índice
    index_file="${output_dir}${filename}.bai"
    # Ordenar el archivo BAM
    samtools sort "${file}" -o "${sorted_file}"
    # Indexar el archivo ordenado
    samtools index "${sorted_file}" "${index_file}"
done
```
