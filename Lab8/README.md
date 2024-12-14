# Практическая работа в DuckDB

artem.ilin.03@yandex.ru

## Цель работы
1. Изучить возможности СУБД DuckDB для обработки и анализа больших данных
2. Получить навыки применения DuckDB совместно с языком программирования R
3. Получить навыки анализа метаинфомации о сетевом трафике
4. Получить навыки применения облачных технологий хранения, подготовки и анализа данных: Yandex Object Storage, Rstudio Server

## Исходные данные

1. Yandex Object Storage
2. RStudio Server

## План

1. Подключиться к RStudio Server
2. Выполнить задания


## Шаги

### 1. Скачивание пакета данных и загрузка их в таблицу

1. Скачивание пакета данных
```r
#download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")
```
2. Подключение библиотек и выгрузка данных в таблицу
```r 
library(duckdb)
library(dplyr)

con <- dbConnect(duckdb())
dbExecute(con, 
    "CREATE TABLE dat as 
    SELECT * 
    FROM read_parquet('tm_data.pqt')"
)
```
```r
105747730
```
### 2. Выполнение заданий
 
Задание 1. Найдите утечку данных из вашей сети
```r
dbGetQuery(con,
    "SELECT 
        src 
        FROM dat
    WHERE (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%') 
    AND NOT (dst LIKE '12.%' AND dst LIKE '13.%' AND dst LIKE '14.%')
    GROUP BY src
    ORDER by sum(bytes) DESC"
) %>% knitr::kable()
```
```{r}
13.37.84.125
```
Задание 2. Найдите утечку данных 2
```r
dbGetQuery(con,
    "SELECT 
        hours,
        COUNT(*) AS trafictime
        FROM (
            SELECT 
                timestamp,
                src,
                dst,
                bytes,
                (
                    (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
                    AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
                ) AS trafic,
                EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS hours
            FROM dat
        ) sub
    WHERE trafic = TRUE AND hours BETWEEN 0 AND 24
    GROUP BY hours;"
) %>% knitr::kable()
```
```{r}
    hours count_star()
1     0       169068
2     1       168539
3     2       168711
4     3       169050
5     4       168422
6     5       168283
7     6       169015
8     7       169241
9     8       168205
10    9       168283
11   10       168750
12   11       168684
13   12       168892
14   13       169617
15   14       169028
16   15       168355
17   16      4490576
18   17      4483578
19   18      4489386
20   19      4487345
21   20      4482712
22   21      4487109
23   22      4489703
24   23      4488093
```

```r 
dbGetQuery(con,
    "SELECT src
    FROM (
        SELECT src, SUM(bytes) AS total_bytes
        FROM (
            SELECT *,
                EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS hours
            FROM tbl
        ) sub
        WHERE src <> '13.37.84.125'
            AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
            AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
            AND hours BETWEEN 1 AND 15
        GROUP BY src
    ) grp
    ORDER BY total_bytes DESC;"
) %>% knitr::kable()
```
```{r}
12.55.77.96
```
Задание 3. Найдите утечку данных 3
```r
dbExecute(con,
    "CREATE TEMPORARY TABLE tab AS
    SELECT 
        src, 
        bytes, 
        port
    FROM dat
    WHERE src <> '13.37.84.125'
        AND src <> '12.55.77.96'
        AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
        AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%');"
)
```
```{r}
38498353
```
```r 
dbGetQuery(con,
    "SELECT 
        port, 
        AVG(bytes) AS mean_bytes, 
        MAX(bytes) AS max_bytes, 
        SUM(bytes) AS sum_bytes, 
        MAX(bytes) - AVG(bytes) AS dif
    FROM tab
    GROUP BY port
    HAVING MAX(bytes) - AVG(bytes) != 0
    ORDER BY dif DESC;"
) %>% knitr::kable()
```
```{r}
14.31.107.42
```

## Оценка результата

Был установлен и проанализирован пакет данных - tm_data и выполнены поставленные задачи

## Вывод

Были изучены возможности СУБД DuckDB для обработки и анализа больших данных, а также получены навыки применения DuckDB совместно с языком программирования R
```{r}
```