---
aliases:
  - git
  - gitignore
---

## .gitignore

Чтобы папка `target` (и всё, что в ней) не попадала в коммит, нужно использовать **`.gitignore`** — это правильный и переносимый способ (не зависит от IDE). IntelliJ IDEA просто подхватит этот файл и перестанет предлагать эти файлы к коммиту.

### Исключение через .gitignore

1. В корне проекта создайте (или откройте) файл `.gitignore`.
2. Добавьте туда строку:
   ```gitignore
   target/
   ```

   Можно также написать `**/target/` — это покроет `target` в любых подпапках (актуально для мультимодульных [[maven|Maven]]‑проектов).

3. Если папка `target` уже была под [[git|Git]], её нужно убрать из отслеживания, но оставить на диске:
   ```bash
   git rm -r --cached target
   ```
4. Сделайте коммит с удалением из индекса:
   ```bash
   git commit -m "Stop tracking target directory"
   ```

После этого IntelliJ IDEA (и любой другой Git‑клиент) перестанет показывать файлы из `target` как изменённые.

### Типовой .gitignore

```gitignore
# --- IntelliJ IDEA ---
.idea/
*.iml
*.ipr
*.iws
out/

# --- Maven ---
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/wrapper/maven-wrapper.jar

# --- Gradle ---
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar

# --- Test & Coverage ---
coverage/
*.gcov
jacoco.exec

# --- Logs & Temp ---
*.log
tmp/
temp/

# --- Secrets & Local Configs ---
# ВАЖНО: не коммить реальные секреты, ключи и локальные профили
application-local.yml
application-local.properties
local.*
.env
.DS_Store
Thumbs.db

# --- OS & Editor ---
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Desktop.ini

# --- Binaries & Native ---
*.dll
*.so
*.dylib
native/

# --- IDE Cache & Indexes (дополнительно) ---
.classpath
.project
.settings/

# Игнорируем все файлы с расширением .env
*.env
# Но не игнорируем SAMPLE.env (где бы он ни лежал)
!SAMPLE.env
```

### Исключение из индекса лишних файлов

```bash
git rm --cached path/to/SAMPLE.env   # если вдруг он уже был под контролем версий
```
