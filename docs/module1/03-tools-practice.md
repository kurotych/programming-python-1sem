# 3. (П1) Налаштування середовища розробки. Запуск Python коду локально та онлайн

## Мета роботи

Налаштувати робоче середовище програміста, написати та запустити першу власну програму локально, а потім виконати аналогічну програму в онлайн-інтерпретаторі та порівняти обидва способи роботи.

## Хід роботи

### Завдання 1. Встановити Python 3

Перевірте, чи Python уже встановлений:

```bash
python3 --version
```

Якщо команда не знайдена — встановіть Python за інструкцією з лекції для вашої системи.

**Результат:** термінал виводить версію, наприклад `Python 3.13.2`.

### Завдання 2. Встановити редактор коду

Виберіть **один** варіант — А(простіший) або Б(складніший і підходить далеко не всім).

#### Варіант А — Visual Studio Code

1. Встановіть VS Code (з [code.visualstudio.com](https://code.visualstudio.com/) або через пакетний менеджер).
2. Відкрийте панель розширень: `Ctrl+Shift+X`.
3. Встановіть розширення **Python** від Microsoft (разом із ним встановиться **Pylance**).
4. Натисніть `Ctrl+Shift+P`, введіть `Python: Select Interpreter` і виберіть встановлений Python 3.
5. Переконайтеся, що вибраний інтерпретатор видно в рядку стану внизу вікна.

#### Варіант Б — Neovim

1. Встановіть Neovim:

    ```bash
    # Debian / Ubuntu
    sudo apt install neovim
    ```

2. Перевірте версію:

    ```bash
    nvim --version
    ```

3. Створіть мінімальний конфігураційний файл `~/.config/nvim/init.lua`:

    ```lua
    vim.opt.number = true          -- показувати номери рядків
    vim.opt.expandtab = true       -- пробіли замість табуляції
    vim.opt.shiftwidth = 4         -- відступ 4 пробіли
    vim.opt.tabstop = 4            -- символ табуляції показувати як 4 пробіли
    vim.opt.mouse = "a"            -- підтримка миші
    vim.opt.clipboard = "unnamedplus"  -- спільний буфер обміну із системою:
                                       -- скопійоване в Neovim можна вставити
                                       -- в браузер і навпаки
    ```

4. **Опціонально** (для тих, хто хоче автодоповнення та підказки): встановіть менеджер плагінів [lazy.nvim](https://github.com/folke/lazy.nvim) і додайте:

    - [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) разом із мовним сервером `pyright` (`pip install pyright`) — підказки про помилки;
    - [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) — автодоповнення;
    - [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) — якісне підсвічування синтаксису.

!!! tip "Мінімум команд Neovim"
    `i` — режим вставки, `Esc` — вийти з нього, `:w` — зберегти, `:q` — вийти, `:wq` — зберегти та вийти, `:!python3 %` — запустити поточний файл.

**Результат:** редактор встановлено та відкрито, підсвічування синтаксису Python працює.

### Завдання 3. Створити проєкт і написати програму

1. Створіть каталог для курсу та перейдіть у нього:

    ```bash
    mkdir -p ~/python-course/practice03
    cd ~/python-course/practice03
    ```

2. Відкрийте цей каталог у редакторі:

    ```bash
    # VS Code
    code ~/python-course/practice03

    # Neovim
    nvim greeting.py
    ```

    !!! warning "Для VS Code"
        Відкривайте саме **каталог** (`File → Open Folder`), а не окремий файл — інакше не працюватимуть відносні шляхи та термінал у каталозі проєкту.

3. Створіть файл `greeting.py` і напишіть програму, яка вітає **саме вас**. Замініть значення в коді на власні дані:

    ```python
    # Personalized greeting program
    name = "Anatolii"
    surname = "Kurotych"
    group = "PZ-11"
    birth_year = 2007
    current_year = 2026

    age = current_year - birth_year
    full_name = name + " " + surname

    print("Hello, world!")
    print("My name is", full_name)
    print("Group:", group)
    print("I am", age, "years old")
    print("Letters in my full name:", len(full_name))
    print("Loud version:", full_name.upper())
    ```

4. Збережіть файл (`Ctrl+S` у VS Code, `:w` у Neovim).

**Очікуваний вивід** (з вашими даними):

```text
Hello, world!
My name is Anatolii Kurotych
Group: PZ-11
I am 19 years old
Letters in my full name: 17
Loud version: ANATOLII KUROTYCH
```

### Завдання 4. Запустити програму локально

1. Відкрийте термінал у каталозі проєкту (у VS Code — ``Ctrl+` ``).
2. Запустіть програму:

    ```bash
    python3 greeting.py
    ```

3. Переконайтеся, що вивід збігається з очікуваним.
4. У VS Code запустіть програму ще раз кнопкою ▷ або комбінацією `Ctrl+F5` — результат має бути таким самим.

!!! danger "Якщо бачите `No such file or directory`"
    Ви запускаєте команду не з того каталогу. Перевірте, де ви зараз (`pwd`), і що лежить поруч (`ls`).

**Результат:** знімок екрана з кодом у редакторі та виводом програми в терміналі.

### Завдання 5. Виконати код в онлайн-інтерпретаторі

1. Відкрийте будь-який онлайн-інтерпретатор із таблиці в лекції, наприклад:

    - [python.org/shell](https://www.python.org/shell/)
    - [programiz.com/python-programming/online-compiler](https://www.programiz.com/python-programming/online-compiler/)
    - [pythontutor.com](https://pythontutor.com/)

2. Вставте туди програму з власними даними.
    ```python
    # Online version: data is set directly in the code
    name = "Anatolii"
    group = "PZ-11"
    editor = "VS Code"

    print("Hello from an online interpreter!")
    print("Author:", name)
    print("Group:", group)
    print("My local editor:", editor)
    print("2 ** 10 =", 2 ** 10)
    ```

3. Запустіть код і зробіть знімок екрана разом із виводом.
4. Запишіть, **який саме сервіс** ви використали.

### Завдання 6. Покрокове виконання в Python Tutor

1. Відкрийте [pythontutor.com](https://pythontutor.com/) → *Python 3*.
2. Вставте цю програму:

    ```python
    name = "Anatolii"
    total = 0

    for i in range(1, 6):
        total = total + i
        print("i =", i, "total =", total)

    print(name, "counted the sum:", total)
    ```

3. Натисніть **Visualize Execution** і пройдіть програму по кроках кнопкою *Next*.
4. Занотуйте, як змінюються значення `i` та `total` на кожному кроці.

**Результат:** знімок екрана з візуалізацією на будь-якому проміжному кроці.
