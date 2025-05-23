# Определение

# Функция

# Объект воздействия

[[loops]]
# Как

## Операторы break и continue
Оператор `break` выполняет прерывание **ближайшего** цикла в котором он расположен. Аналогично, оператор `continue` осуществляет переход на следующую итерацию **ближайшего** цикла.

Если необходимо добиться прерывания внешнего цикла из-за выполнения условия во внутреннем, то стоит сделать это через **сигнальную метку**.
# Особенности/атрибуты
- вложенный цикл выполняет все свои итерации для каждой отдельной итерации внешнего цикла;
- вложенные циклы завершают свои итерации быстрее, чем внешние циклы;
- для того, чтобы получить общее количество итераций вложенного цикла, надо **перемножить** количество итераций всех циклов


**Таблица использования вложенных циклов `for`:**

| Ситуация                                  | Применение вложенных циклов `for`                                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Работа с двумерными массивами (матрицами) | Используется для итерации по строкам и столбцам матрицы.                                                                       |
| Генерация всех возможных комбинаций       | Применяется, когда нужно сгенерировать все возможные комбинации из нескольких списков или наборов данных.                      |
| Поиск вложенных элементов                 | Полезен для поиска элементов вложенных списков или структур данных.                                                            |
| Итерация по словарям вложенных объектов   | Используется для доступа к значениям вложенных словарей или объектов.                                                          |
| Многомерные массивы                       | Подходит для работы с многомерными массивами, где каждый элемент также является массивом.                                      |
| Рекурсивные операции                      | Может применяться для рекурсивных операций, где итерация должна выполняться на нескольких уровнях вложенности.                 |
| Генерация сетки координат                 | При создании сетки координат (например, для графиков) вложенные циклы используются для генерации всех возможных пар координат. |
# Примеры

```python
for i in range(3):
	for j in range(5):
		print('*')

output:
*
*
*
...
```

```python
for i in range(3):
	for j in range(5):
		print('*', end='')

output:
*** ...
```

```python
for i in range(3):
	for j in range(5):
		print('*', end='')
	print()  # Перенос результата на новую строку, после всех прохода всех итераций во вложенном скрипте. 'Таблица' 3x5

output:
*****
*****
*****
```

```python
for i in range(3):
	for j in range(5):
		print(i, end='')
	print()

output:
00000
11111
22222
```

```python
for i in range(3):
	for j in range(5):
		print(j, end='')
	print()

output:
01234
01234
01234
```

```python
for i in range(3):
	for j in range(i):
		print(j, end='')
	print()

output:
0
01
```


# Частые ошибки

# Решение задач

**Задача 2.** Найдите все тройки натуральных чисел (и их количество), являющихся решением уравнения x2+y2+z2=2020

**Решение.**  Поскольку по условию числа x,y, и z являются натуральными, то x,y,z<2020≈45. Напишем программу, которая перебирает всевозможные тройки чисел (x;y;z) и проверяет для них условие x2+y2+z2=2020.

```python
total = 0
for x in range(1, 45):
    for y in range(1, 45):
        for z in range(1, 45):
            if x ** 2 + y ** 2 + z ** 2 == 2020:
                total += 1
                print('x =', x, 'y =', y, 'z =', z)
print('Общее количество натуральных решений =', total)

output:
x = 18 y = 20 z = 36
x = 18 y = 36 z = 20
x = 20 y = 18 z = 36
x = 20 y = 36 z = 18
x = 36 y = 18 z = 20
x = 36 y = 20 z = 18
Общее количество натуральных решений = 6
```