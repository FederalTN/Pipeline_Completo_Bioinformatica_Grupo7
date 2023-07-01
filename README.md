-Archivos grupo 7:
```
Libreria-17-51_S9_L001_R1_001.fastq.gz
Libreria-17-51_S9_L001_R2_001.fastq.gz
Libreria-17-68-R_S7_L001_R1_001.fastq.gz
Libreria-17-68-R_S7_L001_R2_001.fastq.gz
```

-Carpetas:
● DNAref, para almacenar el DNA de referencia
● RawReads, para almacenar los FASTQ obtenidos de la secuenciación (datos crudos)
● FastQC_rawReads, para almacenar los reportes de FastQC de los datos crudos
● Trimmed_reads, para almacenar los FASTQ podados
● FastQC_trimmedReads, para almacenar los reportes de FastQC de los datos podados
● Aligned_reads, para almacenar los archivos de alineamiento
● Variant_calling, para almacenar las variantes identificadas

-Creacion de enlace simbolico:
```
ln -s <ruta_del_objetivo> <ruta_del_enlace>
```

-FastQC, comando utilizado para la creacion de FastQC_rawReads:
```
fastqc RawReads/*R1*.fq.gz RawReads/*R2*.fq.gz -o FastQC_rawReads
```