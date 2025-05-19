# Лабораторная работа №4

**«Исследование криптографических алгоритмов шифрования информации в компьютерных системах»**

---

### Задание 1

Расшифровать сообщение, зашифрованное  методомпростой замены на ключе К (ключом К является таблица №1 см. в приложении у преподавателя, в соответствии с
номером своего варианта).

> Пример ключа (таблица замены):
> Открытые буквы:    а б в г д е ж з и к л м н о п р с т у ф х ц ч ш щ ъ ы ь э ю я
> Шифрованные:       г л ь п д р а м ц в э ъ х о б н с ж я и ю к щ ф е у ы ч ш т з

---

#### Решение:

> Зашифрованный текст: **"цвэъхо"**

**Решение:**

- ц → к
- в → к
- э → л
- ъ → м
- х → н
- о → о

**Ответ:** `"кклмно"`

---

### Задание 2

Зашифровать с помощью метода гаммирования на ключе К сообщение по модулю 32 (ключ К см. в приложении у преподавателя, в соответствии с номером своего
варианта).

#### Формула:

$$
s_i = (C(t_i) + k_i) \mod 32
$$

---

#### Обозначения:

- `t_i` — i-й символ открытого текста
- `C(t_i)` — порядковый номер символа `t_i` в алфавите (0-31)
- `k_i` — i-й элемент ключа
- `s_i` — результат (номер зашифрованного символа)
- `mod 32` — операция взятия остатка по модулю 32
- `n` — длина сообщения

---

#### Решение:

> Сообщение: **НАСТУПАТЬ**
> Ключ: `05 12 03 29 24 12 30 14 11 31`
> Алфавит: 32 буквы русского алфавита, е и ё считаются одной буквой

Сообщение → Н А С Т У П А Т Ь

Номера : [13, 0, 17, 18, 19, 15, 0, 18, 32]

- 13+5 = 18
- 0+12 = 12
- 17+3 = 20
- 18+29 = 47 → 47 % 32 = 15
- 19+24 = 43 % 32 = 11
- 15+12 = 27
- 0+30 = 30
- 18+14 = 32
- 28+11 = 39 % 32 = 7
- 32+31 = 63 % 32 = 31

**Ответ:** `18 12 20 15 11 27 30 32 07 31`

---

### Задание 3

Расшифровать фразу, зашифрованную при помощи шифра вертикальной перестановки, используя прямоугольник 7x8, на ключе К (ключ К и фразу см. в приложении у преподавателя,
в соответствии с номером своего варианта).

#### Решение:

```
def vertical_encrypt(text: str, rows: int, cols: int, key: list[int]) -> str:
    """
    Шифрование методом вертикальной перестановки.

    :param text: Открытый текст длиной rows * cols
    :param rows: Количество строк в таблице
    :param cols: Количество столбцов в таблице
    :param key: Список длиной cols — ключ перестановки (номера столбцов от 1 до cols)
    :return: Зашифрованная строка
    """
    matrix = [list(text[i * cols:(i + 1) * cols]) for i in range(rows)]
    transposed = list(zip(*matrix))
    reordered = [transposed[k - 1] for k in key]
    return ''.join(''.join(col) for col in reordered)


def vertical_decrypt(text: str, rows: int, cols: int, key: list[int]) -> str:
    """
    Расшифровка текста, зашифрованного методом вертикальной перестановки.

    :param text: Зашифрованная строка длиной rows * cols
    :param rows: Количество строк в таблице
    :param cols: Количество столбцов в таблице
    :param key: Ключ перестановки, применённый при шифровании
    :return: Расшифрованный текст
    """
    if len(text) != rows * cols:
        raise ValueError("Длина шифртекста должна быть равна rows * cols.")

    column_chunks = [list(text[i * rows:(i + 1) * rows]) for i in range(cols)]
    ordered_columns: list[list[str]] = [[""]] * cols
    for i, k in enumerate(key):
        ordered_columns[k - 1] = column_chunks[i]

    return ''.join(ordered_columns[c][r] for r in range(rows) for c in range(cols))


# Демонстрация работы
rows = 7
cols = 8
key = [5, 1, 4, 2, 6, 3, 8, 7]
original_text = "вертикальнаяшифровкаиспользуетключдляперестановоквтексте"

encrypted = vertical_encrypt(original_text, rows, cols, key)
decrypted = vertical_decrypt(encrypted, rows, cols, key)

print(encrypted) -> ишиеянквьолюектяаулаеенвьчсвкистпосракздттлролроеафпкевт
print(decrypted) -> вертикальнаяшифровкаиспользуетключдляперестановоквтексте


```

