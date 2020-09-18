# crimeaNGS2020
Short course on genome assembly and annotation


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

`quast.py spades/contigs.fasta spades/scaffolds.fasta ../GCA_001635265*`

`mkdir analasys`

`cd analasys/`

`cp ../spades2/contigs.fasta spades.merged.fna`

`cp ../spades/contigs.fasta spades.nonmerged.fna`

`cp ../GCA_001635265.1_ASM163526v1_genomic.fna genbank.fna`

`checkm taxonomy_wf -t 2 -f checkm.txt domain Bacteria . checkm`


Для закрепления повторим с несколькими другими образцами:

Oleiphilus sp. HI0130    SRR5482702 

Erythrobacter sp. HI0063    SRR5482750 

Erythrobacter sp. HI0037    SRR5482760 


# Гибридная сборка

Bacillus cereus isolate from northern Namibia  

Имеющаяся геномная сборка:

https://www.ncbi.nlm.nih.gov/assembly/GCF_013177495.1   

**Скачивание данных**

Активируем виртуальное окружение: `conda activate crimea`

Переходим в рабочую папку: `cd ~/crimea`

Создаём папку: `mkdir hybrid`

Переходим в папку: `cd hybrid`

**Nanopore**

`fasterq-dump -p -e 1 SRR12342877`

https://www.ncbi.nlm.nih.gov/sra/SRX8842584 

**MiSeq**

`fasterq-dump -p -e 1 -S SRR12342876`

https://www.ncbi.nlm.nih.gov/sra/SRX8842585 

## Оценим качество при помощи fastp аналогично первому набору данных

`fastp -L -Q -G -A -z 1 --stdout -w 1 -i SRR12342876_1.fastq -I SRR12342876_2.fastq -h SRR12342876.fastq.html > /dev/null`

`fastp -L -Q -G -A -z 1 --stdout -w 1 -i SRR12342877.fastq -h SRR12342877.fastq.html > /dev/null`

## Проведём обработку прочтений SRR12342876 аналогично предыдущим данным

`mkdir preprocess`

`cd preprocess`

`bbduk.sh Xmx=4G t=1 ref=../sr.adapters.fasta k=19 ktrim=r qtrim=r trimq=16 entropy=0.5 minlength=33 in=../SRR12342876_1.fastq in2=../SRR12342876_2.fastq out=pe_1.fq out2=pe_2.fq stats=bbduk.stats.pe.txt`

`bbmerge.sh Xmx=4G t=1 in1=pe_1.fq in2=pe_2.fq outu1=nm.pe_1.fq.gz outu2=nm.pe_2.fq.gz out=merged.fq.gz`

## Выполним сборку только парынх прочтений

`cd ..`

`spades.py -1 preprocess/nm.pe_1.fq.gz -2 preprocess/nm.pe_2.fq.gz --merged preprocess/merged.fq.gz -o spades-sr -t 2 -m 8`

## Выполним сборку только длинных прочтений

`unicycler -l SRR12342877.fastq -o unicycler-long -t 2`

`flye --nano-raw SRR12342877.fastq --genome-size XXX --threads 2 --out-dir flye`

## Выполним гибридную сборку

`unicycler -1 preprocess/nm.pe_1.fq.gz -2 preprocess/nm.pe_2.fq.gz -s preprocess/merged.fq.gz -l SRR12342877.fastq -o unicycler -t 2`

## Выполним гибридную сборку - II

### Оцениваем размер генома

`mash sketch -r -m 5 SRR12342877.fastq`

### Проводим сборку без коррекции длинными прочтениями

`flye --nano-raw SRR12342877.fastq --genome-size XXX --threads 2 --out-dir flye-polishing --stop-after contigger`

### Проводим картирование прочтений

`minimap2 -x sr -o flye.0.paf flye-polishing/30-contigger/contigs.fasta preprocess/merged.fq.gz`

### Получаем консенсус

`racon preprocess/merged.fq.gz flye.0.paf flye-polishing/30-contigger/contigs.fasta > flye.1.fna`

### Повторяем

`minimap2 -x sr -o flye.1.paf flye.1.fna preprocess/merged.fq.gz`

`racon preprocess/merged.fq.gz flye.1.paf flye.1.fna > flye.2.fna`

## Сравним результаты

`mkdir contigs`

`cp spades-sr/contigs.fasta contigs/spades-sr.fna`

`cp unicycler-long/assembly.fasta contigs/unycycler-l.fna`

`cp unicycler/assembly.fasta contigs/unycycler-h.fna`

`cp flye/assembly.fasta contigs/flye-l.fna`

`cp flye.2.fna contigs.flye-polished.fna`

### Оценим количественные характеристики сборки

`quast.py contigs/*fna`

### Оценим полноту сборок

`checkm taxonomy_wf -f checkm.txt genus Bacillus contigs checkm`

# Аннотация геномов

XXX - путь к лучшему геному по нашей оценке

`dfast --cpu 1 --center_name TPA --organism "Bacillus cereus" -g XXX -o dfast-annotation`

