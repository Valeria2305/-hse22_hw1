# hse22_hw1
## Обязательная часть 
- Создание ссылок для доступа к файлам:
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```
- Создание random seed = 523 и выбор случайных 5.000.000 чтений типа paired-end и 1.500.000 чтений типа mate-pairs:
```
seqtk sample -s523 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s523 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s523 oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s523 oilMP_S4_L001_R2_001.fastq 1500000 > matep2.fastq
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
- Общая функция для подсчета:
```
def counter(f, text, outfile = True):
    lengths = []
    total_len = 0
    num = 0
    max_len = 0
    length = 0
    flag = 0
    max_seq = ''
    curr_seq = ''
    for line in f:
        if (line[0] == '>'):
            if num != 0:
                lengths.append(length)
            num += 1
            if length >= max_len:
                max_len = length
                max_seq = curr_seq
            curr_seq = ''
            length = 0
        else:
            curr_seq += line.strip()
            length += len(line.strip())
            total_len += len(line.strip())
     
    lengths.sort(reverse = True) 
    for i in lengths:
        flag += i
        if flag >= total_len / 2:
            if outfile == True:
                print(f'Анализ {text}\n\
Общее количество: {num},\n\
Общая длина: {total_len},\n\
Длина самого длинного: {max_len},\n\
N50: {i}\n')
            break
    return max_seq
```
- Континги:
```
max_cont = get_info(open('Poil_contig.fa', 'r'), 'Контигов')
```
Анализ Контигов<br />
Общее количество: 610,<br />
Общая длина: 3924012,<br />
Длина самого длинного: 179307,<br />
N50: 53980<br />
- Скаффолды:
```
max_scaffolds = counter(open('Poil_scaffold.fa', 'r'), 'Скаффолдов')
```
Анализ Скаффолдов<br />
Общее количество: 68,<br />
Общая длина: 3876321,<br />
Длина самого длинного: 3832011,<br />
N50: 3832011<br />
- Количество гэпов:
```
print(f'Общая длина гэпов: {max_scaffolds.count("N")}')
max_scaffolds = re.sub(r'N{2,}', 'N', max_scaffolds)
print(f'Число гэпов: {max_scaffolds.count("N")}')
```
Общая длина гэпов: 6812<br />
Число гэпов: 65<br />
- Количество гэпов для уменьшенного числа:
```
max_scaffolds = counter(open('Poil_gapClosed.fa', 'r'), 'Скаффолдов', False)
print(f'Общая длина гэпов для обрезанных чтений: {max_scaffolds.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaffolds)
print(f'Число гэпов для обрезанных чтений: {max_scaf.count("N")}')
```
Общая длина гэпов для обрезанных чтений: 2409<br />
Число гэпов для обрезанных чтений: 10<br />

## Дополнительная часть

- Уменьшим число чтений в 10 раз. Для paired-end 500000 чтений, для mate-pairs 150000 чтений.
- Проделаем те же действия, что и в первой части. 
- Общая статистика multiQC для исходных чтений:

<img width="711" alt="image" src="https://user-images.githubusercontent.com/77625525/193671896-d29b5b6b-b638-4a1a-9b2c-ad55f3fa674a.png">
<img width="692" alt="image" src="https://user-images.githubusercontent.com/77625525/193672124-5f915c4a-a987-4a8d-85ae-135405b80522.png">
- Общая статистика multiQC для обрезанных чтений:

<img width="705" alt="image" src="https://user-images.githubusercontent.com/77625525/193673308-f3e24fd4-4d97-4c56-9a65-dd136e883bfe.png">
<img width="686" alt="image" src="https://user-images.githubusercontent.com/77625525/193673667-a6466ae0-8ec1-409f-a6d7-be8e0c00781b.png">
- Общая функция для подсчета:

```
def counter(f, text, outfile = True):
    lengths = []
    total_len = 0
    num = 0
    max_len = 0
    length = 0
    flag = 0
    max_seq = ''
    curr_seq = ''
    for line in f:
        if (line[0] == '>'):
            if num != 0:
                lengths.append(length)
            num += 1
            if length >= max_len:
                max_len = length
                max_seq = curr_seq
            curr_seq = ''
            length = 0
        else:
            curr_seq += line.strip()
            length += len(line.strip())
            total_len += len(line.strip())
     
    lengths.sort(reverse = True) 
    for i in lengths:
        flag += i
        if flag >= total_len / 2:
            if outfile == True:
                print(f'Анализ {text}\n\
Общее количество: {num},\n\
Общая длина: {total_len},\n\
Длина самого длинного: {max_len},\n\
N50: {i}\n')
            break
    return max_seq
```
- Континги:

```
max_contigs = counter(open('Poil_contig.fa', 'r'), 'Контигов')
```
Анализ Контигов<br />
Общее количество: 3425,<br />
Общая длина: 3915715,<br />
Длина самого длинного: 30735,<br />
N50: 4024<br />

- Скаффолды:

```
max_scaffolds = counter(open('Poil_scaffold.fa', 'r'), 'Скаффолдов')
```
Анализ Скаффолдов<br />
Общее количество: 473,<br />
Общая длина: 3869539,<br />
Длина самого длинного: 1286249,<br />
N50: 863189<br />

- Число гэпов:

```
print(f'Общая длина гэпов: {max_scaffolds.count("N")}')
max_scaffolds = re.sub(r'N{2,}', 'N', max_scaffolds)
print(f'Число гэпов: {max_scaffolds.count("N")}')
```
Общая длина гэпов: 26056<br />
Число гэпов: 519<br />

- Количество гэпов для обрезанного числа:

```
max_scaffolds = counter(open('Poil_gapClosed.fa', 'r'), 'Скаффолдов', False)
print(f'Общая длина гэпов для обрезанных чтений: {max_scaffolds.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaffolds)
print(f'Число гэпов для обрезанных чтений: {max_scaf.count("N")}')
```
Общая длина гэпов для обрезанных чтений: 11379<br />
Число гэпов для обрезанных чтений: 43<br />

## Вывод 
При изменении числа чтений до 10%: <br />
- Континги: 
1. Общее число контингов увеличилось с 610 до 3425
2. Общая длина изменилась незначительно: c 3924012 до 3915715
3. Длина самого длинного в разы уменьшилась: c 179307 до 30735
4. Показатель N50 изменился примерно в 13 раз: с 53980 до 4024
- Скаффолды: 
1. Общее число увеличилось в несколько раз: c 68 до 473
2. Общая длина изменилась незначительно: c 3876321 до 3869539
3. Длина самого длинного уменьшилась примерно в 2.5 раза: c 3832011 до 1286249
4. Показатель N50 изменился примерно в 4 раза: с 3832011 до 863189
- Количество гэпов:
1. Общая длина увеличилась: с 6812 до 26056
2. Число гэпов увеличилось в несколько раз: c 65 до 519
- Число гэпов для обрезанных даннных:
1. Общая длина увеличилась: с 2409 до 11379
2. Число гэпов увеличилось: c 10 до 43

## ССЫЛКА НА GOOGLE COLAB: 
https://colab.research.google.com/drive/1_Do0LP-mDxFZJBwWL_bUMDiw94r4sz83?usp=sharing
