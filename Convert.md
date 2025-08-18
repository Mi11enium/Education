Шпаргалка по самым часто используемым методам `Convert` в C#

|Метод|Преобразует|Пример|Результат|
|---|---|---|---|
|`Convert.ToInt32()`|строка / число → **int**|`Convert.ToInt32("123")`|`123`|
|`Convert.ToDouble()`|строка / число → **double**|`Convert.ToDouble("3.14")`|`3.14`|
|`Convert.ToDecimal()`|строка / число → **decimal**|`Convert.ToDecimal("19.99")`|`19.99M`|
|`Convert.ToBoolean()`|строка / число → **bool**|`Convert.ToBoolean("True")`|`true`|
||0 → false, ≠0 → true|`Convert.ToBoolean(0)`|`false`|
|`Convert.ToString()`|любой тип → **string**|`Convert.ToString(42)`|`"42"`|
## Мини-заметки

- `null` → обычно в ноль или `false`, а не ошибка:
    ```csharp
Convert.ToInt32(null); // 0
    ```
- Пустая строка `""` или неверный формат → **ошибка**:
    ```csharp
Convert.ToInt32(""); // Ошибка
```