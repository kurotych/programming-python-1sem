# 24. (П11) Обробка винятків: програми, які не падають на поганих файлах

## Мета роботи

Навчитися перехоплювати винятки, що виникають під час роботи з файлами, обирати правильний тип і порядок гілок `except`, застосовувати `else` і `finally`, обробляти виняток на тому рівні програми, де є достатньо контексту, та завершувати програму зрозумілим повідомленням і кодом завершення замість traceback.

!!! danger "Персоналізація обовʼязкова"
    Кожне завдання має працювати з **вашими справжніми даними**: іменем, групою, днем народження, вашим файлом витрат із попередньої роботи, вашим зображенням. Роботи з вигаданими `test.txt`, `aaa;bbb;111` або з даними іншого студента не приймаються.

## Підготовка

Створіть каталог `lab24`. Кожне завдання виконуйте в окремому файлі: `task1.py` … `task3.py`. Теоретичний матеріал — [лекція 23](23-exceptions-lecture.md) та [лекція 21](21-files-lecture.md).

### Ваші персональні дані

| Позначення | Звідки береться | Приклад | Завдання |
|---|---|---|---|
| `name`, `surname` | ваше імʼя та прізвище латиницею | `Ivan Petrenko` | усі |
| `group` | ваша група | `PZ-11` | усі |
| `day` | день вашого народження | `14` | 3 |
| профіль | файл `data/profile.txt` з роботи 22 | `name: Ivan` | 1, 2 |
| зображення | будь-яке **ваше** фото або малюнок (`.jpg` чи `.png`) | `photo.jpg` | 1, 3 |
| витрати | ваш файл `data/expenses.csv` з роботи 22 | `2026-09-21;food;85.50;lunch` | 3 |

### Вимоги до всіх завдань

- Першим рядком програма виводить ваше імʼя, прізвище та групу.
- Лише вбудовані винятки; голий `except:` та `except Exception: pass` заборонені.
- Повідомлення про помилки — у `sys.stderr`, результат — у звичайний вивід.
- Очікувана помилка (немає файлу, зіпсований рядок, не той аргумент) не завершується traceback.
- Після кожного запуску показуйте код завершення (`echo $?`).

## Хід роботи

### Завдання 1. Чому файл не читається

Напишіть утиліту, яка отримує з командного рядка один або кілька шляхів і для кожного або показує кількість рядків і перший рядок файлу, або пояснює, чому прочитати його не вдалося (разом з назвою типу винятку). Порожній файл теж вважається помилкою. Наприкінці — підсумок: скільки перевірено і скільки не вдалося.

Коди завершення: `0` — усі файли прочитано, `1` — хоча б один не прочитано, `2` — шляхів не передано.

Перевірте програму на: вашому профілі, неіснуючому файлі з вашим прізвищем у назві, каталозі, вашому зображенні, порожньому файлі.

Приклад:  
```text
$ python3 task1.py data/profile.txt data/petrenko.txt data data/photo.jpg data/empty.txt
Ivan Petrenko, PZ-11
ok:    data/profile.txt: 5 lines, first: name: Ivan
error: data/petrenko.txt: file not found (FileNotFoundError)
error: data: is a directory (IsADirectoryError)
error: data/photo.jpg: not a UTF-8 text file (UnicodeDecodeError)
error: data/empty.txt: file is empty (ValueError)
checked 5, failed 4
$ echo $?
1

$ python3 task1.py
usage: python3 task1.py <path> [<path> ...]
$ echo $?
2
```

### Завдання 2. Копіювання файлів

Напишіть утиліту копіювання файлу — спрощений аналог команди `cp`:

```text
python3 task2.py [--force] <source> <destination>
```

Програма копіює текстові файли і після успішного копіювання повідомляє про це.

Програма має зрозуміло повідомляти про кожну з цих ситуацій:

| Ситуація | Що відбувається |
|---|---|
| джерела не існує | помилка |
| джерело — каталог | помилка |
| каталогу призначення не існує | помилка |
| файл призначення вже існує | помилка; з `--force` файл перезаписується |
| призначення — каталог | помилка |
| джерело і призначення — той самий файл | помилка, файл лишається неушкодженим |

