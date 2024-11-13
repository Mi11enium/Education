# Description

# Action
used to control the flow of a program based on certain conditions
# Object
the code block

# How does it work (Logic)

# Features/Attribute

## Else
- Python допускает необязательный блок `else` в конце циклов `while` и `for`.
```python
while условие:
    блок кода1
else:
    блок кода2

# или

for i in range(10):
    блок кода1
else:
    блок кода2
```

`Блок кода2`, указанный в `else`, будет выполнен, когда **штатным образом** завершается цикл `while` или `for`.
! Если слово `else` отсутствует в описании цикла, то `блок кода2` будет выполняться после завершения цикла, несмотря ни на что. Если же слово `else` присутствует, то `блок кода2` будет выполняться только в том случае, если цикл завершается **штатным образом**. Под **штатным завершением** цикла подразумевается его завершение **без** использования оператора прерывания `break`.
# Syntax
```python
if condition1:
    # Code for condition1
elif condition2:
    # Code for condition2
else:
    # Code to be executed if all conditions are false
```
# Examples

# Common mistakes
