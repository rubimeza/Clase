
# Clase: Tutorial para Hybpiper
Este repositorio sirve para recordar todo lo que no debo hacer

1. No debo ponerle espacios a los nombres de los archivos

## Parte 1: Preparación
```
mkdir hybpiper_tutorial
```

```
cd hybpiper_tutorial
```

```
wget https://github.com/mossmatters/HybPiper/raw/master/test_dataset.tar.gz
```

```
# Descomprimir datos de archivo .tar.gz
tar -zxvf test_dataset.tar.gz
```

```
rm test_dataset.tar.gz
```

```
cd test_dataset
```

```
ls
```

```
tar -zxvf test_reads.fastq.tar.gz
```

```
rm test_reads.fastq.tar.gz
```

```
mkdir raw_reads
```

```
mv *.fastq raw_reads/
```

```
conda activate hybpiper
```

## Parte 2: Revisar calidad del *target_file*
```
less test_targets.fasta
```
```
hybpiper check_targetfile -t_dna test_targets.fasta
```

## Parte 3: Ensamble
```
# Ensamblar solo una muestra
hybpiper assemble -t_dna test_targets.fasta -r NZ281_R*_test.fastq --prefix NZ281 --bwa --cpu 4
```

```
ls NZ281/
```

```
rm -r NZ281/
```

```
less namelist.txt
```

```
# Ensamblar todas las secuencias
while read name;
do hybpiper assemble -t_dna test_targets.fasta -r raw_reads/$name*.fastq --prefix $name --bwa --cpu 4;
done < namelist.txt
```


```
# Ver toda la carpeta, incluyendo dentro de las carpetas
ls *
```


## Parte 4: Estadisticos de ensamble

```
hybpiper stats -t_dna test_targets.fasta gene namelist.txt
```

```
ls
```


```
### Ver en forma de tabla el archivo con la longitud de las secuencias principales
column -t -s $'\t' seq_lengths.tsv
```


```
### Ver primeras 7 columnas de estadísticos, referidas al ensamble
awk -F'\t' '{print $1, $2, $3, $4, $5, $6, $7}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```

```
### Ver siguientes cuatro columnas, referidas a la completitud de los genes
awk -F'\t' '{print $1, $8, $9, $10, $11}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```


```
### Ver siguientes cuatro columnas, referidas a estadísticos de parálogos y quimeras
awk -F'\t' '{print $1, $12, $13, $14, $15, $16, $17}' OFS='\t' hybpiper_stats.tsv | column -t -s $'\t'
```

```
### Generar heatmap de longitud de secuencias recuperadas
hybpiper recovery_heatmap seq_lengths.tsv
```

```
ls
```


Para descargar del servidor y ver el archivo png en la computadora local
```
# Para Windows 10 con ssh
# Ver como se llama la carpeta principal en la que estamos y cuál es el nombre de usuario
echo %cd%
scp -P 14420 rubis@189.241.xxx/xxx:/home/rubis/hybpiper_tutorial/test_dataset/recovery_heatmap.png C:\Users\usr\Desktop
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

```
### Generar matrices fasta de exones
hybpiper retrieve_sequences dna -t_dna test_targets.fasta --sample_names namelist.txt --fasta_dir exon_fastas
```


```
### Generar matrices fasta de intrones
hybpiper retrieve_sequences intron -t_dna test_targets.fasta --sample_names namelist.txt --fasta_dir intron_fastas
```

## Parte 6: Revisar parálogos

```
### Generar archivos de paralogos (directorios fasta, estadisticos y heatmap)
hybpiper paralog_retriever namelist.txt -t_dna test_targets.fasta
```

```
ls paralog*
```



```
### Ver tabla de cantidad de paralogos por muestra/gen
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

```
cat gene074_paralogs_all.fasta | mafft --auto - > gene074_paralogs_all_mafft.fas
```
```
iqtree2 -s gene074.fasta
```
