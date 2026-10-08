# 27. (Л) Модулі та пакети у Python. Імпорт, структура проєкту

## Зміст лекції

1. Навіщо ділити програму на файли
2. Модуль — це файл `.py`
3. Форми інструкції `import`
4. Модуль — це обʼєкт
5. Що відбувається під час імпорту
6. `__name__` і `if __name__ == "__main__":`
7. `from ... import` створює нове імʼя, а не посилання на змінну модуля
8. Де Python шукає модулі: `sys.path`
9. Затінення стандартних модулів
10. Стандартна бібліотека
11. Пакети: каталог з `__init__.py`
12. Абсолютний і відносний імпорт
13. Запуск модуля з пакета: `python3 -m`
14. Циклічні імпорти
15. Порядок і стиль імпортів
16. Структура проєкту
17. Приклад: проєкт «журнал оцінок»
18. Типові помилки

## Навіщо ділити програму на файли

Досі кожна наша програма жила в одному файлі. Поки програма займає 50–100 рядків, так і має бути. Але програми ростуть: у домашніх завданнях лекцій 21–26 вже були функції для читання файлу, для обчислень, для виводу звіту, для налаштування логування. Коли все це лежить в одному файлі на 500 рядків, зʼявляються проблеми:

- **Важко знайти потрібне.** Щоб виправити формат звіту, доводиться гортати повз читання файлів і обчислення.
- **Неможливо використати повторно.** Функцію `average()` з одного завдання хочеться взяти в інше. Копіювання означає, що помилку доведеться виправляти в кількох місцях.
- **Важко працювати вдвох.** Двоє людей, які одночасно редагують один великий файл, постійно отримують конфлікти в git (лекція 12).
- **Усе бачить усе.** Будь-яка функція може випадково використати чи змінити будь-яку глобальну змінну.

Розвʼязок — розбити програму на **модулі**: окремі файли, кожен з яких відповідає за одну частину роботи. Модулі, повʼязані за змістом, обʼєднують у **пакети**.

Ви вже користувалися чужими модулями: `import sys`, `import logging`, `from pathlib import Path`. Сьогодні розберемо, як це працює, і навчимося писати власні.

## Модуль — це файл `.py`

**Модуль** (module) — це будь-який файл з кодом на Python. Імʼя модуля — це імʼя файлу без `.py`: файл `grades.py` — це модуль `grades`.

Створіть у порожньому каталозі два файли.

```python
# File: grades.py
PASS_SCORE = 60


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def is_passed(score: float) -> bool:
    return score >= PASS_SCORE
```

```python
# File: main.py
import grades

scores = [78, 92, 55]
avg = grades.average(scores)

print(f"average: {avg:.1f}")
print("passed:", grades.is_passed(avg))
print("pass score:", grades.PASS_SCORE)
```

Запустіть `main.py`:

```bash
python3 main.py
```

```text
average: 75.0
passed: True
pass score: 60
```

Інструкція `import grades` знаходить файл `grades.py` **у тому ж каталозі**, де лежить `main.py`, виконує його і дає доступ до всього, що в ньому визначено, через крапку: `grades.average`, `grades.PASS_SCORE`.

Тепер `grades.py` можна підключити до будь-якої іншої програми — функції написано один раз.

!!! warning "Імʼя файлу має бути правильним ідентифікатором"
    Імʼя модуля використовується в коді, тому воно підпорядковується тим самим правилам, що й імена змінних (лекція 04): латинські літери, цифри, `_`, не починається з цифри.

    | Файл | Чи можна імпортувати |
    |---|---|
    | `grades.py`, `student_report.py` | так |
    | `student-report.py` | ні — `import student-report` означає «`student` мінус `report`» |
    | `01_grades.py` | ні — імʼя починається з цифри |
    | `Grades.py` | так, але PEP 8 радить імена модулів **малими літерами** |

    Скрипти, які лише запускають, можна називати як завгодно. Але файли, які імпортуватимуть, — тільки `lowercase_with_underscores`.

## Форми інструкції `import`

### `import module`

```python
import grades

print(grades.average([78, 92, 55]))
```

Імпортується модуль цілком, до вмісту звертаються через `grades.`. Це **рекомендована форма за замовчуванням**: з коду одразу видно, звідки взялася функція. `grades.average(...)` читається однозначно, а просто `average(...)` змушує шукати, де її визначено.

### `import module as alias`

```python
import grades as gr

print(gr.average([78, 92, 55]))
```

Модулю дається коротше імʼя (псевдонім). Використовують для довгих назв або коли є загальноприйнята домовленість, наприклад `import numpy as np`. Не варто вигадувати власні загадкові скорочення.

### `from module import name`

```python
from grades import average, is_passed

avg = average([78, 92, 55])
print(avg, is_passed(avg))
```

Імпортуються конкретні імена — їх можна використовувати без префікса. Зручно, коли імʼя часто використовується і його походження очевидне: `from pathlib import Path`, `from datetime import datetime`.

### `from module import name as alias`

```python
from grades import average as grades_average

print(grades_average([78, 92, 55]))
```

Допомагає, коли імена з різних модулів збігаються.

### `from module import *`

```python
# File: shapes.py
__all__ = ["circle_area"]

PI = 3.14159


def circle_area(radius: float) -> float:
    return PI * radius**2


def _check(radius: float) -> None:
    if radius < 0:
        raise ValueError("radius must be non-negative")
```

