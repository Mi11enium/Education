# Определение
The 'for' loop in Python used for iterating over a sequence or an iterable object.

# Функция
 It allows you to execute a block of code repeatedly for each item in the sequence.
# Объект воздействия
[[List]], [[tuple]], [[String]], [[set]], [[Dictionary]], or any other [[object]]

# Как
Синтаксис
```python
for item in iterable:
```
- `item`: This is a variable that takes on the value of each item in the iterable, one at a time.
- `iterable`: This can be a list, tuple, string, set, dictionary, or any other object that can be iterated over.
# Особенности/атрибуты
- Если переменная(item) не используется в теле цикла, то нужно использовать вместо неё _


# Примеры
```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
    print(fruit)
```
# Частые ошибки
