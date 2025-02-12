# Основы обработки данных с помощью R и Dplyr

## Цель работы

Развить практические навыки использования функций обработки данных пакета dplyr – функции select(), filter(), mutate(), arrange(), group_by()

## Исходные данные

1.  Программное обеспечение Windows 11

2.  Rstudio Desktop

3.  Программные пакеты nycflights13 и dplyr

## План

1.  Установить программный пакет nycflights13

2.  Проанализировать набор данных

3.  Ответить на вопросы

## Шаги

### 1. Установка программного пакета nycflights13

```{r}
    install.packages('nycflights13')
```
### 2. Подключение nycflights13 и dplyr.

```r
    library(nycflights13)
    library(dplyr)
```

### 3. Ответы на вопросы.

a\) Сколько встроенных в пакет nycflights13 датафреймов?
```r
    length(data(package="nycflights13")$results[, "Item"])
```
b\) Сколько строк в каждом датафрейме?

```r
    list(nrow(airlines), nrow(airports), nrow(flights), nrow(planes), nrow(weather))
```

c\) Сколько столбцов в каждом датафрейме?

```r
    list(ncol(airlines), ncol(airports), ncol(flights), ncol(planes), ncol(weather))
```

d\) Как просмотреть примерный вид датафрейма?

```r
    flights %>% glimpse()
```

e\) Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

```r
    flights %>% filter(!is.na(carrier)) 
    %>% distinct(carrier) 
    %>% nrow()
```

f\) Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

```r
    flights %>% filter(origin == "JFK", month == 5) 
    %>% nrow()
```

g\) Какой самый северный аэропорт?

```r
    airports %>% arrange(desc(lat)) 
    %>% slice(1)
```

h\) Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

```r
    airports %>% arrange(desc(alt)) 
    %>% slice(1)
```

i\) Какие бортовые номера у самых старых самолетов?

```r
    planes %>% arrange(desc(year)) 
    %>% select(tailnum, year)
```

g\) Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия).

```r
    weather %>% filter(origin == "JFK", month == 9) 
    %>% summarise(avg_temp = mean((temp - 32) * 5 / 9, na.rm = TRUE))
```

k\) Самолеты какой авиакомпании совершили больше всего вылетов в июне?

```r
    flights %>% filter(month == 6) 
    %>% count(carrier, sort = TRUE)
```

l\) Самолеты какой авиакомпании задерживались чаще других в 2013 году?

```r
    flights %>% filter(dep_delay > 0 & year == 2013) 
    %>% count(carrier, sort = TRUE)
```

## Оценка результата

В результате работы была использована библиотека nycflights13 и dplyr и были выполнены задания с использованием набора данных nycflights13

## Вывод

Были проанализированны данные набора nycflights13 и выполнены ответы на вопросы.