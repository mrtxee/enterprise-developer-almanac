---
aliases:
  - Markdown‑links
  - Markdown‑ссылки
  - obsidian
  - Wikilinks
---
**Wikilinks** (`[[...]]`) и **Markdown‑ссылки** (`[текст](путь)`) в Obsidian отличаются по **стандарту, синтаксису, поведению и переносимости**. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1mj1vp7/why_do_internal_and_external_links_text/)

## Главное отличие

- **Wikilinks** — это **Obsidian‑специфичный** формат ссылок, не входящий в стандарт Markdown. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1mj1vp7/why_do_internal_and_external_links_text/). Wikilinks не поддерживает внешний ссылки.
- **Markdown‑ссылки** — это **стандартный** синтаксис Markdown, который понимают почти все редакторы и системы. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1mj1vp7/why_do_internal_and_external_links_text/)

## Синтаксис и порядок аргументов

- **Wikilink:** `[[Цель|Текст]]` — сначала **цель** (имя заметки/путь), потом через `|` отображаемый текст [obsibrain](https://www.obsibrain.com/blog/obsidian-linking-the-complete-guide-to-connecting-your-notes).
- **Markdown‑ссылка:** `[Текст](Цель)` — сначала **текст**, потом в скобках **цель** (путь/URL). [obsibrain](https://www.obsibrain.com/blog/obsidian-linking-the-complete-guide-to-connecting-your-notes)

## На что удобно ссылаться

- **Wikilinks** в Obsidian работают **по имени заметки**, автоматически обновляются при переименовании файла и поддерживают заголовки/блоки через `#` и `#^`. [obsibrain](https://www.obsibrain.com/blog/obsidian-linking-the-complete-guide-to-connecting-your-notes)
- **Markdown‑ссылки** обычно требуют **путь к файлу** (относительный или абсолютный), при переименовании могут «ломаться», если не включена специальная настройка Obsidian. [obsibrain](https://www.obsibrain.com/blog/obsidian-linking-the-complete-guide-to-connecting-your-notes)

## Backlinks и граф

- **Wikilinks** полноценно участвуют в системе **backlinks** и отображаются в **графе связей**. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1c325y3/wiki_links_vs_markdown_links/)
- **Markdown‑ссылки** внутри хранилища **могут не создавать backlinks** и не всегда корректно отображаются в графе, особенно если используются как «внешние» ссылки. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1c325y3/wiki_links_vs_markdown_links/)

## Переносимость

- **Wikilinks** читаются только в Obsidian (и некоторых других инструментах, которые их специально поддерживают). [reddit](https://www.reddit.com/r/ObsidianMD/comments/1c325y3/wiki_links_vs_markdown_links/)
- **Markdown‑ссылки** работают везде, где есть Markdown: GitHub, GitLab, MkDocs, Docusaurus, обычные редакторы и т. п.. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1c325y3/wiki_links_vs_markdown_links/)

## Когда что использовать

- **Wikilinks** — если ты работаешь **внутри Obsidian** и хочешь удобные связи, автообновление при переименовании, backlinks и граф. [obsibrain](https://www.obsibrain.com/blog/obsidian-linking-the-complete-guide-to-connecting-your-notes)
- **Markdown‑ссылки** — если важна **переносимость** заметок за пределы Obsidian или нужна совместимость с другими инструментами и стандартами. [reddit](https://www.reddit.com/r/ObsidianMD/comments/1c325y3/wiki_links_vs_markdown_links/)

Если нужно, могу показать параллельные примеры одной и той же ссылки в обоих форматах.
