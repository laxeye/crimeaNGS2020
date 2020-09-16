# crimeaNGS2020
SHort course on genome assembly and annotation


# Установка программ

## Установка conda

https://docs.conda.io/en/latest/miniconda.html

## Настройка каналов

conda config --add channels bioconda

conda config --add channels conda-forge

conda config --add channels defaults

## Установка программ для сборки и анализа генома

conda create -n crimeangs spades quast minimap2 bowtie2 samtools fastqc fastp bbmap blast biopython flye racon "samtools>=1.9" unicycler mash pyani sra-tools

conda activate crimeangs

## Программы для аннотации и анализа генома

conda install dfast prokka checkm-genome


## Активация виртуального окружения

conda activate crimeangs

## Скачивание данных

Узнайте, где вы сейчас: `pwd`

Перейдите в домашнюю папку: `cd ~`

Создайте папку для работы: `mkdir crimea`

Перейдите в папку: `cd crimea`

`cd ~/crimea`

`mkdir HI0118`

`cd HI0118`

`fasterq-dump -S -p -e 1 SRR5482709`

**SRR5482709** - идентфикатор в базе NCBI SRA: https://www.ncbi.nlm.nih.gov/sra

Просмотрим содержимое папки: `ls -lh`

Посмотрим, как выглядят данные: `less SRR5482709_1.fastq`

Для выхода из **less** надо нажать **q**.


## Оценка качества

**FastQC**

`fastqc *fastq`

**fastp**

`fastp -L -Q -G -A -z 1 --stdout -w 1 -i SRR5482709_1.fastq -I SRR5482709_2.fastq -h SRR5482709.fastq.html -j SRR5482709.fastq.json > /dev/null`

Полученные отчёты можно открыть в браузере.


## Оценка длины генома

`mash sketch -r -m 1 SRR*fastq`

`mash sketch -r -m 3 SRR*fastq`


## Предобработка прочтений

### Обрезка по качеству, фильтрация

`mkdir preprocess`

`cd preprocess`

`bbduk.sh Xmx=4G t=1 ref=../sr.adapters.fasta k=19 ktrim=r qtrim=r trimq=16 entropy=0.5 minlength=33 in=../SRR5482709_1.fastq in2=../SRR5482709_2.fastq out=pe_1.fq out2=pe_2.fq stats=bbduk.stats.pe.txt`

### Объединение парных прочтений

`bbmerge.sh Xmx=4G t=1 in1=pe_1.fq in2=pe_2.fq outu1=nm.pe_1.fq.gz outu2=nm.pe_2.fq.gz out=merged.fq.gz`


### Оценка результатов

`mash sketch -r -m 3 pe*fq`

`fastqc pe*fq`

`fastqc nm.pe*gz merged.fq.gz`

Просмотрим отчёты в браузере.

Удалим промежуточные файлы `rm pe*fq`


## Сборка генома при помощи SPAdes

`cd ..`

`spades.py -1 preprocess/nm.pe_1.fq.gz -2 preprocess/nm.pe_2.fq.gz -t 2 -m 4 -o spades --merged preprocess/merged.fq.gz`


Пока идёт сборка найдём, что есть в базах по этому геному:

https://www.ncbi.nlm.nih.gov/assembly/

HI0118

Скачаем эту сборку

Перенести файл `mv ~/Downloads/name1 ./name2`


## Оценка качества генома

quast.py spades/contigs.fasta spades/scaffolds.fasta ../GCA_001635265*

mkdir analasys
cd analasys/
cp ../spades2/contigs.fasta spades.merged.fna
cp ../spades/contigs.fasta spades.nonmerged.fna
cp ../GCA_001635265.1_ASM163526v1_genomic.fna genbank.fna
checkm taxonomy_wf -t 2 -f checkm.txt domain Bacteria . checkm


Для закрепления повторим с несколькими другими образцами:

Oleiphilus sp. HI0130    SRR5482702 

Erythrobacter sp. HI0063    SRR5482750 

Erythrobacter sp. HI0037    SRR5482760 


## Гибридная сборка

Bacillus cereus isolate from northern Namibia  

https://www.ncbi.nlm.nih.gov/assembly/GCF_013177495.1   

Nanopore SRR12342877 https://www.ncbi.nlm.nih.gov/sra/SRX8842584 

MiSeq SRR12342876 https://www.ncbi.nlm.nih.gov/sra/SRX8842585 

## Скачиваем данные

`cd ~/crimea`

`mkdir hybrid`

`cd hybrid`

`fasterq-dump -p -e 1 -S SRR12342876`

`fasterq-dump -p -e 1 SRR12342877`

Оценим качество при помощи fastp аналогично первому набору данных

## Проведём обработку прочтений SRR12342876 аналогично предыдущим данным

## Выполним сборку только парынх прочтений

`spades.py -1 nm.pe_1.fq.gz -2 nm.pe_2.fq.gz --merged merged.fq.gz -o spades-sr`

## Выполним сборку только длинных прочтений

`unicycler -l SRR12342877.fq -o unicycler-long -t 2`

## Выполним гибридную сборку

`unicycler -1 nm.pe_1.fq.gz -2 nm.pe_2.fq.gz -s merged.fq.gz -l SRR12342877.fq -o unicycler -t 2`

## Сравним результаты

`quast.py spades/contigs.fasta unicycler-long/assembly.fna unicycler/assembly.fna`

