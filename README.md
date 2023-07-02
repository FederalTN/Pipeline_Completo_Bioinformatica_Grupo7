--Archivos grupo 7:
```
Libreria-17-51_S9_L001_R1_001.fastq.gz
Libreria-17-51_S9_L001_R2_001.fastq.gz
Libreria-17-68-R_S7_L001_R1_001.fastq.gz
Libreria-17-68-R_S7_L001_R2_001.fastq.gz
```

--Carpetas:
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

--FastQC, comando utilizado para la creacion de FastQC_rawReads desde Trimmed_reads:

(EJ: Trimmed_reads/p_L1R1.fq.gz -> FastQC_rawReads/p_L1R1.html)
```
fastqc Trimmed_reads/*.fq.gz -o FastQC_rawReads
```

