## Markdown Formatting Rules for AI Agent

```plain

1. Front matter: every file MUST start with a YAML block: --- then aliases: list of strings, then ---. Create if missing. aliases cover the key technical terms of content; every Russian term must be paired with an equivalent English technical term (e.g. "KD-дерево" + "KD Tree").

2. Headings: ATX (#..######). No trailing #. No numbering prefix ("1.", "Шаг 1:"). No bold/italic formatting inside a heading (keep `code` spans). No level skipping (avoid ## -> ####). Meaningful, reflect content. If file has no heading at all, add one for its main topic.

3. Emoji (U+10000–U+10FFFF) in headings: keep if used ≤1 time on page; remove all instances if repeated in ≥2 headings.

4. Lists: marker "-" (never "*"); nesting = 2 spaces; max 4 levels; first level flush left. Fix broken numbering in ordered lists. Remove stray/extra blank lines inside lists.

5. Code blocks: fenced with language tag (e.g. ```java, ```sql, ```bash, ```yaml, ```text). 2-space indentation inside. A short plain descriptive header line must precede each block. Convert dangling raw code lines (e.g. bare "python"/"sql" words followed by code, indented comments) into proper fenced blocks. Verbs like "Пример ...", "Создание индекса ..." are fine headers.

6. Text: escape literal square brackets \[ ... \] when NOT inside a code block or wiki link. Remove external web links (http(s) and markdown-style [text](http...)); keep Obsidian [[wiki-link]] and \![[images]]. Tag mentioned people right after name with " #👨" and books with " #📘" (as done in edit-distance.md). Standardize Pros/Cons: merge "Плюсы:"/"Минусы:" sections into one "Особенности:" list with inline ✅/❌ bullets (see examples.md; tables titled Плюсы/Минусы may stay).

7. Diagrams: always render in Mermaid if possible.

8. Mermaid blocks must start inside the fence with: --- then title: <descriptive title> then ---.

9. Formulas: LaTeX via $...$ (short) or $$...$$ (display). Fix broken math fences like $$$$.

10. Whitespace: exactly one blank line between logical blocks (max 2 consecutive blank lines). No trailing whitespace any line. Final newline at EOF. One space after punctuation. Fix mangled segments like `- **Оператор `<->`** ...` => `- **Оператор `<->`** — ...` style.

11. NEVER rename or delete files. Write in place (UTF-8). NEVER touch README.md (root), the activities/ dir, or attachments/ dir, and nothing outside your assigned scope.

12. Remove duplicate sections: merge repeated content into a single concise block, eliminating redundancy and filler text.
```

EXAMPLES: /home/a/Documents/GitHub/enterprise-developer-almanac/attachments/agent/examples.md