```python
# File: main.py
from shapes import *

print(circle_area(2))
print(PI)
```

```text
12.56636
Traceback (most recent call last):
  File "/home/student/project/main.py", line 5, in <module>
    print(PI)
          ^^
NameError: name 'PI' is not defined
```

`*` імпортує всі «публічні» імена модуля. Що саме вважається публічним:

- якщо в модулі є список `__all__` — тільки імена з нього (тому `PI` не імпортувався);
- якщо `__all__` немає — усі імена, що **не починаються з `_`**.

!!! danger "Не використовуйте `import *` у своєму коді"
    Після `from shapes import *` неможливо зрозуміти, звідки взялося імʼя, редактор і mypy гірше підказують, а нове імʼя в модулі може непомітно **перезаписати** вашу змінну з такою самою назвою. `import *` іноді доречний в інтерактивному сеансі `python3`, але не в програмах.

!!! tip "Підкреслення на початку імені"
    Імʼя `_check` означає «внутрішня деталь модуля, ззовні не використовуйте». Python не забороняє `shapes._check(...)`, це домовленість між програмістами. Але `import *` такі імена пропускає, а редактор зазвичай не пропонує їх у підказках.

### Порівняння

| Форма | Як викликати | Коли використовувати |
|---|---|---|
| `import grades` | `grades.average(...)` | за замовчуванням |
| `import grades as gr` | `gr.average(...)` | довга назва або загальноприйнятий псевдонім |
| `from grades import average` | `average(...)` | імʼя часто використовується і його походження очевидне |
| `from grades import average as avg_of` | `avg_of(...)` | конфлікт імен |
| `from grades import *` | `average(...)` | не використовувати |

## Модуль — це обʼєкт

Після `import grades` змінна `grades` посилається на **обʼєкт модуля** — такий самий обʼєкт, як список чи функція. У нього є атрибути: усе, що визначено у файлі, плюс кілька службових.

```python
# File: main.py  (grades.py from the previous section is next to it)
import grades

print(type(grades))
print(grades.__name__)
print(grades.__file__)
print([name for name in dir(grades) if not name.startswith("__")])
```

```text
<class 'module'>
grades
/home/student/project/grades.py
['PASS_SCORE', 'average', 'is_passed']
```

| Атрибут | Що містить |
|---|---|
| `__name__` | імʼя модуля |
| `__file__` | повний шлях до файлу модуля |
| `__doc__` | рядок документації модуля (перший рядок-літерал у файлі) |
| `dir(module)` | список усіх імен у модулі |

**Кожен модуль має власний простір імен** (namespace). Змінна `PASS_SCORE` у `grades.py` і змінна `PASS_SCORE` у `main.py` — це дві різні змінні, які не заважають одна одній. Саме тому модулі розвʼязують проблему «усе бачить усе»: те, що визначено в модулі, видно ззовні лише через `grades.`.

!!! tip "Документуйте модуль"
    Першим рядком файлу пишіть рядок документації — він потрапить у `__doc__` і в `help(grades)`:
    ```python
    """Calculations with student scores. No input/output here."""
    ```

## Що відбувається під час імпорту

```python
# File: config.py
print("config: loading")

APP_NAME = "gradebook"
```

```python
# File: main.py
import sys

print("main: start")
import config
import config
from config import APP_NAME

print("main:", APP_NAME)
print("config" in sys.modules)
```

```text
main: start
config: loading
main: gradebook
True
```

Три імпорти — одне повідомлення `config: loading`. Тому модуль може безпечно імпортуватися з десяти різних файлів: усі вони отримають **той самий обʼєкт**.

!!! note "Байткод у `__pycache__`"
    Після першого імпорту поруч із вашими модулями зʼявиться каталог `__pycache__` з файлами `grades.cpython-313.pyc` — це скомпільований байткод (лекція 01). Він пришвидшує наступні запуски і створюється тільки для **імпортованих** модулів, а не для файлу, який ви запускаєте. Каталог має бути в `.gitignore` (лекція 12).

!!! warning "Зміни в модулі не підхоплюються «на льоту»"
    Якщо ви в інтерактивному `python3` зробили `import grades`, а потім змінили `grades.py`, повторний `import grades` нічого не змінить — модуль уже в `sys.modules`. Перезапустіть інтерпретатор.

## `__name__` і `if __name__ == "__main__":`

Кожен модуль має змінну `__name__`. Її значення залежить від того, **як** використовується файл:

- файл **запустили** командою `python3 grades.py` — `__name__ == "__main__"`;
- файл **імпортували** (`import grades`) — `__name__ == "grades"`.

```python
# File: grades.py
print("grades: __name__ =", __name__)
```

```python
# File: main.py
import grades

print("main: __name__ =", __name__)
```

```bash
python3 grades.py
```

```text
grades: __name__ = __main__
```

```bash
python3 main.py
```

```text
grades: __name__ = grades
main: __name__ = __main__
```

### Навіщо це потрібно

Типова ситуація: ви написали `grades.py` і внизу додали кілька рядків, щоб перевірити функції.

```python
# File: grades.py
PASS_SCORE = 60


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


print("self-check:", average([78, 92, 55]))
```

