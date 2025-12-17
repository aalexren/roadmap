# Tinkoff (Т-Банк) Middle Data Engineer 2025

## Algorithms part

### String compression
Given an array of characters chars, compress it using the following algorithm:

Begin with an empty string s. For each group of consecutive repeating characters in chars:

- If the group's length is 1, append the character to s.
- Otherwise, append the character followed by the group's length.

The compressed string s should not be returned separately, but instead, be stored in the input character array chars. Note that group lengths that are 10 or longer will be split into multiple characters in chars. After you are done modifying the input array, return the new length of the array. You must write an algorithm that uses only constant extra space.

- [HakerRank](https://www.hackerrank.com/challenges/string-compression/problem)
- [LeetCode (array version)](https://leetcode.com/problems/string-compression/description/)

### Sort out the athletes (SQL)
Дана таблица sportsmen со столбцами id: int (primary key) и score (int). Необходимо каждому спортсмену в таблице присвоить место по результатам score. Реализовать без использования оконных функций или 'order by'.

```
Origin:
id / score
1 - 35
2 - 95
3 - 55
4 - 17
5 - 63

Result:
id / place
1 - 4
2 - 1
3 - 3
4 - 5
5 - 2
```

*Solution: идея в том, чтобы считать не количество спортсменов, которые набрали меньше, а именно количество уникальных баллов, которые оказались меньше, чем у спортсмена. Фактически, количество мест это количество уникальных баллов, т.к. если спортсмены набрали одинаковое количество баллов у них будет одно и то же место.*
```sql
SELECT t1.id,
       t1.score,
       1 + COUNT(DISTINCT t2.score) AS rank
  FROM sportsmen t1
  LEFT JOIN sportsmen t2 
         ON t2.score > t1.score
 GROUP BY t1.id, t1.score;
```

# Magnit Tech (Магнит) Middle Data Engineer 2025

## Algorithms part

### Longest substring with K uniques
You are given a string `s` consisting only lowercase alphabets and an integer `k`. Your task is to find the length of the longest substring that contains exactly `k` distinct characters.

- [geekforgeeks](https://www.geeksforgeeks.org/problems/longest-k-unique-characters-substring0853/1)
- [LeetCode (related problem)](https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/description/)


# Yandex (Яндекс) Middle Software Engineer 2025

## Practical part

*Remark: в этой секции общение как раз важно, умение пользоваться поисковым движком, находить релевантную информацию и т.д. Не переусложняйте код, пишите сначала "baseline", затем улучшайте. Заранее всё настройте, разберитесь с импортированием в python в созданной среде. Тем не менее, в конечно итоге от вас минимум требуется — хотя бы одно рабочее решение и полностью рабочие тесты к нему (пускай и простые), иначе "финиш".*

У вас есть некая система, которая позволяет сохранить бэкап и после скачать его в виде потока байтов.
Перед тем, как положить бекап на хранение, его надо сжать и зашифровать. Есть ресурсоемкий алгоритм
сжатия/шифрования, позволяющий (относительно медленно) сжимать/шифровать поток данных. В итоге данные
сжимаются, примерно, в 1.5-3 раза. Сжатый и шифрованный бэкап может занимать до 1 TiB.

Для хранения сжатого и зашифрованного бэкапа используется холодное сетевое хранилище (например, S3),
которое позволяет сохранять наш бекап, но у хранилища есть ограничения на размер файла – размер одного
файла в хранилище не может превышать 100 MiB. Для каждого нового бекапа создается фолдер в хранилище.
Фолдер может содержать только 1 бэкап.

```
  +---+-----+-----+-----+-----+-----+-----+      +-----+  +-----+
  | 7 |  6  |  5  |  4  |  3  |  2  |  1  |----->|  1  |  |  5  |
  +---+-----+-----+-----+-----+-----+-----+      +-----+  +-----+
                                 |               +-----+  +-----+
                                 +-------------->|  2  |  |  6  |
                                                 +-----+  +-----+
                                                 +-----+  +---+
                                                 |  3  |  | 7 |
                                                 +-----+  +---+
                                                 +-----+
                                                 |  4  |
                                                 +-----+
```
Необходимо реализовать функции для сохранения бэкапа в сетевое хранилище и восстановления обратно.

```python
import asyncio
import time
from abc import ABC, abstractmethod
from typing import BinaryIO


class Processor(ABC):
    @abstractmethod
    def compress_and_encrypt(self, data: bytes) -> bytes: ...

    @abstractmethod
    def decrypt_and_uncompress(self, data: bytes) -> bytes: ...


class Folder(ABC):
    MAX_FILE_SIZE = 100 * 1024 * 1024

    @abstractmethod
    async def write_file(self, name: str, data: bytes): ...

    @abstractmethod
    async def read_file(self, name: str) -> bytes: ...

    @abstractmethod
    async def list_files(self) -> list[str]: ...


class BackupManager:
    def __init__(self):
        pass

    async def backup(self, in_stream: BinaryIO) -> None:
        pass
        
    async def restore(self, out_stream: BinaryIO) -> None:
        pass
```

#### Тесты

```python
class StubFolder(Folder):
    MAX_FILE_SIZE = 1024

    def __init__(self):
        self.files: dict[str, bytes] = {}
        self.delay = 0.02

    async def write_file(self, name: str, data: bytes) -> None:
        if len(data) > self.MAX_FILE_SIZE:
            raise ValueError(
                f"File {name} is too large: {len(data)} bytes > {self.MAX_FILE_SIZE} bytes"
            )
        await asyncio.sleep(self.delay)
        self.files[name] = data

    async def read_file(self, name: str) -> bytes:
        if name not in self.files:
            raise FileNotFoundError(f"File {name} not found")
        await asyncio.sleep(self.delay)
        return self.files[name]

async def list_files(self) -> list[str]:
        await asyncio.sleep(self.delay)
        return list(self.files.keys())


class StubProcessor(Processor):
    def __init__(self, encryption_key: int = 0x55):
        self.encryption_key = encryption_key
        self.delay = 0.02

    def compress_and_encrypt(self, data: bytes) -> bytes:
        if not data:
            return b""

        time.sleep(self.delay)
        return bytes(b ^ self.encryption_key for b in data)

    def decrypt_and_uncompress(self, data: bytes) -> bytes:
        if not data:
            return b""

        time.sleep(self.delay)
        return bytes(b ^ self.encryption_key for b in data)
```

## Algorithms part

Учтите, что несмотря на всё, что пишет Яндекс ваша цель одна — решить задачу как можно быстрее **без** ошибок. Все разговоры про общение на интервью, рассуждение вслух и т.д. — сплошная формальность, единственное зачем это может быть нужно — вам так проще рассуждать. Поэтому сосредоточтесь полностью на решении. Важно! Иногда вас будут просить дать теоретическое решение перед написанием кода, в таком случае, если вы его знаете, то проблем нет, но если возник ступор, обязательно имейте в виду, что *подсказка* в решении будет практически полностью гарантировать **провал** секции! Даже если вы старались, выдали много разных версий, всё на что смотрят — решение задачи без ошибок. **Любая** неточность или ошибка, которые обнаружит собеседующий после ваших слов "готово" кратно сокращает шансы на прохождение секции.

### Longest subarray of 1's after deleting one element
Given a binary array `nums`, you should delete one element from it.

Return the size of the longest non-empty subarray containing only 1's in the resulting array. Return 0 if there is no such subarray.

- [LeetCode](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/description/?envType=problem-list-v2&envId=v3ga2xk6)

*Remark: отдельно стоит рассмотреть два случая: когда все элементы 0 и когда все элементы 1 (удалять можно не только нули).*

### Maximum average subarray II
You are given an integer array `nums` with `n` elements and an integer `k`.

Your task is to find a contiguous subarray whose length is at least `k` that has the maximum possible average value. Return this maximum average value.

- [LeetCode](https://leetcode.com/problems/maximum-average-subarray-ii/description/?envType=problem-list-v2&envId=v3ga2xk6)
- [algo.monster](https://algo.monster/liteproblems/644)
- [algomap.io](https://algomap.io/question-bank/maximum-average-subarray-ii)

*Remark: "жадное" решение тут не работает, попытайтесь свести задачу к "Two Sum".*
