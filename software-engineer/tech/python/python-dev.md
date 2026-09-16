---
aliases:
  - .env
  - .gitignore
  - activate.bat
  - aiogram
  - Anaconda
  - configparser
  - deactivate.bat
  - distutils
  - dotenv
  - FastAPI
  - Google Colab
  - Heroku
  - interpreter
  - ipynb
  - Jupyter
  - Jupyter Notebook
  - Jupyter-ноутбук
  - Package Installer for Python
  - pip
  - py2app
  - py2exe
  - PyInstaller
  - PyPA
  - PyPi
  - PyPI
  - python
  - Python
  - Python Development
  - requirements.txt
  - settings.ini
  - setup.py
  - setuptools
  - starlette
  - venv
  - virtual environment
  - виртуальная среда
  - виртуальное окружение
  - интерпретатор
  - Мидлваре
  - Разработка Python
---

## Разработка на Python

### Основные команды Shell

Основные команды окружения и pip:

```bash
choco install python
python --version
pip --version
# Обновить pip
python -m pip install --upgrade pip
python -m pip install -U pip
# Список пакетов
pip freeze
pip list
pip install package-name
pip install package-name --upgrade --force-reinstall
pip uninstall package-name
pip show package-name
```

### PaaS Heroku

Deploy the project:

```bash
cd c:\h\a\dev\tuyahomebot\bot4
git init
heroku login
heroku create mrtxeebot4
git add .                  # добавить в индекс все новые, изменённые, удалённые файлы из текущей директории и её поддиректорий
git commit -m "start"      # зафиксировать в коммите проиндексированные изменения и добавить сообщение
git remote -v              # показать список удалённых репозиториев, связанных с локальным
git push heroku master     # отправить данные ветки master в удалённый репозиторий heroku
heroku ps
heroku ps:scale worker=1
heroku ps
heroku logs --tail
heroku logs
```

`heroku config` — посмотреть переменные среды.

### Хранение конфигов

Для хранения конфигов Python рекомендует использовать файл `settings.ini` и модуль `configparser`.

Пример файла `settings.ini`:

```text
[Twitter]
username="johndoe"
password="johndoespassword"
token="....."
```

Чтение конфига через `configparser`:

```python
import configparser  # импортируем библиотеку
config = configparser.ConfigParser()  # создаём объект парсера
config.read("settings.ini")  # читаем конфиг
print(config["Twitter"]["username"])  # обращаемся как к обычному словарю; выведет 'johndoe'
```

#### Сокрытие конфига в git

Добавить `.env` в `.gitignore`:

```text
.env
```

Содержимое `.env`:

```text
BOT_TOKEN='super_secret_data'
```

Чтение секретов из `.env` в код:

```python
import os
from dotenv import load_dotenv
dotenv_path = os.path.join(os.path.dirname(__file__), '.env')
if os.path.exists(dotenv_path):
    load_dotenv(dotenv_path)
BOT_TOKEN = os.environ.get("BOT_TOKEN")
```

### pip

Package Installer for Python (pip) — установщик пакетов Python.

- `pip list` — список установленных пакетов.
- `pip install <package_name>` или `pip uninstall <package_name>` — установка/удаление пакета.
- Справочник по командам pip доступен на сайте PyPA.
- `pip show Jinja2` — узнать версию пакета.

PyPI — сайт, где описан список доступных пакетов для Python.

### logging

[[logging|Уровни логирования]]:

- `Debug (10)` — самый низкий уровень логирования, предназначенный для отладочных сообщений, для вывода диагностической информации о приложении.
- `Info (20)` — этот уровень предназначен для вывода данных о фрагментах кода, работающих так, как ожидается.
- `Warning (30)` — этот уровень логирования предусматривает вывод предупреждений, он применяется для записи сведений о событиях, на которые программист обычно обращает внимание. Такие события вполне могут привести к проблемам при работе приложения. Если явно не задать уровень логирования, по умолчанию используется warning.
- `Error (40)` — этот уровень логирования предусматривает вывод сведений об ошибках, о том, что часть приложения работает не так, как ожидается, о том, что программа не смогла правильно выполниться.
- `Critical (50)` — этот уровень используется для вывода сведений об очень серьёзных ошибках, наличие которых угрожает нормальному функционированию всего приложения. Если не исправить такую ошибку, это может привести к тому, что приложение прекратит работу.

Типовая настройка логгера: DEBUG — в файл, ERROR — в консоль:

```python
import logging
from logging.handlers import RotatingFileHandler
# типовая настройка логгера с выводом DEBUG в файл, а ERROR в консоль
logger_formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
# logger_file_handler = logging.FileHandler(f'{__file__}.log')
# logger_file_handler = TimedRotatingFileHandler(f'{__file__}.log', when="s", interval=1, backupCount=5)
logger_file_handler = logging.handlers.RotatingFileHandler(f'{__name__}.log', maxBytes=51200, backupCount=2)
logger_file_handler.setLevel(logging.DEBUG)
logger_file_handler.setFormatter(logger_formatter)
logger_stream_handler = logging.StreamHandler()
logger_stream_handler.setLevel(logging.ERROR)
logger_stream_handler.setFormatter(logger_formatter)
# getLogger(__name__) — имя логгера совпадает с именем логгера в других модулях программы,
# чтобы сообщения собирались согласно данным настройкам
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.addHandler(logger_stream_handler)
logger.addHandler(logger_file_handler)
logger.info("No custom logger provided. Default logger is on")
# DEBUG и INFO не попадают в консоль
logging.debug("A DEBUG Message")  # ''
logging.info("An INFO")  # ''
logging.warning("A WARNING")  # WARNING:root:A WARNING
logging.error("An ERROR")  # ERROR:root:An ERROR
logging.critical("A message of CRITICAL severity")  # CRITICAL:root:A message ...
```

