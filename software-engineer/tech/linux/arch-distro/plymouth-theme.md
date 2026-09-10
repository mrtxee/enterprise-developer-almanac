---
aliases:
  - plymouth
  - plymouth-theme
---

Вы можете **посмотреть демо (предпросмотр) любой темы Plymouth** без перезагрузки системы. Это удобно, чтобы выбрать подходящую тему перед установкой через `plymouth-set-default-theme`.

Вот как это сделать:

---

## ✅ Шаг 1: Убедитесь, что Plymouth запущен

```bash
sudo systemctl status plymouth.service
```

Если не активен — запустите:

```bash
sudo systemctl start plymouth.service
```

---

### 🔹 Вариант B: Предпросмотр **конкретной темы** (без смены текущей)

Сначала посмотрите список доступных тем:

```bash
sudo plymouth-set-default-theme --list
sudo mkinitcpio -P
```

Пример вывода:

```
bgrt
details
fade-in
glow
script
solar
spinfinity
spinner
text
tribar
```

Теперь запустите демо **любой из них**, например `glow`:

```bash
sudo plymouth-set-default-theme details
sudo plymouthd --debug --debug-file=/tmp/plymouth-debug.log
sudo plymouth --show-splash
```

Симулируйте прогресс (как выше):

```bash
for i in {1..100}; do sudo plymouth --update=step-$i; sleep 0.1; done
```

И закройте:

```bash
sudo plymouth --quit
```

---

## ✅ Шаг 3: Быстрый просмотр на 5 секунд (для теста)

Если хотите просто **быстро глянуть**, как выглядит тема:

```bash
sudo plymouthd && sudo plymouth --show-splash && sleep 5 && sudo plymouth --quit
```

> Тема отобразится на 5 секунд и автоматически закроется.

---

## 💡 Советы

- Во время просмотра темы можно переключаться между терминалами: `Ctrl+Alt+F6` — в консоль, `Ctrl+Alt+F1` (или `F7`) — обратно в графику.
- Анимация логотипа и прогресс-бара появляется только при обновлении статуса (`--update`).
- Если ничего не отображается — проверьте, включён ли KMS (Kernel Mode Setting) для вашей видеокарты.

---

## 🧪 Пример: просмотр темы `spinfinity`

```bash
sudo plymouth-set-default-theme --list | grep spinfinity  # проверить наличие
sudo plymouthd --theme=spinfinity
sudo plymouth --show-splash
# Ждём 3 секунды
sleep 3
sudo plymouth --quit
```

---

Теперь вы можете **безопасно тестировать любую тему**, прежде чем устанавливать её как основную с помощью:

```bash
sudo plymouth-set-default-theme spinfinity -R
```

> Ключ `-R` автоматически пересоберёт `initramfs`, чтобы изменения вступили в силу при следующей загрузке.

Хотите, я помогу создать скрипт для удобного перебора всех тем?
