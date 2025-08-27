
# Clase: Tutorial para Hybpiper de la sra perez
Este repositorio sirve para recordar todo lo que no debo hacer

1. No debo ponerle espacios a los nombres de los archivos

## Parte 1: Preparación
Crear directorio principal de Hybpiper y moverse a el:
```
mkdir hybpiper_tutorial
cd hybpiper_tutorial
```
Descargar datos de prueba de *Artocarpus*

```
wget https://github.com/mossmatters/HybPiper/raw/master/test_dataset.tar.gz
```
Descomprimir datos de archivo .tar.gz

```
tar -zxvf test_dataset.tar.gz
rm test_dataset.tar.gz
```
Ver estructura de la carpeta. Es necesario volver a descomprimir.
```
cd test_dataset
ls
```
Descomprimir lecturas en formato fastq
```
tar -zxvf test_reads.fastq.tar.gz
rm test_reads.fastq.tar.gz
```
Ver lecturas descomprimidas

```
mkdir raw_reads
mv *.fastq raw_reads/
ls raw_reads/
```
Esta es la estructura actual de la carpeta:
```
.
├── namelist.txt
├── raw_reads
│   ├── EG30_R1_test.fastq
│   ├── EG30_R2_test.fastq
│   ├── EG98_R1_test.fastq
│   ├── EG98_R2_test.fastq
│   ├── MWL2_R1_test.fastq
│   ├── MWL2_R2_test.fastq
│   ├── NZ281_R1_test.fastq
│   ├── NZ281_R2_test.fastq
│   ├── NZ728_R1_test.fastq
│   ├── NZ728_R2_test.fastq
│   ├── NZ739_R1_test.fastq
│   ├── NZ739_R2_test.fastq
│   ├── NZ866_R1_test.fastq
│   ├── NZ866_R2_test.fastq
│   ├── NZ874_R1_test.fastq
│   ├── NZ874_R2_test.fastq
│   ├── NZ911_R1_test.fastq
│   └── NZ911_R2_test.fastq
├── run_hybpiper_test_dataset.sh
└── test_targets.fasta

1 directory, 21 files
```
Activar ambiente conda de hybpiper para empezar a trabajar.
```
conda activate hybpiper
```

## Parte 2: Revisar calidad del *target_file*

Ver formato de archivo de referencia
```
less test_targets.fasta
```
Ver calidad de archivo de referencia
```
hybpiper check_targetfile -t_dna test_targets.fasta
```
Resultados esperados:
```
[INFO]:    Checking target file for issues...
[INFO]:    The target file FASTA header formatting looks good!
[INFO]:    The target file test_targets.fasta has been specified as containing DNA
           sequences. These DNA sequences will be checked for low-complexity regions.
           NOTE: the sequences flagged as having low-complexity regions can sometimes
           differ between a DNA target file and the corresponding translated protein
           target file. If you translate your target file to run "hybpiper assemble" with
           BLASTx/DIAMOND, we recommend re-checking the translated sequences for low-
           complexity regions.
[INFO]:    Checking the target file for sequences with low-complexity regions...
[INFO]:    Type of sequences: DNA
[INFO]:    Sliding window size: 100
[INFO]:    Minimum complexity threshold: 1.5
           Elapsed Time: 0:00:13|####################################|Time:  0:00:13
[INFO]:    No sequences with low-complexity regions found.
[INFO]:    There are 26 sequences in your target file that contain a single terminal stop
           codon. Sequence names have been written to a report file.
[INFO]:    No problems found in target file!
[INFO]:    A target file report has been written to:
           /home/angel/hybpiper_tutorial/test_dataset/check_targetfile_report-test_targets.txt

[INFO]:    The control file required for command "hybpiper fix_targetfile" has been written to
           "fix_targetfile_2025-08-27-15_24_25.ctl".

```

## Parte 3: Ensamble

### 3.1 Ensamblar solo una muestra