### Виртуальные окружения Python

**virtual environment**. Основная цель виртуального окружения Python — создание изолированной среды для Python-проектов.

#### Создание виртуальной среды

Команды создания и активации виртуальной среды:

```bash
cd c:\h\a\dev\django1\
python -m venv .venv
# активация в cmd:
# .venv\Scripts\activate.bat
# активация в PowerShell (требуется изменить политику выполнения):
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.venv\Scripts\Activate.ps1
# (venv1) c:\h\a\dev\django1> — маркер нахождения в виртуальной среде venv1
python -m pip install --upgrade pip
pip install django
# pip install djangorestframework
# pip install django-allauth
# pip install -r requirements.txt
django-admin --version
# создание нового проекта с одним приложением
django-admin startproject project1 .  # обратите внимание на завершающую точку
cd project1
django-admin startapp app1
cd ..
python manage.py migrate
python manage.py createsuperuser --email mrtxee@bk.ru --username admin
# подготовить код приложений admin
python manage.py runserver
```

Флаг `-m` указывает Python запустить модуль `venv` как исполняемый. `venv1/` — название виртуального окружения, где хранятся библиотеки проекта. В результате создаётся каталог `venv1/` с копией интерпретатора Python, стандартной библиотекой и другими вспомогательными файлами. Чтобы начать пользоваться окружением, нужно запустить файл `venv1\Scripts\activate.bat`, после чего можно устанавливать требуемые пакеты. При переключении между виртуальными средами каждый раз выполняется `activate.bat`/`deactivate.bat`. В PowerShell используется файл `.ps1`.

#### Список установленных пакетов

Вывод списка установленных пакетов: `pip freeze`.

### aiogram

#### Терминология aiogram

- **ЛС** — личные сообщения. В контексте бота это диалог один-на-один с пользователем, а не группа или канал.
- **Чат** — общее название для ЛС, групп, супергрупп и каналов.
- **Апдейт** — любое событие из этого списка: сообщение, редактирование сообщения, колбэк, инлайн-запрос, платёж, добавление бота в группу и т.д.
- **Хэндлер** — асинхронная функция, которая получает от диспетчера/роутера очередной апдейт и обрабатывает его.
- **Диспетчер** — объект, занимающийся получением апдейтов от Telegram с последующим выбором хэндлера для обработки принятого апдейта.
- **Роутер** — аналогично диспетчеру, но отвечает за подмножество множества хэндлеров. Можно сказать, что диспетчер — это корневой роутер.
- **Фильтр** — выражение, которое обычно возвращает True или False и влияет на то, будет вызван хэндлер или нет.
- **Мидлварь** — прослойка, которая вклинивается в обработку апдейтов.

### FastAPI

Фреймворк для создания асинхронных веб-сервисов на Python. Надстройка над фреймворком `starlette`, который служит для той же задачи.

### Packaging

#### Пакетирование для PyPI

**Пакет**, **package** — набор взаимосвязанных модулей. Сам пакет тоже является модулем. Предназначен для решения задач определённого класса некоторой предметной области. Это способ структуризации модулей. Пакет — это папка, в которой содержатся модули, другие пакеты и обязательные файлы `__init__.py`, отвечающие за инициализацию пакета, и `setup.py`.

Структура пакета:

```text
SoloLearn/
   LICENSE.txt
   README.txt
   setup.py
   sololearn/
      __init__.py
      sololearn.py
      sololearn2.py
```

Для создания пакета используются модули `setuptools` и `distutils`:

```python
from distutils.core import setup
setup(
   name='SoloLearn',
   version='0.1dev',
   packages=['sololearn',],
   license='MIT',
   long_description=open('README.txt').read(),
)
```

Файл `setup.py` необходим для того, чтобы пакет можно было выложить на PyPI и установить через pip.

Сборка и публикация дистрибутива:

```bash
python setup.py sdist
python setup.py bdist
python setup.py bdist_wininst
python setup.py register
python setup.py sdist upload
python setup.py install
```

#### Пакетирование для операционной системы

Для пакетирования приложения Python для операционной системы доступны инструменты:

| ОС | Инструменты пакетирования |
| --- | --- |
| Windows | `py2exe`, `PyInstaller`, `cx_Freeze` |
| macOS | `py2app`, `PyInstaller`, `cx_Freeze` |

### Jupyter Notebook

**Jupyter-ноутбук** — это интерактивная среда разработки, в которой сразу виден результат выполнения кода и его отдельных фрагментов. В отличие от традиционной среды разработки код можно разбить на части и выполнять их в произвольном порядке. Поддерживает языки Ruby, Perl, R, bash, Python. Данные хранятся в файлах `*.ipynb` (формат `JSON`).

**Реализации**

- Google Colab.
- Anaconda.
- `pip install jupyter` — установка через pip, запуск командой `jupyter notebook`.
- VS Code.

Применяется преимущественно для задач [[python-data-processing|Data Science]], Machine Learning, обучения и т.п.