Коди завершення: `0` — файл скопійовано, `1` — помилка копіювання, `2` — неправильний виклик.

Приклад:
```text
$ python3 task2.py data/profile.txt backup_petrenko/profile.txt
Ivan Petrenko, PZ-11
copied: data/profile.txt -> backup_petrenko/profile.txt
$ echo $?
0

$ python3 task2.py data/profile.txt backup_petrenko/profile.txt
Ivan Petrenko, PZ-11
error: backup_petrenko/profile.txt already exists, use --force to overwrite
$ echo $?
1

$ python3 task2.py --force data/profile.txt backup_petrenko/profile.txt
Ivan Petrenko, PZ-11
copied: data/profile.txt -> backup_petrenko/profile.txt

$ python3 task2.py data/petrenko.txt backup_petrenko/profile.txt
Ivan Petrenko, PZ-11
error: source data/petrenko.txt not found

$ python3 task2.py data backup_petrenko/data
Ivan Petrenko, PZ-11
error: source data is a directory, not a file

$ python3 task2.py data/profile.txt nowhere/profile.txt
Ivan Petrenko, PZ-11
error: destination folder nowhere not found

$ python3 task2.py --force data/profile.txt backup_petrenko
Ivan Petrenko, PZ-11
error: destination backup_petrenko is a directory, not a file

$ python3 task2.py --force data/profile.txt data/profile.txt
Ivan Petrenko, PZ-11
error: source and destination are the same file
$ echo $?
1

$ python3 task2.py data/profile.txt
Ivan Petrenko, PZ-11
usage: python3 task2.py [--force] <source> <destination>
$ echo $?
2
```

### Завдання 3. Трекер витрат, який не боїться поганих даних

Головне завдання роботи. Візьміть свій трекер витрат з роботи 22 (команди `add`, `list`, `report`) і зробіть його стійким до поганих даних.

Зіпсуйте копію свого `expenses.csv`: у чотирьох різних рядках зробіть суму текстом, приберіть одне поле, поставте неіснуючу дату (місяць `13`, день — ваш `day`) і відʼємну суму.

Що має вміти програма:

- **`list` і `report`** пропускають погані рядки, працюють з рештою і наприкінці виводять у `sys.stderr` перелік пропущених рядків з номером і причиною. Код завершення — `0`.
- **`add`** не записує некоректні дані: повідомлення про помилку, код `1`, файл без змін. Правила перевірки — ті самі, що й під час читання файлу (без дублювання коду).
- **Проблеми самого файлу** (його немає, це каталог, він не в UTF-8) обробляються в одному місці програми, кожна зі своїм повідомленням, код `1`.
- **Порожній файл** (лише заголовок): `report` повідомляє, що звітувати нічого, і повертає `1`.
- **Неправильний виклик** (немає команди, невідома команда, замало аргументів) — підказка і код `2`.

Перевірте програму на: зіпсованому файлі, невдалих і вдалому виклику `add`, відсутньому файлі, каталозі замість файлу, вашому зображенні під іменем `expenses.csv`, файлі лише із заголовком.

**Питання до звіту:** чому після `python3 task3.py list > list.txt` перелік пропущених рядків зʼявляється на екрані, а не у файлі?

```text
$ python3 task3.py list
date        category       amount  note
2026-09-21  food            85.50  lunch at canteen
2026-09-21  transport       32.00  bus to college
...
9 records, total 1041.00
skipped 4 bad line(s):
  line 5: bad amount 'sixty five'
  line 7: expected 4 fields, got 3
  line 10: bad date '2026-13-14'
  line 12: amount must be positive, got -32.00
$ echo $?
0

$ python3 task3.py add 2026-09-28 food ninety pizza
error: bad amount 'ninety'
nothing was added
$ echo $?
1

$ python3 task3.py list
error: data/expenses.csv is a directory, not a file
$ echo $?
1

$ python3 task3.py report
error: nothing to report, no valid records
$ echo $?
1

$ python3 task3.py summary
error: unknown command summary
usage: python3 task3.py add <date> <category> <amount> <note>
       python3 task3.py list
       python3 task3.py report
$ echo $?
2
```
