`find` — одна из самых мощных команд Linux для поиска файлов и папок. Вот подробное руководство.

---

## 🔍 Базовый синтаксис

```bash
find [путь] [условия] [действия]
```

Если путь не указан, поиск начнётся с текущей директории (`.`).

---

## 📁 Поиск по имени

### Точное имя
```bash
find /home -name "file.txt"
```

### Поиск без учёта регистра
```bash
find /home -iname "file.txt"    # найдёт и File.txt, и FILE.TXT
```

### По маске (wildcards)
```bash
find /home -name "*.gguf"       # все .gguf файлы
find /home -name "photo-*"      # все файлы, начинающиеся с "photo-"
find /home -name "file.???"     # file.txt, file.log, file.jpg (ровно 3 символа в расширении)
```

### Поиск и файлов, и папок с именем
```bash
find /home -name "Downloads"    # найдёт и папку Downloads, и файл с таким именем
```

---

## 📄 Фильтрация по типу

### Только файлы
```bash
find /home -type f -name "*.txt"
```

### Только папки
```bash
find /home -type d -name "project"
```

### Символические ссылки
```bash
find /home -type l
```

---

## 📊 Поиск по размеру

```bash
find /home -type f -size +100M    # файлы больше 100 МБ
find /home -type f -size -1M      # файлы меньше 1 МБ
find /home -type f -size 50k      # файлы ровно 50 КБ
find /home -type f -size +1G      # файлы больше 1 ГБ
```

Единицы: `c` (байты), `k` (КБ), `M` (МБ), `G` (ГБ).

---

## ⏰ Поиск по времени

### По дате изменения (содержимого)
```bash
find /home -type f -mtime -7      # изменённые за последние 7 дней
find /home -type f -mtime +30     # изменённые более 30 дней назад
```

### По дате доступа (чтения)
```bash
find /home -type f -atime -1      # прочитанные за последние 24 часа
```

### В минутах (для точности)
```bash
find /home -type f -mmin -60      # изменённые за последние 60 минут
```

---

## 👤 Поиск по владельцу и правам

### По владельцу
```bash
find /home -user username
```

### По группе
```bash
find /home -group groupname
```

### По правам доступа
```bash
find /home -type f -perm 755      # файлы с правами ровно 755
find /home -type f -perm -u+w     # файлы, доступные для записи владельцу
```

---

## 🧠 Комбинирование условий

### И (AND) — указать несколько условий подряд
```bash
find /home -type f -name "*.txt" -size +1M
```

### ИЛИ (OR) — использовать `-o`
```bash
find /home -name "*.txt" -o -name "*.md"
```

### Отрицание (NOT)
```bash
find /home -type f -not -name "*.log"
find /home -type f ! -name "*.log"    # ! — короткая форма -not
```

### Группировка условий
```bash
find /home \( -name "*.txt" -o -name "*.md" \) -size +1M
```

---

## ⚡ Действия над найденным

### Просто вывести (по умолчанию)
```bash
find /home -name "*.txt"
```

### Удалить все найденные файлы
```bash
find /home -name "*.tmp" -delete
```

### Выполнить команду для каждого файла
```bash
find /home -name "*.txt" -exec chmod 644 {} \;
```

### Выполнить команду для нескольких файлов сразу (быстрее)
```bash
find /home -name "*.txt" -exec chmod 644 {} +
```

### Вывести с подробной информацией
```bash
find /home -name "*.txt" -ls
```

### Переместить найденные файлы
```bash
find /home -name "*.jpg" -exec mv {} /backup/ \;
```

### Интерактивное выполнение (с подтверждением)
```bash
find /home -name "*.tmp" -ok rm {} \;
```

---

## 🎯 Полезные готовые примеры

### Найти и удалить пустые файлы
```bash
find /home -type f -empty -delete
```

### Найти и удалить пустые папки
```bash
find /home -type d -empty -delete
```

### Найти 10 самых больших файлов в домашней папке
```bash
find /home -type f -exec du -h {} + | sort -rh | head -10
```

### Найти файлы, изменённые за последние 2 часа
```bash
find /home -type f -mmin -120
```

### Найти все изображения (по расширениям)
```bash
find /home -type f \( -name "*.jpg" -o -name "*.png" -o -name "*.gif" \)
```

### Поиск файлов, принадлежащих определённому пользователю, больше 100 МБ
```bash
find /home -type f -user username -size +100M
```

### Игнорировать определённые папки
```bash
find /home -type f -not -path "*/node_modules/*" -name "*.js"
```

---

## 🚀 Советы

- Всегда тестируйте команду с `-delete` или `-exec rm` **сначала без них**, чтобы увидеть, что будет удалено.
- Используйте `2>/dev/null` в конце, чтобы скрыть ошибки "Permission denied".
- Для очень больших директорий добавляйте `-maxdepth N`, чтобы ограничить глубину поиска:
  ```bash
  find /home -maxdepth 2 -name "*.txt"   # искать только в /home и его подпапках первого уровня
  ```