Тепер **кожна** програма, яка робить `import grades`, першим рядком виведе `self-check: 75.0`. Перевірка, яка мала виконуватися лише під час розробки, просочилася в чужий вивід. Ще гірше, якщо там був `input()` — імпорт «зависне» в очікуванні введення.

Розвʼязок — виконувати такий код лише тоді, коли файл запущено напряму:

```python
# File: grades.py
PASS_SCORE = 60


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def main() -> None:
    print("self-check:", average([78, 92, 55]))


if __name__ == "__main__":
    main()
```

- `python3 grades.py` — виводить `self-check: 75.0`;
- `import grades` — нічого не виводить, лише визначає `PASS_SCORE`, `average` і `main`.

!!! tip "Шаблон для кожної програми"
    Відтепер оформлюйте **кожну** програму, яку запускають, так:
    ```python
    # Program: template of a runnable module
    def main() -> None:
        print("hello")


    if __name__ == "__main__":
        main()
    ```
    Уся логіка — у функціях, на верхньому рівні — лише імпорти, константи, визначення і цей блок. Такий файл можна і запустити, і безпечно імпортувати (наприклад, щоб протестувати його функції).

## `from ... import` створює нове імʼя, а не посилання на змінну модуля

Згадаймо лекцію 20: змінна — це імʼя, привʼязане до обʼєкта. `from grades import PASS_SCORE` створює в **поточному** модулі нове імʼя `PASS_SCORE`, яке посилається на той самий обʼєкт `60`. Але це два різні імені в різних просторах імен.

```python
# File: grades.py
PASS_SCORE = 60


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def is_passed(score: float) -> bool:
    return score >= PASS_SCORE
```

```python
# File: main.py
import grades
from grades import PASS_SCORE, is_passed

PASS_SCORE = 50
print("local PASS_SCORE:", PASS_SCORE)
print("grades.PASS_SCORE:", grades.PASS_SCORE)
print("is_passed(55):", is_passed(55))

grades.PASS_SCORE = 50
print("is_passed(55):", is_passed(55))
```

```text
local PASS_SCORE: 50
grades.PASS_SCORE: 60
is_passed(55): False
is_passed(55): True
```

- `PASS_SCORE = 50` у `main.py` змінило лише **локальне** імʼя. Функція `is_passed` бере `PASS_SCORE` зі **свого** модуля, де досі `60`.
- `grades.PASS_SCORE = 50` змінило змінну в самому модулі, і `is_passed` це побачила.

!!! warning "Не змінюйте змінні чужих модулів"
    Те, що `grades.PASS_SCORE = 50` працює, не означає, що так варто робити: будь-який модуль може непомітно змінити поведінку іншого, і помилку буде дуже важко знайти. Якщо значення має налаштовуватися — передайте його **параметром** функції: `is_passed(score, pass_score=50)`. Константи модуля (імена ВЕЛИКИМИ літерами) — тільки для читання.

## Де Python шукає модулі: `sys.path`

Коли модуля ще немає в `sys.modules`, Python шукає його:

1. серед **вбудованих** модулів, скомпільованих усередину інтерпретатора (`sys`, `builtins` та деякі інші);
2. у каталогах зі списку **`sys.path`** — по черзі, до першого збігу.

```python
# Program: print module search path
import sys

for path in sys.path:
    print(repr(path))
```

```text
'/home/student/project'
'/usr/lib/python313.zip'
'/usr/lib/python3.13'
'/usr/lib/python3.13/lib-dynload'
'/usr/local/lib/python3.13/dist-packages'
'/usr/lib/python3/dist-packages'
```

| Елемент `sys.path` | Що там |
|---|---|
| перший елемент | каталог **запущеного скрипта** (для `python3 -m ...` — поточний каталог) |
| вміст змінної оточення `PYTHONPATH` | додаткові каталоги, якщо змінну задано |
| `/usr/lib/python3.13` | **стандартна бібліотека** |
| `.../site-packages` або `.../dist-packages` | **сторонні пакети**, встановлені через `pip` або `apt` |

Вивід на вашому компʼютері відрізнятиметься.

Звідси важливі наслідки:

- **Модуль знаходиться відносно запущеного скрипта**, а не відносно поточного каталогу терміналу. `python3 project/main.py` знайде `project/grades.py`.
- Якщо модуль не знайдено в жодному каталозі — `ModuleNotFoundError`:

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    import nosuchmodule
ModuleNotFoundError: No module named 'nosuchmodule'
```

- Якщо модуль знайдено, але в ньому немає потрібного імені — `ImportError`:

```text
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    from math import squareroot
ImportError: cannot import name 'squareroot' from 'math' (unknown location)
```

!!! danger "Не змінюйте `sys.path` вручну"
    В інтернеті часто радять `sys.path.append("..")`, щоб «побачити» модуль з іншого каталогу. Це крихкий прийом: він залежить від того, звідки запускають програму, і ламається на чужому компʼютері. Правильний розвʼязок — нормальна структура проєкту і запуск через `python3 -m` (розділи нижче).

## Затінення стандартних модулів

Каталог скрипта стоїть у `sys.path` **першим**, раніше за стандартну бібліотеку. Тож якщо назвати свій файл так само, як стандартний модуль, Python імпортує **ваш** файл.

```python
# File: random.py  (bad name!)
print("my random.py")
```

```python
# File: main.py
import random