```
hybpiper assemble -t_dna test_targets.fasta -r raw_reads/NZ281_R*_test.fastq --prefix NZ281 --bwa --cpu 4
```
Ver resultados de ensamble para una muestra
```
ls NZ281/
```
Eliminar resultados
```
rm -r NZ281/
```
### 3.2 Ensamblar todas las secuencias con un solo comando:
Primero es necesario crear un "namelist" (una lista de las muestras a procesar)
```
less namelist.txt
```
Ensamble multiple con while loop
```
while read name;
do hybpiper assemble -t_dna test_targets.fasta -r raw_reads/$name*.fastq --prefix $name --bwa --cpu 4;
done < namelist.txt
```
Ver toda la carpeta, incluyendo dentro de las carpetas

```
ls *
# Tal vez tree si esta instalado
```
Estructura actual de la carpeta: 
```
.
├── check_targetfile_report-test_targets.txt
├── EG30
├── EG98
├── fix_targetfile_2025-08-27-15_24_25.ctl
├── MWL2
├── namelist.txt
├── NZ281
├── NZ728
├── NZ739
├── NZ866
├── NZ874
├── NZ911
├── raw_reads
├── run_hybpiper_test_dataset.sh
└── test_targets.fasta

```

## Parte 4: Estadisticos de ensamble

Generar estadisticos
```
hybpiper stats -t_dna test_targets.fasta gene namelist.txt
```
Genera dos archivos
```
ls
```
Ver en forma de tabla el archivo con la longitud de las secuencias principales

```
column -t -s $'\t' seq_lengths.tsv
```
Ver primeras 7 columnas de estadísticos, referidas al ensamble

```
awk -F'\t' '{print $1, $2, $3, $4, $5, $6, $7}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```
Ver siguientes cuatro columnas, referidas a la completitud de los genes
```
awk -F'\t' '{print $1, $8, $9, $10, $11}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```
Ver siguientes cuatro columnas, referidas a estadísticos de parálogos y quimeras

```
awk -F'\t' '{print $1, $12, $13, $14, $15, $16, $17}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```
Generar heatmap de longitud de secuencias recuperadas
```
hybpiper recovery_heatmap seq_lengths.tsv
```
Heatmap de recuperación para datos de *Artocarpus*
![alt text](https://github.com/rubimeza/Clase/blob/main/recovery_heatmap.png?raw=true)

Para descargar del servidor y ver el archivo png en la computadora local
```
# Para Windows 10 con ssh
# Ver como se llama la carpeta principal en la que estamos y cuál es el nombre de usuario
echo %cd%
scp -P 14420 rubis@189.241.xxx/xxx:/mnt/furys/rubis/hybpiper_tutorial/test_dataset/recovery_heatmap.png C:\Users\usr\Desktop
```

```
# Si no funciona el anterior o es una versión anterior a Windows 10, instala Putty y utiliza pscp
pscp -scp  rubis@189.241.xxx/xxx:/home/rubis/hybpiper_tutorial/test_dataset/recovery_heatmap.png C:\Users\usr\Desktop
```

```
# En sistema UNIX
scp -p 14420 rubis@189.241.xxx/xxx:/home/rubis/hybpiper_tutorial/test_dataset/recovery_heatmap.png /home/usr/Desktop
```




## Parte 5: Extracción de secuencias
Generar matrices fasta de exones


```
hybpiper retrieve_sequences dna -t_dna test_targets.fasta --sample_names namelist.txt --fasta_dir exon_fastas
```

Generar matrices fasta de intrones
```
hybpiper retrieve_sequences intron -t_dna test_targets.fasta --sample_names namelist.txt --fasta_dir intron_fastas
```

## Parte 6: Revisar parálogos

Generar archivos de paralogos (directorios fasta, estadisticos y heatmap)
```
hybpiper paralog_retriever namelist.txt -t_dna test_targets.fasta
```

```
ls paralog*
```

Ver tabla de cantidad de paralogos por muestra/gen

```
column -t -s $'\t' paralog_report.tsv
```

Ver árboles de parálogos

```
cat gene074_paralogs_all.fasta | mafft --auto - | FastTree -nt -gtr > gene074.paralogs.tre
```
```
cat gene006_paralogs_all.fasta | mafft --auto - | FastTree -nt -gtr > gene006.paralogs.tre
```

```
conda activate phyluce 
```
Alineamiento y árbol
```
cat gene074_paralogs_all.fasta | mafft --auto - > gene074_paralogs_all_mafft.fas
iqtree2 -s gene074.fasta
```
