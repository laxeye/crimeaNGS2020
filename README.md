# crimeaNGS2020
Short course on genome assembly and annotation

https://docs.google.com/presentation/d/1DCtWySNo2XPIe_3eq772xl_j6F0srThFbTtExzNIDEU/edit?usp=sharing


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

# Аннотация геномов прокариот

## Поиск генов 16S рРНК

`cd ~/crimea/hybrid`

`mkdir annotation`

`cp contigs/flye.2.fna annotation/bcer.genome.fna`

`cd annotation`

`conda install easel barrnap`

`barrnap bcer.genome.fna > bcer.rrna.gff`

`less bcer.rrna.gff`

`barrnap bcer.genome.fna | grep 16S > bcer.16S.gff`

`esl-sfetch --index bcer.genome.fna`

`awk '{n++; print "16S_"n, $4, $5, $1}' bcer.16S.gff > bcer.16S.list`

`esl-sfetch -Cf bcer.genome.fna bcer.16S.list > bcer.16S.fna`

## Установление таксономии

https://blast.ncbi.nlm.nih.gov/

Nucleotide BLAST -> Database: rRNA/ITS databases - 16S ribosomal RNA

## Предсказание белок-кодирующих генов

`conda install prodigal`

`prodigal -i bcer.genome.fna -f gff -o bcer.cds.gff -a bcer.protein.faa -d bcer.cds.fna`

`grep -c ">" bcer.protein.faa`

## Аннотация при помощи DFAST

`dfast_file_downloader.py --protein dfast`

`dfast_file_downloader.py --cdd Cdd`

`dfast_file_downloader.py --hmm pfam`

`dfast_file_downloader.py --hmm TIGR`

`dfast -g bcer.genome.fna --cpu 1 -o bcer-dfast --organism "Bacillus sp."`

## Онлайн аннотация прокариот

**RAST** (Rapid Annotation using Subsystem Technology) i

https://rast.nmpdr.org/

**KAAS** (KEGG Automatic Annotation Server)

https://www.genome.jp/tools/kaas/

### Поиск вторичных метаболитов

https://antismash.secondarymetabolites.org/

### Поиск генов резистентности

**CARD** (The Comprehensive Antibiotic Resistance Database)

https://card.mcmaster.ca/analyze/rgi


# Пангеномный анализ

`for x in *fna; do prodigal -o /dev/null -i $x -a ${x/fna/faa}; done`

`proteinortho -singles --cpus=4 *faa`

`proteinortho -singles --cpus=4 *faa -p=blastp`

`cat myproject.proteinortho.tsv | grep -v "^#" | cut -f 4- | sed 's/,/\t/g' | sed 's/NC_00967[0-9].1/Bcyt/g' | sed 's/NZ_ABJC01000[0-9]\+.1/BantA0488/g' | sed 's/NZ_CP00[0-9]\+.1/BantVollum/g' | sed 's/NZ_CP03[0-9]\+.1/Bcer/g' | sed 's/contig_[1-4]/BNamibia/g' | sed 's/_/\|/g;s/\*\t//g; s/\t\*//g' > clusters.tsv`

https://orthovenn2.bioinfotoolkits.net/cluster-venn
