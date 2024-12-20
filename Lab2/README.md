
# Основы обработки данных с помощью R и Dplyr


## Цель работы
Развить практические навыки использования функций обработки данных пакета dplyr – функции select(), filter(), mutate(), arrange(), group_by()

## Исходные данные

1.  Программное обеспечение Windows 11

2.  Rstudio Desktop

3.  Программный пакет dplyr

## План

1.  Установить программный пакет dplyr

2.  Проанализировать набор данных

3.  Ответить на вопросы

## Шаги

### 1. Установка dplyr
```{r}         
    install.packages("dplyr")
```

### 2. Подключение dplyr
```{r}
    library(dplyr)
``` 

### 3. Ответы на вопросы

a\) Сколько строк в датафрейме?

```r
    starwars %>% nrow()
```

b\) Сколько столбцов в датафрейме?

```r
    starwars %>% ncol()
```

c\) Как просмотреть примерный вид датафрейма?

```r
    starwars %>% glimpse()
```

d\) Сколько уникальных рас персонажей (species) представлено в данных?

```r
    starwars %>% distinct(species)

```

e\) Найти самого высокого персонажа.

```r
    max_values <- starwars 
```

f\) Найти всех персонажей ниже 170

```r
    starwars %>% filter(!is.na(height) & height < 170) 
    %>% select(name,height)
```

g\) Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ подсчитать по формуле

```r
    starwars %>% filter(!is.na(mass) & !is.na(height)) 
    %>% mutate(bmi = mass / (height/100)^2) 
    %>% select(name,bmi)

```

h\) Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

```r
    starwars %>% filter(!is.na(mass) & !is.na(height)) 
    %>% mutate(stretch = mass / height) 
    %>% arrange(desc(stretch)) 
    %>% slice(1:10) 
    %>% select(name,stretch)
```

i\) Найти средний возраст персонажей каждой расы вселенной Звездных войн.

```r
    starwars %>% filter(!is.na(species) & !is.na(birth_year)) 
    %>% group_by(species) 
    %>% summarise(average_age = mean(birth_year, na.rm = TRUE))
```

j\)Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

```r
    starwars %>% filter(!is.na(eye_color)) 
    %>% group_by(eye_color)
    %>% summarise(count = n()) 
    %>% arrange(desc(count)) 
    %>% slice(1)
```

k\) Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

```r
    starwars %>% filter(!is.na(species) & !is.na(name)) 
    %>% mutate(name_length = nchar(name)) 
    %>% group_by(species) 
    %>% summarise(len = mean(name_length, na.rm = TRUE))
```

## Оценка результата

В результате работы была установленна библиотека dplyr и были выполнены ответы на вопросы с использованием набора данных starwars.

## Вывод

Были изучены и применены функции библиотеки dplyr на наборе данных starwars