print(random.randint(1, 6))
```

```text
my random.py
Traceback (most recent call last):
  File "/home/student/project/main.py", line 3, in <module>
    print(random.randint(1, 6))
          ^^^^^^^^^^^^^^
AttributeError: module 'random' has no attribute 'randint' (consider renaming '/home/student/project/random.py' since it has the same name as the standard library module named 'random' and prevents importing that standard library module)
```

Python 3.13 вже підказує причину, у старіших версіях буде лише `AttributeError: module 'random' has no attribute 'randint'`. Найчастіші «жертви»: `random.py`, `math.py`, `string.py`, `logging.py`, `json.py`, `csv.py`, `test.py`, `statistics.py` — студенти люблять називати файли за темою заняття.

!!! tip "Як виправити"
    1. Перейменуйте свій файл (`random_practice.py`).
    2. Видаліть каталог `__pycache__` поруч із ним, щоб не лишилося старого байткоду.
    3. Перевірити, який файл насправді імпортується, можна так: `print(random.__file__)`.

## Стандартна бібліотека

Разом з Python встановлюються сотні модулів — **стандартна бібліотека**. Перш ніж писати щось своє чи встановлювати сторонній пакет, перевірте, чи немає готового розвʼязку в ній.

| Модуль | Для чого | Приклад |
|---|---|---|
| `math` | математичні функції | `math.sqrt(2)`, `math.pi` |
| `random` | випадкові числа | `random.randint(1, 6)`, `random.choice(names)` |
| `statistics` | середнє, медіана, відхилення | `statistics.median(scores)` |
| `datetime` | дати і час | `datetime.now()`, `date(2026, 10, 6)` |
| `time` | пауза, вимірювання часу | `time.sleep(1)`, `time.perf_counter()` |
| `pathlib` | шляхи до файлів (лекція 21) | `Path("data") / "grades.txt"` |
| `os` | взаємодія з ОС, змінні оточення | `os.environ.get("HOME")` |
| `sys` | аргументи, вихід, інтерпретатор | `sys.argv`, `sys.exit(1)` |
| `json`, `csv` | читання і запис форматів даних | `json.load(file)` |
| `collections` | `Counter`, `defaultdict`, `deque` (лекція 25) | `Counter(words)` |
| `itertools` | комбінації і перестановки, зручні цикли | `itertools.combinations(names, 2)` |
| `re` | регулярні вирази | `re.fullmatch(r"[A-Z]{2}-\d{2}", group)` |
| `argparse` | розбір аргументів командного рядка | `parser.add_argument("--debug")` |
| `logging` | логування (лекція 26) | `logging.getLogger(__name__)` |
| `typing` | типи для анотацій (лекція 20) | `from typing import Final` |

```python
# Program: a few standard library modules together
import random
import statistics
from datetime import date

scores = [random.randint(50, 100) for _ in range(5)]

print("today:", date.today().isoformat())
print("scores:", scores)
print("mean:", statistics.mean(scores))
print("median:", statistics.median(scores))
```

Повний перелік — у [документації стандартної бібліотеки](https://docs.python.org/3/library/index.html).

## Пакети: каталог з `__init__.py`

Коли модулів стає багато, їх групують у **пакети** (packages). **Пакет** — це каталог з модулями та файлом `__init__.py`. Пакети можуть містити інші пакети (**підпакети**).

```text
project/
├── main.py
└── school/
    ├── __init__.py
    ├── grades.py
    └── students.py
```

`school` — це пакет, `school.grades` і `school.students` — модулі в ньому. Імʼя модуля з пакета пишеться через крапку, як шлях: `school.grades` відповідає файлу `school/grades.py`.

Усі форми імпорту працюють і з пакетами:

```python
import school.grades                 # use: school.grades.average(...)
from school import grades            # use: grades.average(...)
from school.grades import average    # use: average(...)
```

Створімо пакет і перевіримо:

```python
# File: school/__init__.py
"""School: utilities for working with student grades."""
```

```python
# File: school/grades.py
def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)
```

```python
# File: main.py
from school import grades

print(grades.average([78, 92, 55]))
```

```bash
python3 main.py
```

```text
75.0
```

### Що таке `__init__.py`

- Його наявність позначає каталог як **звичайний пакет**.
- Він **виконується** під час першого імпорту пакета (або будь-якого модуля з нього) — так само, як звичайний модуль.
- Усе, що в ньому визначено, стає атрибутами пакета: `import school` дає доступ до імен з `school/__init__.py`.

Найчастіше `__init__.py` залишають **порожнім** або з рядком документації. Іноді в ньому «піднімають» найважливіші імена нагору, щоб користувачам пакета не треба було знати, в якому модулі вони лежать:

```python
# File: school/__init__.py
"""School: utilities for working with student grades."""

from .grades import average

