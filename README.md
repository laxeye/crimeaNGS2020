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
