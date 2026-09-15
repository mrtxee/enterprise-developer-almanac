---
aliases:
  - plymouth
  - plymouth-theme
---

## Plymouth

**Демо (предпросмотр) любой темы Plymouth** можно посмотреть без перезагрузки системы. Это удобно, чтобы выбрать подходящую тему перед установкой через `plymouth-set-default-theme`.

Вот как это сделать:

### Проверка запуска Plymouth

```bash
sudo systemctl status plymouth.service
```

Если сервис не активен — запустите его:

```bash
sudo systemctl start plymouth.service
```

#### Предпросмотр конкретной темы

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

Запустите демо **любой из них**, например `glow`:

```bash
sudo plymouth-set-default-theme details
sudo plymouthd --debug --debug-file=/tmp/plymouth-debug.log
sudo plymouth --show-splash
```

Симулируйте прогресс:

```bash
for i in {1..100}; do sudo plymouth --update=step-$i; sleep 0.1; done
```

И закройте:

```bash
sudo plymouth --quit
```

### Быстрый просмотр на 5 секунд

Для быстрого просмотра, как выглядит тема:

```bash
sudo plymouthd && sudo plymouth --show-splash && sleep 5 && sudo plymouth --quit
```

> Тема отобразится на 5 секунд и автоматически закроется.

### Советы

- Во время просмотра темы можно переключаться между терминалами: `Ctrl+Alt+F6` — в консоль, `Ctrl+Alt+F1` (или `F7`) — обратно в графику.
- Анимация логотипа и прогресс-бара появляется только при обновлении статуса (`--update`).
- Если ничего не отображается — проверьте, включён ли KMS (Kernel Mode Setting) для вашей видеокарты.

### Пример: просмотр темы spinfinity

```bash
sudo plymouth-set-default-theme --list | grep spinfinity  # проверить наличие
sudo plymouthd --theme=spinfinity
sudo plymouth --show-splash
# Ждём 3 секунды
sleep 3
sudo plymouth --quit
```

Проверенную тему можно установить как основную с помощью:

```bash
sudo plymouth-set-default-theme spinfinity -R
```

> Ключ `-R` автоматически пересоберёт `initramfs`, чтобы изменения вступили в силу при следующей загрузке.
