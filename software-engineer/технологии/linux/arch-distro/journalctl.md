## читать системный журнал
```bash
journalctl -f -u gnome-software
journalctl -f | grep -i "packagekit\|pacman\|install\|remove"
systemctl --user status packagekit
```


May 12 23:10:10 an5l systemd[3129]: Started dbus-:1.2-org.gnome.Console@1.service