### Задание 4: Расшифровка методом Плейфера


Дешифрировать перехваченное сообщение, в предположении, что оно зашифровано при помощи метода Плейфера в 30-ти буквенном русском алфавите, ключевым словом является элемент алфавита (биграмму см. в приложении у преподавателя, в соответствии с номером своего варианта).

#### Решение:

```
russian_alphabet = "абвгдежзийклмнопрстуфхцчшщыюя"

def generate_playfair_table(keyword: str) -> list[list[str]]:
    """
    Генерирует таблицу Плейфера 5x6 по заданному ключевому слову.

    :param keyword: Ключевое слово
    :return: Таблица 5x6 в виде списка списков
    """
    filtered_key = ''
    seen = set()
    for c in keyword:
        if c in russian_alphabet and c not in seen:
            seen.add(c)
            filtered_key += c
    remaining = ''.join(c for c in russian_alphabet if c not in filtered_key)
    sequence = filtered_key + remaining
    return [list(sequence[i * 6:(i + 1) * 6]) for i in range(5)]

def find_position(table: list[list[str]], char: str) -> tuple[int, int]:
    """
    Находит позицию символа в таблице.

    :param table: Таблица Плейфера
    :param char: Символ для поиска
    :return: Кортеж (строка, столбец)
    """
    for i, row in enumerate(table):
        if char in row:
            return i, row.index(char)
    raise ValueError(f"Символ {char} не найден в таблице.")

def playfair_encrypt(text: str, table: list[list[str]]) -> str:
    """
    Шифрует текст методом Плейфера.

    :param text: Открытый текст
    :param table: Таблица Плейфера
    :return: Зашифрованный текст
    """
    bigrams = []
    i = 0
    while i < len(text):
        a = text[i]
        b = text[i + 1] if i + 1 < len(text) else 'ф'
        if a == b:
            b = 'ф'
            i += 1
        else:
            i += 2
        bigrams.append((a, b))

    result = ""
    for a, b in bigrams:
        ra, ca = find_position(table, a)
        rb, cb = find_position(table, b)

        if ra == rb:
            result += table[ra][(ca + 1) % 6]
            result += table[rb][(cb + 1) % 6]
        elif ca == cb:
            result += table[(ra + 1) % 5][ca]
            result += table[(rb + 1) % 5][cb]
        else:
            result += table[ra][cb]
            result += table[rb][ca]

    return result

def playfair_decrypt(cipher: str, table: list[list[str]]) -> str:
    """
    Расшифровывает текст, зашифрованный методом Плейфера.

    :param cipher: Зашифрованный текст
    :param table: Таблица Плейфера
    :return: Расшифрованный текст
    """
    bigrams = [(cipher[i], cipher[i + 1]) for i in range(0, len(cipher), 2)]

    result = ""
    for a, b in bigrams:
        ra, ca = find_position(table, a)
        rb, cb = find_position(table, b)

        if ra == rb:
            result += table[ra][(ca - 1) % 6]
            result += table[rb][(cb - 1) % 6]
        elif ca == cb:
            result += table[(ra - 1) % 5][ca]
            result += table[(rb - 1) % 5][cb]
        else:
            result += table[ra][cb]
            result += table[rb][ca]

    return result

# Демонстрация работы
keyword = "ключ"
text = "текст для теста"
text = ''.join(c for c in text if c in russian_alphabet)

table = generate_playfair_table(keyword)
encrypted = playfair_encrypt(text, table)
decrypted = playfair_decrypt(encrypted, table)

print("Таблица Плейфера:")
for row in table:
    print(" ".join(row))

print("\nЗашифрованный текст:", encrypted) # -> удлрщмашудтужя
print("Расшифрованный текст:", decrypted) # -> текстдлятестаф

```
