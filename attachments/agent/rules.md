## Rules for AI Agent (Правила работы ИИ агента)

### Ignored Files
- `README.md` in root — NEVER read or modify
- `activities` directory in root — NEVER read or modify
- `attachments` directory in root — NEVER modify

### Правила форматирования
#### Заголовки
- Use ATX-style headings (`#`, `##`, `###`, `####`, etc)
- No trailing `#` symbols
- Заголовки никогда не содержат форматирования отличающегося от последовательных уровней заголовка (`#`, `##`, `###`, `####`, etc)
  - Если заголовок содержит нумерацию - удалить нумерацию
  - Если заголовок содержит **жирное выделение** или *курсив* - удалить форматирование
  - Если заголовок содержит `код` - оставить
- Заголовки без пропуска уровней
- Если в файле отсутствует Заголовок, следует прочитать файл и выявить основную тему текста и добавить его
- Все заголовки всегда осмысленные, заголовки отражают содержание текста

#### Изображения
- Store in `attachments/`
- Use descriptive alt text

#### Mermaid-diagrams
- всегда содержат заголовок вида: `title: <descriptive title>`

#### Блоки кода
- Specify language
- Use 2-space indentation inside
- Над каждым блоком кода должен быть краткий заголовок (строка без форматирования), объясняющая содержимое блока кода. Если такой нету, её следует добавить.
- Код: блочный код с указанием языка, inline-код через одинарные backticks

#### Пробельные символы
- Single blank line between paragraphs
- No trailing whitespace
- Final newline at end of file
- Пробелы: один пробел после знаков препинания, пустая строка между блоками
- Пустые строки: ровно одна пустая строка между логическими блоками, не больше двух подряд

#### Списки
- Списки: маркеры `-` (не `*`), вложенность через 2 пробела, не больше 4 уровней
	- Список первого уровня без отступов (пробелов, табуляций) от начала строки
- Если в нумерованном списке нарушена наумерация, следует исправить ее.
- убрать лишние переводы строк в списках

#### Текст
- Экранировать квадратные скобки, если они не часть блока кода
- Убрать из текста ссылки на внешние ресурсы
- Если в тексте есть упоминание человека, либо книги, необходимо макировать это упоминание соответствующим тегом
- Стандартизировать все фрагменты Pros/Cons

#### Формулы
- Все формулы должны быть в latex, через одинарный либо двойной знак $, в зависимости от длины формулы.
### Front matter Rules
- Каждый файл всегда начинается с Front matter содержит
- Front matter содержит обязательный тег aliases
- Если aliases отсутствует его надо создать и заполнить
- aliases всегда включает в себя ключевые технические термины, которые рассмотрены в контентной части документа
- Если некоторые термины из контентной части документа упущены, их следует добавить
- Каждый русский термин в aliases обязательно должен быть продублирован английским известным техническим термином, означающим то же самое

### Ограничения
- ИИ агент не меняет содержимое файлов, он только исправляет форматирование по правилам и заполняет aliases



AUTHORITATIVE RULES: /home/a/Documents/GitHub/enterprise-developer-almanac/attachments/agent/rules.md
EXAMPLES: /home/a/Documents/GitHub/enterprise-developer-almanac/attachments/agent/examples.md

1. Front matter: every file MUST start with a YAML block: --- then aliases: list of strings, then ---. Create if missing. aliases cover the key technical terms of content; every Russian term must be paired with an equivalent English technical term (e.g. "KD-дерево" + "KD Tree").
2. Headings: ATX (#..######). No trailing #. No numbering prefix ("1.", "Шаг 1:"). No bold/italic formatting inside a heading (keep `code` spans). No level skipping (avoid ## -> ####). Meaningful, reflect content. If file has no heading at all, add one for its main topic.
3. Convert standalone bold pseudo-headings like **Как работает?** or **Плюсы и минусы** (acting as headers) into real headings ### ... (keeps heading text without **).
4. Lists: marker "-" (never "*"); nesting = 2 spaces; max 4 levels; first level flush left. Fix broken numbering in ordered lists. Remove stray/extra blank lines inside lists.
5. Code blocks: fenced with language tag (e.g. ```java, ```sql, ```bash, ```yaml, ```text). 2-space indentation inside. A short plain descriptive header line must precede each block. Convert dangling raw code lines (e.g. bare "python"/"sql" words followed by code, indented comments) into proper fenced blocks. Verbs like "Пример ...", "Создание индекса ..." are fine headers.
6. Text: escape literal square brackets \[ ... \] when NOT inside a code block or wiki link. Remove external web links (http(s) and markdown-style [text](http...)); keep Obsidian [[wiki-link]] and ![[images]]. Tag mentioned people right after name with " #👨" and books with " #📘" (as done in edit-distance.md). Standardize Pros/Cons: merge "Плюсы:"/"Минусы:" sections into one "Особенности:" list with inline ✅/❌ bullets (see examples.md; tables titled Плюсы/Минусы may stay).
7. Mermaid blocks must start inside the fence with: --- then title: <descriptive title> then ---.
8. Formulas: LaTeX via $...$ (short) or $$...$$ (display). Fix broken math fences like $$$$.
9. Whitespace: exactly one blank line between logical blocks (max 2 consecutive blank lines). No trailing whitespace any line. Final newline at EOF. One space after punctuation. Fix mangled segments like `- **Оператор `<->`** ...` => `- **Оператор `<->`** — ...` style.
10. DO NOT change the meaning/content of the text, facts, numbers, formula values, table data, or code logic. Only formatting + aliases.
11. NEVER rename or delete files. Write in place (UTF-8). NEVER touch README.md (root), the activities/ dir, or attachments/ dir, and nothing outside your assigned scope.


TODO:
- исправить экстрагированные правила, переписать все правила лаконично на английском
- написать правила про линкование, которое должно быть в тексте, но не в заголовках
- разработай правило про пиктограммы