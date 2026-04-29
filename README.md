# COVID Lab Journal
**Проект 1: Идентификация SARS-CoV-2 из метагеномных данных секвенирования**  
**Курс: Анализ геномных данных**

---

##  Скачивание данных

Скачал предварительно собранные скаффолды из Google Drive.

```bash
mkdir ~/covid_project
cd ~/covid_project
gdown 1PU6gQiF2CvxYhmWxVCWuESnG1XCZvibf
ls -lh
```

Получен файл: spades_scaffolds.fasta (31MB).

---

##  Анализ сборки (QUAST)

```bash
conda install quast -c bioconda -c conda-forge -y
quast spades_scaffolds.fasta -o quast_out
cat quast_out/report.txt
```

**Результаты:**

| Параметр | Значение |
|----------|---------|
| Всего контигов | 111 219 |
| Контигов >= 1000 п.н. | 666 |
| Наибольший контиг | 29 907 п.н. |
| N50 | 779 п.н. |
| GC-состав | 45.92% |

---

## 29.04.2026 — Поиск вирусного контига

Проиндексировал файл и посмотрел топ контигов по длине и покрытию.

```bash
samtools faidx spades_scaffolds.fasta
sort -k2 -rn spades_scaffolds.fasta.fai | head -3
```

**Результат:**

NODE_1_length_29907_cov_150.822528   29907   coverage 150x   → главный кандидат (SARS-CoV-2)
NODE_2_length_8134_cov_6.182822      8134    coverage 6x
NODE_3_length_5584_cov_13759.613946  5584    coverage 13759x → phiX174 (контроль Illumina)
NODE_1 длиной ~30 000 п.н. — подозрительно похож на геном коронавируса.  
NODE_3 с аномально высоким покрытием — стандартный контрольный фаг phiX174.

---

##  Извлечение кандидата и BLAST

```bash
samtools faidx spades_scaffolds.fasta NODE_1_length_29907_cov_150.822528 > candidate.fa
```

Загрузил candidate.fa на https://blast.ncbi.nlm.nih.gov/  
Параметры:
- Database: Core nucleotide database
- Organism: Viruses (taxid:10239)
- Entrez Query: `1900/01/01:2020/01/01[PDAT]`
- Algorithm: Megablast

**Результат BLAST:**

| Параметр | Значение |
|----------|---------|
| Лучший хит | bat-SL-CoVZC45 (bat SARS-like coronavirus) |
| % Identity | 89.12% |
| E-value | 0.0 |
| Хозяин | Rhinolophus affinis (подковоносая летучая мышь) |

Вирус новый — 89% идентичности с известным коронавирусом летучих мышей.

---

##  Аннотация генома (Prokka)

Загрузил candidate.fa на https://usegalaxy.eu/  
Запустил инструмент Prokka с параметрами по умолчанию.

**Результат (из файла txt):**
organism: Genus species strain
contigs: 1
bases: 29907
CDS: 9

**Предсказанные CDS (из файла tsv):**

| Локус | Длина (п.н.) | Продукт |
|-------|-------------|---------|
| DFCKDBFJ_00001 | 1 260 | Гипотетический белок |
| DFCKDBFJ_00002 | 366 | Гипотетический белок |
| DFCKDBFJ_00003 | 366 | Гипотетический белок |
| DFCKDBFJ_00004 | 186 | Гипотетический белок |
| DFCKDBFJ_00005 | 669 | Гипотетический белок |
| DFCKDBFJ_00006 | 828 | Гипотетический белок |
| DFCKDBFJ_00007 | 3 822 | Гипотетический белок |
| DFCKDBFJ_00008 | 7 788 | Гипотетический белок |
| DFCKDBFJ_00009 | 13 218 | O-acetyl-ADP-ribose deacetylase (ymdB) |

8 гипотетических белков — Prokka не смогла их аннотировать так как вирус новый.  
DFCKDBFJ_00009 соответствует макродомену nsp3 в составе ORF1ab.  
Для сравнения: в GenBank (NC_045512) аннотировано ~12 белков включая S, E, M, N.
