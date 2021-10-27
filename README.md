## Домашнее задание №1
### Абу Аль Лабан Надя, группа 2 

Целью данного задания является сборка генома бактерии, выделенной из воды с нефтью, на основании парно-концевых (paired-end, PE) и чтений типа mate-pairs (MP).

Описание использованных команд
---
#### 1. Создаем папку для ДЗ 1 и заходим в нее

```
$ mkdir hw1  
$ cd hw1
```

---
#### 2. Создаем символические ссылки для нужных файлов

```
$ ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq  
$ ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
$ ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
$ ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

---
#### 3. Выбираем чтения
Случайным образом с зерном 0403

Сначала 5 миллионов чтений типа paired-end:  

```
$ seqtk sample -s0403 oil_R1.fastq 5000000 > random_R1.fq  
$ seqtk sample -s0403 oil_R2.fastq 5000000 > random_R2.fq 
```

Затем 1.5 миллиона чтений типа mate-pairs:

```
$ seqtk sample -s0403 oilMP_S4_L001_R1_001.fastq 1500000 > random_MP_R1.fq 
$ seqtk sample -s0403  oilMP_S4_L001_R2_001.fastq 1500000 > random_MP_R2.fq
```

---
#### 4. Оцениваем качество чтений с помощью fastQC

```
$ ls random_* | xargs -tI{} fastqc {} 
```

Создаем папку и переносим туда результат
```
$ mkdir fastQC
$ mv *.zip *.html fastQC/
```

---
#### 5. Создааем папку и делаем единый отчет с помощью multiQC

```
$ mkdir multiQC 
$ multiqc -o multiQC fastQC
```

---
#### 6. Подрезаем чтения по качеству

```
$ platanus_internal_trim random_MP_R1.fq random_MP_R2.fq  
$ platanus_trim random_R1.fq random_R2.fq
```
И удаляем исходники

```
$ rm *.fq
```

---
#### 7. Делаем отчеты по подрезанным чтениям

Создаем папки для результата

```
$ mkdir trimmed_fastQC 
$ mkdir trimmed_multiQC
```
И делаем сами отчеты
```
$ ls random_* | xargs -tI{} fastqc -o trimmed_fastQC {}
$ multiqc -o trimmed_multiQC trimmed_fastQC
```

---
#### 8. Собираем континги из подрезанных чтений

```
$ time platanus assemble -f random_R1.fq.trimmed random_R2.fq.trimmed 2> assemble.log
```

---
#### 8. Собираем скаффолды из контигов, а также из подрезанных чтений

```
$ time platanus scaffold -c out_contig.fa -IP1 random_R1.fq.trimmed random_R2.fq.trimmed -OP2 random_MP_R1.fq.int_trimmed random_MP_R2.fq.int_trimmed 2> scaffold.log
```

#### 9. Для самого длинного скаффолда считаем количество гэпов и их общую длину

```
$ head -1 out_scaffold.fa 
```
Затем по выводу:
```
$ echo scaffold1_len3833967_cov231 > longest_with_gaps.txt
$ seqtk subseq out_scaffold.fa longest_with_gaps.txt > longest_with_gaps.fa
```


#### 10. Уменьшаем гэпы
```
$ time platanus gap_close -c out_scaffold.fa -IP1 random_R1.fq.trimmed random_R2.fq.trimmed -OP2 random_MP_R1.fq.int_trimmed random_MP_R2.fq.int_trimmed 2> gap.log
```

#### 11. Для самого длинного скаффолда считаем количество гэпов и их общую длину

```
$ head -1 out_gapClosed.fa 
```
Затем по выводу:
```
$ echo scaffold1_cov231 > longest_with_closed_gaps.txt
$ seqtk subseq out_gapClosed.fa longest_with_closed_gaps.txt > longest_with_closed_gaps.fa
```
#### 12. Удаляем лишние файлы
```
$ rm *.trimmed
$ rm *.txt
```
Тетрадка с решением
---
Ссылка на [коллаб](https://colab.research.google.com/drive/1uoYn4tTkKsK52ueDclepkhlLNiURcRDu?usp=sharing)  

Отчет
---
#### 1. Общий отчет по исходным чтениям
![image](https://user-images.githubusercontent.com/23341597/139136025-32ce5244-0f60-432b-bacd-b831659fb458.png)
![image](https://user-images.githubusercontent.com/23341597/139136099-2f754cd8-9065-4edb-94c8-b21b56092b8c.png)
![image](https://user-images.githubusercontent.com/23341597/139136174-16e65343-7d46-4db3-98e8-f691b7d2ab44.png)
![image](https://user-images.githubusercontent.com/23341597/139136208-0fc7599f-3555-49ae-9f9c-cce25b2e380f.png)
![image](https://user-images.githubusercontent.com/23341597/139136511-57ca131d-4ab3-4924-839f-69658dda8328.png)
#### 2. Общий отчет по подрезанным чтениям
![image](https://user-images.githubusercontent.com/23341597/139137631-a8af18a9-8866-49af-ab15-b509073d91bd.png)
![image](https://user-images.githubusercontent.com/23341597/139137656-fe53738a-3696-4900-a7fa-7c4576f95e53.png)
![image](https://user-images.githubusercontent.com/23341597/139137697-55b69d8d-d762-46d3-afa0-0724c0afe96f.png)
![image](https://user-images.githubusercontent.com/23341597/139137732-d49c55fe-a62d-4601-a29c-a41bf36fe4e2.png)
![image](https://user-images.githubusercontent.com/23341597/139137767-774c998c-bdde-4412-a7ee-99ac8a2ef6b2.png)