__all__ = ["average"]
```

Тепер працює і коротша форма: `from school import average`.

!!! note "А без `__init__.py`?"
    З Python 3.3 каталог без `__init__.py` теж можна імпортувати — це так званий **namespace package**, механізм для складних випадків, коли один пакет розкидано по кількох каталогах. Для власних проєктів **завжди створюйте `__init__.py`**, навіть порожній: так поведінка передбачувана, і інструменти (mypy, редактор, тести) працюють без сюрпризів.

!!! warning "Не тримайте важкий код у `__init__.py`"
    `__init__.py` виконується при **будь-якому** імпорті з пакета. Читання файлів, мережеві запити чи `print()` у ньому сповільнюють і засмічують кожен імпорт.

## Абсолютний і відносний імпорт

Усередині пакета модулі часто імпортують один одного. Є два способи вказати, що саме імпортувати.

**Абсолютний імпорт** — повний шлях від кореня проєкту:

```python
# File: school/students.py
from school.grades import average
```

**Відносний імпорт** — шлях відносно **поточного пакета**, починається з крапки:

```python
# File: school/students.py
from .grades import average      # module grades from the same package
from . import grades             # the same package itself
```

| Запис | Означає |
|---|---|
| `from .grades import average` | модуль `grades` з **того ж** пакета |
| `from . import grades` | модуль `grades` з того ж пакета (як обʼєкт) |
| `from ..config import APP_NAME` | модуль `config` з **батьківського** пакета |

Обидва способи правильні. PEP 8 рекомендує **абсолютний** як очевидніший, а відносний доречний для імпортів **усередині одного пакета**: якщо пакет перейменують, відносні імпорти не доведеться змінювати.

!!! note "Форма `import .grades` не існує"
    Відносний імпорт можливий тільки у формі `from . import ...` або `from .module import ...`.

### Головна пастка відносних імпортів

Відносний імпорт працює **лише в модулі, який є частиною пакета**. Файл, запущений напряму, пакета «не знає»:

```python
# File: school/students.py
from .grades import average

print(average([78, 92, 55]))
```

```bash
python3 school/students.py
```

```text
Traceback (most recent call last):
  File "/home/student/project/school/students.py", line 1, in <module>
    from .grades import average
ImportError: attempted relative import with no known parent package
```

Під час запуску `python3 school/students.py` Python бачить просто файл, його `__name__` — `"__main__"`, і він не входить у жоден пакет. Тому «крапка» ні на що не вказує. Розвʼязок — наступний розділ.

## Запуск модуля з пакета: `python3 -m`

Ключ `-m` запускає **модуль за його іменем**, а не файл за шляхом:

```bash
cd project
python3 -m school.students
```

```text
75.0
```

Що змінюється порівняно з `python3 school/students.py`:

| | `python3 school/students.py` | `python3 -m school.students` |
|---|---|---|
| Що вказуємо | шлях до файлу | імʼя модуля через крапки |
| `sys.path[0]` | каталог `school/` | **поточний** каталог (`project/`) |
| Модуль знає свій пакет | ні | так |
| Відносні імпорти | `ImportError` | працюють |
| Абсолютні `from school...` | `ModuleNotFoundError` | працюють |

Отже, `python3 -m` треба запускати **з кореневого каталогу проєкту** — того, де лежить каталог пакета.

Ви вже бачили `-m`: `python3 -m venv env`, `python3 -m pip install ...` — це запуск модулів `venv` і `pip` зі стандартної бібліотеки та site-packages.

### `__main__.py` — пакет, який можна запустити

Якщо в пакеті є файл `__main__.py`, пакет можна запустити за іменем:

```text
project/
└── school/
    ├── __init__.py
    ├── __main__.py
    └── grades.py
```

```python
# File: school/__main__.py
from .grades import average


def main() -> None:
    print("average:", average([78, 92, 55]))


if __name__ == "__main__":
    main()
```

```bash
python3 -m school
```

```text
average: 75.0
```

Так зазвичай оформлюють **точку входу** програми, що складається з пакета: `python3 -m gradebook`.

## Циклічні імпорти

**Циклічний імпорт** виникає, коли модуль `A` імпортує модуль `B`, а `B` — модуль `A` (напряму або через ланцюжок).

```python
# File: students.py
from grades import average

STUDENTS = {"Shevchenko": [78, 92, 55]}


def report() -> None:
    for name, scores in STUDENTS.items():
        print(name, average(scores))
```

```python
# File: grades.py
from students import STUDENTS


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def best_student() -> str:
    return max(STUDENTS, key=lambda name: average(STUDENTS[name]))
```

```python
# File: main.py
import students

students.report()
```

```text
Traceback (most recent call last):
  File "/home/student/project/main.py", line 1, in <module>
    import students
  File "/home/student/project/students.py", line 1, in <module>
    from grades import average
  File "/home/student/project/grades.py", line 1, in <module>
    from students import STUDENTS
ImportError: cannot import name 'STUDENTS' from 'students' (consider renaming '/home/student/project/students.py' if it has the same name as a library you intended to import)
```

Що сталося (згадайте схему імпорту):

1. `main.py` імпортує `students`. Python створює **порожній** модуль `students`, кладе його в `sys.modules` і починає виконувати `students.py`.
2. Перший рядок `students.py` імпортує `grades`. Починається виконання `grades.py`.
3. Перший рядок `grades.py` імпортує `STUDENTS` з `students`. Модуль `students` уже є в `sys.modules`, але він **виконаний лише до першого рядка** — `STUDENTS` ще не визначено.

У старіших версіях Python повідомлення прямо називає причину: `cannot import name 'STUDENTS' from partially initialized module 'students' (most likely due to a circular import)`. У Python 3.13 підказка про перейменування тут вводить в оману — дивіться на traceback: ланцюжок `students → grades → students` і є ознакою циклу.

## Порядок і стиль імпортів

PEP 8 і загальна практика встановлюють такі правила:

1. Усі імпорти — **на початку файлу**, після рядка документації модуля.
2. Імпорти групують у три блоки, розділені порожнім рядком:
    1. стандартна бібліотека;
    2. сторонні пакети (встановлені окремо, про них — у наступних лекціях);
    3. власні модулі проєкту.
3. Усередині блоку — за алфавітом.
4. Один модуль — один рядок `import`: `import os` і `import sys` окремо, а не `import os, sys`. Для `from ... import a, b` кілька імен в одному рядку — нормально.
5. Жодного `import *`.

```python
"""Entry point of the gradebook application."""

