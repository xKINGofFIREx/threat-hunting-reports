# Анализ данных сетевого трафика
artem.ilin.03@yandex.ru

## Цель работы
1. Изучить возможности технологии Apache Arrow для обработки и анализа больших данных
2. Получить навыки применения Arrow совместно с языком программирования R
3. Получить навыки анализа метаинфомации о сетевом трафике
4. Получить навыки применения облачных технологий хранения, подготовки и анализа данных: Yandex Object Storage, Rstudio Server.

## Исходные данные

1. Yandex Object Storage
2. Rstudio Server

## План

1. Импотировать данные
2. Выполнить задания 


## Шаги

### 1. Импортирование данных

```r 
library(arrow)
library(tidyverse)

download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")

dataset <- arrow::open_dataset(sources = "tm_data.pqt", format = "parquet")
```

### 2. Выполнение заданий
 
Задание 1. Найдите утечку данных из вашей сети
```r
findLeak <- dataset %>% filter(
    str_detect(src, "^12.") 
    | str_detect(src, "^13.") 
    | str_detect(src, "^14."))  
    %>% filter(
            !str_detect(dst, "^12.") 
            & !str_detect(dst, "^13.") 
            & !str_detect(dst, "^14.")
        )  
        %>% group_by(src) 
        %>% summarise("sum" = sum(bytes)) 
        %>% arrange(desc(sum)
) 
%>% head(1) 
%>% select(src) 


findLeak %>% collect() 
%>% knitr::kable()
```
```{r}
|src          |
|:------------|
|13.37.84.125 |
```
Задание 2. Найдите утечку данных 2
```r
findLeak2 <- dataset %>% 
select(timestamp, src, dst, bytes) 
%>% mutate(
    trafic = (
        str_detect(src, "^((12|13|14)\\.)") 
        & !str_detect(dst, "^((12|13|14)\\.)")
    ),
    hours = hour(as_datetime(timestamp / 1000))
) 
%>% filter(trafic == TRUE, hours >= 0 & hours <= 24) 
%>% group_by(hours) 
%>% summarise(trafictime = n()) 


findLeak2 %>% collect()
%>% arrange(desc(hours))
%>% knitr::kable()
```
```{r}
| hours| trafictime|
|-----:|----------:|
|    23|    4488093|
|    22|    4489703|
|    21|    4487109|
|    20|    4482712|
|    19|    4487345|
|    18|    4489386|
|    17|    4483578|
|    16|    4490576|
|    15|     168355|
|    14|     169028|
|    13|     169617|
|    12|     168892|
|    11|     168684|
|    10|     168750|
|     9|     168283|
|     8|     168205|
|     7|     169241|
|     6|     169015|
|     5|     168283|
|     4|     168422|
|     3|     169050|
|     2|     168711|
|     1|     168539|
|     0|     169068|
```
На полученных данных видно, что саммыми загруженными являются часы с 16 до 23

```r 
findLeak3 <- dataset %>% 
mutate(
    hours = hour(as_datetime(timestamp / 1000))
) 
%>% filter(
    !str_detect(src, "^13.37.84.125")
) 
%>% filter(
    str_detect(src, "^12.") 
    | str_detect(src, "^13.") 
    | str_detect(src, "^14.")
)  
%>% filter(
    !str_detect(dst, "^12.") 
    | !str_detect(dst, "^13.") 
    | !str_detect(dst, "^14.")
)  
%>% filter(hours >= 1 & hours <= 15) 
%>% group_by(src) 
%>% summarise("sum" = sum(bytes)) 
%>% arrange(desc(sum)) 
%>% select(src) 

findLeak3 %>% collect()
%>% head(1)
%>% knitr::kable()
```

```{r}
|src         |
|:-----------|
|12.55.77.96 |
```

Задание 3. Найдите утечку данных 3

```r 
findLeak4 <- dataset 
%>% filter(!str_detect(src, "^13.37.84.125")) 
%>% filter(!str_detect(src, "^12.55.77.96")) 
%>% filter(
    str_detect(src, "^12.") 
    | str_detect(src, "^13.") 
    | str_detect(src, "^14.")
)  
%>% filter(
    !str_detect(dst, "^12.") 
    & !str_detect(dst, "^13.") 
    & !str_detect(dst, "^14.")
)  
%>% select(src, bytes, port)

findLeak4 %>% group_by(port) 
%>% summarise(
    "mean"=mean(bytes), 
    "max"=max(bytes), 
    "sum" = sum(bytes)
) 
%>% mutate("dif" = max-mean)  
%>% filter(dif != 0) 
%>% arrange(desc(dif))  
%>% collect() 
%>% knitr::kable()
```

```{r}
| port|        mean|    max|         sum|         dif|
|----:|-----------:|------:|-----------:|-----------:|
|   37| 35089.98907| 209402| 32136394510| 174312.0109|
```

```r 
findLeak5 <- findLeak4  
%>% filter(port == 37) 
%>% group_by(src) 
%>% summarise("mean"=mean(bytes)) 
%>% arrange(desc(mean)) 
%>% select(src)

findLeak5 %>% collect()
%>% head(1)
%>% knitr::kable()
```

```{r}
|src          |
|:------------|
|14.31.107.42 |
```

## Оценка результата

Был установлен и проанализирован пакет данных - tm_data и выполнены поставленные задачи

## Вывод

Были изучены возможности технологии Apache Arrow для обработки и анализа больших данных и получены навыки применения Arrow совместно с языком программирования R
