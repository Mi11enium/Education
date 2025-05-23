# Description
<mark style="background: #FFB86CA6;">тип данных</mark> в Python, <mark style="background: #FFB86CA6;">предназначенный для хранения целых чисел</mark> (как положительных, так и отрицательных). <mark style="background: #FFB86CA6;">Числа без дробной части</mark>.

# Action
Предназначен для выполнения математических операций, подсчёта, итераций, логики, а также для представления дискретных данных.
# Object
Целые числа, такие как 0, 1, -5, 1000. 
<mark style="background: #FFB86CA6;">Все объекты типа</mark> `int` <mark style="background: #FFB86CA6;">неизменяемы (immutable)</mark>.


# How does it work (Logic)
- Python автоматически интерпретирует числа без десятичной точки как тип `int`.
- `int` может быть использован в выражениях для выполнения математических операций.
- Размер числа ограничен только памятью компьютера.
# Features/Attributes
- Представляет числа без дробной части.
- Поддерживает стандартные операции: сложение, вычитание, умножение, деление (результат деления может быть `float`).
- Не имеет фиксированного диапазона (в отличие от некоторых других языков программирования).
- Может быть преобразован в строку (`str`) или число с плавающей точкой (`float`).

# Syntax
```python
# Присваивание int
number = 42
negative_number = -10
```
# Examples
```python

# Арифметические операции
x = 10
y = 5
sum_result = x + y      # 15
diff_result = x - y     # 5
mult_result = x * y     # 50
div_result = x // y     # 2 (целочисленное деление)

# Преобразование типов
float_number = float(x)  # 10.0
string_number = str(x)   # "10"

# Использование в логике
if x > 5:
    print("x больше 5")

# Работа с большими числами
large_number = 10**100
print(large_number)  # 1 с 100 нулями
```

# Common mistakes
- 1. **Попытка деления без обработки типа**:
    - Деление `/` возвращает `float`, даже если деление нацело.
    ```python
    result = 10 / 5  # Результат: 2.0, а не 2
    ```
- 2. **Попытка изменения самого числа**:
    - Целые числа неизменяемы. Пример:
    ```python
    x = 5 x += 1  # Создаётся новый объект, а не изменяется существующий.
    ```
- 3. **Использование числа как строки без преобразования**:
    ```python
x = 10 print("Результат: " + x)  # Ошибка! Строку нельзя объединить с int.
```
- **Игнорирование порядка операций**:
    ```python
result = 2 + 3 * 4  # Результат: 14, а не 20 (умножение приоритетнее).
```
- **Попытка использовать int с неполным преобразованием**:
 ```python
int("10.5")  # ValueError, так как строка не является целым числом.
```