import logging
import sys
from pathlib import Path

from gradebook.grades import average
from gradebook.storage import load_scores
```

!!! tip "Автоматичне сортування"
    Редактор VS Code з розширенням Python вміє впорядковувати імпорти командою **Organize Imports** (`Shift+Alt+O`). Інструменти `isort` або `ruff` роблять це з командного рядка.

## Структура проєкту

Немає єдиного «правильного» способу організувати проєкт, але є загальноприйняті домовленості. Для невеликого застосунку, який ви пишете в курсі, рекомендуємо таку структуру:

```text
gradebook-project/           <- project root, git repository, run commands from here
├── .gitignore
├── README.md                <- what the program does and how to run it
├── data/                    <- input files
│   └── grades.txt
└── gradebook/               <- the package: all the code
    ├── __init__.py
    ├── __main__.py          <- entry point: python3 -m gradebook
    ├── grades.py            <- calculations
    ├── storage.py           <- reading and writing files
    └── report.py            <- building output
```

Принципи, на яких вона побудована:

- **Один модуль — одна відповідальність.** `grades.py` лише обчислює, `storage.py` лише читає файли, `report.py` лише формує текст. Щоб змінити формат звіту, відкриваємо тільки `report.py`.
- **Обчислення без вводу/виводу.** Функції в `grades.py` отримують дані параметрами і повертають результат — жодних `print()`, `input()`, `open()`. Такі функції легко перевіряти і використовувати повторно.
- **Тонка точка входу.** `__main__.py` лише налаштовує логування, викликає інші модулі і повертає код завершення. Логіки в ньому мінімум.
- **Залежності йдуть в один бік:** `__main__` → `report`, `storage` → `grades`. Нижчі модулі нічого не знають про вищі — тож модулі не імпортують один одного по колу.
- **Код — у пакеті, дані — окремо.** Шлях до даних обчислюється від розташування коду (`Path(__file__)`), а не від поточного каталогу терміналу.
- **Команди запускаються з кореня проєкту:** `python3 -m gradebook`.

`.gitignore` для такого проєкту (див. лекцію 12):

```text
__pycache__/
*.pyc
*.log
.vscode/
```

!!! note "Більші проєкти: `pyproject.toml` і src-layout"
    Бібліотеки, які публікують на PyPI, зазвичай мають файл `pyproject.toml` (назва, версія, залежності, налаштування інструментів) і кладуть пакет у підкаталог `src/` (`src/gradebook/`). Крім того, поруч із пакетом зазвичай є каталог `tests/` з автоматичними тестами. Для навчальних проєктів у цьому курсі достатньо структури вище.

## Приклад: проєкт «журнал оцінок»

Зберемо все разом: прочитаємо оцінки студентів з файлу, обчислимо середні бали і виведемо звіт, з логуванням з лекції 26. Створіть структуру з попереднього розділу.

```text
# File: data/grades.txt
Shevchenko;78,92,55
Kovalenko;95,88,100
Bondarenko;40,52,61
Melnyk;90,abc,70
Tkachenko
```

Останні два рядки зіпсовано навмисно.

```python
# File: gradebook/__init__.py
"""Gradebook: read student scores from a file and print a report."""

from .grades import average, is_passed

__all__ = ["average", "is_passed"]
```

```python
# File: gradebook/grades.py
"""Calculations with scores. No input/output here."""

PASS_SCORE = 60


def average(scores: list[int]) -> float:
    return sum(scores) / len(scores)


def is_passed(score: float) -> bool:
    return score >= PASS_SCORE


def letter(score: float) -> str:
    if score >= 90:
        return "A"
    if score >= 75:
        return "B"
    if score >= PASS_SCORE:
        return "C"
    return "F"
```

```python
# File: gradebook/storage.py
"""Reading scores from a text file."""

import logging
from pathlib import Path

logger = logging.getLogger(__name__)


def load_scores(path: Path) -> dict[str, list[int]]:
    result: dict[str, list[int]] = {}
    with path.open(encoding="utf-8") as file:
        for number, line in enumerate(file, start=1):
            line = line.strip()
            if not line:
                continue
            try:
                name, raw_scores = line.split(";")
                scores = [int(item) for item in raw_scores.split(",")]
            except ValueError:
                logger.warning("line %d skipped: %r", number, line)
                continue
            result[name] = scores
    logger.info("loaded %d students from %s", len(result), path.name)
    return result
```

```python
# File: gradebook/report.py
"""Building a text report."""

from .grades import average, is_passed, letter


