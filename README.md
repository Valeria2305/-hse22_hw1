# hse22_hw1
## Обязательная часть 
- Создание ссылок для доступа к файлам:
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
```
- Создание random seed = 523 и выбор случайных 5.000.000 чтений типа paired-end и 1.500.000 чтений типа mate-pairs:
```
seqtk sample -s523 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s523 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s523 oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s523 oilMP_S4_L001_R1_002.fastq 1500000 > matep2.fastq
```
- Оценка количества исходных чтений методом FastQC:
```
mkdir fastqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
```
- Оценка количества исходных чтений методом multiQC:
```
mkdir multiqc
multiqc -o multiqc fastqc
```
