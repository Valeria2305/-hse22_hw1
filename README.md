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
- Оценка качества исходных чтений методом FastQC:
```
mkdir fastqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
```
- Создание общей статистики при помощи multiQC:
```
mkdir multiqc
multiqc -o multiqc fastqc
```
<img width="711" alt="image" src="https://user-images.githubusercontent.com/77625525/193105953-d1727a92-1372-4d40-bb67-434fd61669ee.png">
<img width="695" alt="image" src="https://user-images.githubusercontent.com/77625525/193210991-558c5351-8615-49d0-950b-88658ea5aec8.png">
<img width="704" alt="image" src="https://user-images.githubusercontent.com/77625525/193210932-ec381634-5da8-4e16-af6b-0cc1d85640d4.png">

- Подрезание чтений по качеству и удаление адаптеров:
```
platanus_trim sub*
platanus_internal_trim matep*
```
- Удаление исходных файлов:
```
rm sub1.fastq
rm sub2.fastq
rm matep1.fastq 
rm matep2.fastq
```
- Оценка качества обрезанных чтений методом FastQC:
```
mkdir fastqc_trimmed
ls sub* matep*| xargs -tI{} fastqc -o fastqc_trimmed {}
```
- Создание общей статистики при помощи multiQC:
```
mkdir multiqctrimmed
multiqc -o multiqctrimmed fastqc_trimmed
```
<img width="721" alt="image" src="https://user-images.githubusercontent.com/77625525/193212463-21014e93-a3f3-4103-8978-2967cb273e31.png">
<img width="696" alt="image" src="https://user-images.githubusercontent.com/77625525/193212683-e1faadab-650f-4b2b-86df-5b4114f1b386.png">

- Сбор контиг c обрезанных чтений при помощи "platanus assemble":
```
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```
-  Сбор скаффолдов при помощи "platanus scaffold":
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> scaffold.log
```
- Уменьшение промежутков:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed  matep2.fastq.int_trimmed 2> gapclose.log
```
- Удаление файлов с подрезанными чтениями:
```
rm matep1.fastq.int_trimmed
rm matep2.fastq.int_trimmed
rm sub1.fastq.trimmed
rm sub2.fastq.trimmed
```