def build_report(data: dict[str, list[int]]) -> list[str]:
    lines = []
    for name, scores in sorted(data.items()):
        avg = average(scores)
        status = "passed" if is_passed(avg) else "failed"
        lines.append(f"{name:<12} {avg:6.1f}  {letter(avg)}  {status}")
    return lines
```

```python
# File: gradebook/__main__.py
"""Entry point: python3 -m gradebook"""

import logging
import sys
from pathlib import Path

from .report import build_report
from .storage import load_scores

DATA_FILE = Path(__file__).parent.parent / "data" / "grades.txt"

logger = logging.getLogger(__name__)


def main() -> int:
    logging.basicConfig(
        level=logging.INFO,
        format="%(levelname)-7s %(name)s: %(message)s",
    )
    try:
        data = load_scores(DATA_FILE)
    except FileNotFoundError:
        logger.error("data file not found: %s", DATA_FILE)
        return 1
    for line in build_report(data):
        print(line)
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

Запуск з кореня проєкту:

```bash
cd gradebook-project
python3 -m gradebook
```

```text
WARNING gradebook.storage: line 4 skipped: 'Melnyk;90,abc,70'
WARNING gradebook.storage: line 5 skipped: 'Tkachenko'
INFO    gradebook.storage: loaded 3 students from grades.txt
Bondarenko     51.0  F  failed
Kovalenko      94.3  A  passed
Shevchenko     75.0  B  passed
```

Зверніть увагу:

- **`%(name)s` у лозі — це `__name__` модуля.** Одразу видно, що попередження прийшли з `gradebook.storage`, — саме для цього в лекції 26 ми писали `getLogger(__name__)`.
- **`Path(__file__).parent.parent`** — `__file__` у `__main__.py` дорівнює `.../gradebook-project/gradebook/__main__.py`; перший `.parent` — каталог пакета, другий — корінь проєкту. Програма знайде дані незалежно від того, з якого каталогу запущено термінал, якщо пакет видно в `sys.path`.
- **`sys.exit(main())`** — код завершення `0` або `1` (лекція 23) повертається операційній системі.
- Пакет можна використовувати як бібліотеку: `from gradebook import average` працює завдяки `__init__.py`.

Що буде, якщо запускати неправильно:

| Команда | Результат |
|---|---|
| `python3 -m gradebook` з кореня проєкту | працює |
| `python3 -m gradebook` з іншого каталогу | `No module named gradebook` — пакета немає в `sys.path` |
| `python3 gradebook/__main__.py` | `ImportError: attempted relative import with no known parent package` |

## Типові помилки

| Помилка | Чому погано | Як правильно |
|---|---|---|
| Файл названо `random.py`, `math.py`, `logging.py`… | затінює стандартний модуль: `AttributeError` у неочікуваному місці | унікальні імена; перевірка через `module.__file__` |
| Імʼя модуля з дефісом або з цифри на початку | такий файл неможливо імпортувати | `lowercase_with_underscores` |
| `print()`, `input()` або тестовий код на верхньому рівні модуля | виконується при кожному імпорті | код — у функціях; запуск — під `if __name__ == "__main__":` |
| `from module import *` | незрозуміло, звідки імʼя; тихе перезаписування змінних | `import module` або явний перелік імен |
| Змінено `PASS_SCORE` після `from grades import PASS_SCORE` і очікується зміна в модулі | змінено лише локальне імʼя | передавати значення параметром |
| Змінні чужого модуля змінюються ззовні | неочевидні побічні ефекти | константи — тільки для читання |
| `sys.path.append("..")` | працює лише з певного каталогу | правильна структура + `python3 -m` |
| Відносний імпорт у файлі, який запускають напряму | `ImportError: ... no known parent package` | `python3 -m package.module` з кореня проєкту |
| `python3 -m` з неправильного каталогу | `ModuleNotFoundError` | запускати з каталогу, де лежить пакет |
| Модулі імпортують один одного | `ImportError` через частково ініціалізований модуль | залежності між модулями мають іти в один бік |
| Шлях до даних відносно поточного каталогу терміналу | програма працює лише з одного місця | `Path(__file__).parent / ...` |
| Зміни в модулі не видно в інтерактивному `python3` | модуль уже в `sys.modules` | перезапустити інтерпретатор |
| `__pycache__/` у git | зайві файли, привʼязані до версії Python | `.gitignore` |

## Підсумок

- **Модуль** — файл `.py`; його імʼя — імʼя файлу без розширення. Модуль має власний простір імен.
- Форми імпорту: `import m`, `import m as x`, `from m import name`, `from m import name as x`. За замовчуванням — `import m`; `import *` не використовуємо.
- Під час першого імпорту код модуля **виконується повністю, один раз**; результат зберігається в `sys.modules`.
- `__name__` дорівнює `"__main__"` для запущеного файлу та імені модуля для імпортованого. Код запуску — під `if __name__ == "__main__":`.
- `from m import name` створює нове імʼя в поточному модулі; присвоєння йому не змінює `m.name`.
- Модулі шукаються у `sys.path`: каталог скрипта, стандартна бібліотека, site-packages. Свій файл з іменем стандартного модуля його **затінює**.
- **Пакет** — каталог з `__init__.py`; модулі в ньому — `package.module`. `__init__.py` виконується при імпорті пакета.
- **Абсолютний** імпорт — `from package.module import name`; **відносний** — `from .module import name`, працює лише всередині пакета.
- `python3 -m package.module` запускає модуль як частину пакета; `__main__.py` дозволяє запускати `python3 -m package`.
- **Циклічний імпорт** (модулі імпортують один одного) призводить до `ImportError`: один із модулів виконаний лише частково.
- Імпорти — на початку файлу, групами: стандартна бібліотека, сторонні, власні.
- Структура проєкту: пакет з модулями за відповідальностями, тонка точка входу, дані окремо від коду, запуск з кореня проєкту.

## Корисні посилання

- [Tutorial: Modules — офіційний посібник](https://docs.python.org/3/tutorial/modules.html)
- [The import system — як працює імпорт](https://docs.python.org/3/reference/import.html)
- [`__main__` — модуль верхнього рівня і `__main__.py`](https://docs.python.org/3/library/__main__.html)
- [`sys.path` — шлях пошуку модулів](https://docs.python.org/3/library/sys_path_init.html)
- [The Python Standard Library — перелік модулів](https://docs.python.org/3/library/index.html)
- [PEP 8: Imports — стиль імпортів](https://peps.python.org/pep-0008/#imports)

## Домашнє завдання

Мета — навчитися розбивати програму на модулі й пакети, правильно імпортувати і запускати їх. Усі дані — **ваші власні**, латиницею. Для кожного завдання покажіть дерево файлів (`find . -name "*.py" | sort` або `tree`), вміст файлів і вивід запуску.

1. Створіть модуль `about_me.py` з константами `FIRST_NAME`, `LAST_NAME`, `GROUP`, `BIRTH_YEAR` (ваші дані) і функціями `full_name() -> str` та `age(current_year: int) -> int`. У `main.py` використайте модуль трьома способами: `import about_me`, `import about_me as me`, `from about_me import full_name, age`. Виведіть `dir(about_me)` без службових імен і `about_me.__file__`.

2. Додайте в `about_me.py` рядок `print("about_me loaded")` на верхньому рівні. Імпортуйте модуль у `main.py` тричі різними формами і покажіть, скільки разів зʼявилося повідомлення. Поясніть результат через `sys.modules`. Потім приберіть `print`, оформіть самоперевірку модуля через `main()` і `if __name__ == "__main__":` і покажіть вивід `python3 about_me.py` та `python3 main.py`.

3. Створіть у каталозі файл `random.py` (або `statistics.py`) з однією функцією. Напишіть поруч `main.py`, який використовує відповідний стандартний модуль, наприклад, обирає випадкове число від 1 до кількості літер у вашому прізвищі. Покажіть помилку, поясніть її, виведіть `__file__` імпортованого модуля, виправте і покажіть правильний вивід.

4. Продемонструйте різницю між `import grades` + `grades.X = ...` і `from grades import X` + `X = ...`. У модулі `limits.py` задайте `MAX_ABSENCES` рівним номеру вашого дня народження і функцію `is_allowed(absences: int) -> bool`. Покажіть вивід, поясніть, чому результати різні, і перепишіть функцію так, щоб межа передавалася параметром.

5. Створіть пакет `myschool` з модулями `grades.py` (обчислення) і `schedule.py` (ваш розклад на тиждень у вигляді словника та функція, що повертає пари на заданий день). У `schedule.py` використайте **відносний** імпорт з `grades.py`. Покажіть: (а) запуск `python3 myschool/schedule.py` і помилку; (б) запуск `python3 -m myschool.schedule` з кореня проєкту; (в) запуск `python3 -m myschool.schedule` з каталогу `myschool/`. Поясніть кожен результат через `sys.path[0]`.

6. Додайте до пакета `myschool` файл `__main__.py`, щоб `python3 -m myschool` виводив ваше імʼя, групу і розклад на сьогоднішній день тижня (`datetime.date.today().weekday()`). У `__init__.py` «підніміть» дві найважливіші функції і задайте `__all__`. Покажіть, що `from myschool import <function>` працює з окремого скрипта в корені проєкту.

7. Створіть два модулі, які імпортують один одного (наприклад, `students.py` з вашими даними та даними 3–4 одногрупників і `stats.py` з обчисленнями). Отримайте помилку циклічного імпорту, збережіть traceback і поясніть по кроках, що сталося. Намалюйте схему імпортів між модулями (можна mermaid) і позначте на ній, де виникає цикл.

8. Розбийте програму з домашнього завдання лекції 26 (або 23), яка читала файл і логувала пропущені рядки, на пакет за структурою з розділу «Структура проєкту»: окремі модулі для обчислень, читання файлу і виводу; точка входу — `__main__.py`; дані — у каталозі `data/`; логер у кожному модулі — `getLogger(__name__)`. Покажіть дерево проєкту, вивід `python3 -m <your_package>` (в лозі мають бути видні імена різних модулів), `.gitignore` і `README.md` з інструкцією запуску. Закомітьте проєкт у git і покажіть `git status` — у ньому не має бути `__pycache__`.

9. Знайдіть у своєму коді з попередніх лабораторних щонайменше три функції, які ви копіювали з однієї програми в іншу. Винесіть їх в окремий модуль (або пакет) `utils` і перепишіть дві старі програми так, щоб вони імпортували функції звідти. Покажіть `git diff` до і після та поясніть, які проблеми з розділу «Навіщо ділити програму на файли» це вирішує.
