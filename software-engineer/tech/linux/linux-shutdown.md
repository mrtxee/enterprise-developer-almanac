---
aliases:
  - linux-poweroff
  - linux-shutdown
  - poweroff
  - shutdown
---

shutdown -h +80

```bash
sudo pacman -S at
sudo systemctl enable --now atd
Created symlink '/etc/systemd/system/multi-user.target.wants/atd.service' → '/usr/lib/systemd/system/atd.service'.

echo "systemctl poweroff" | at now + 60 minutes
warning: commands will be executed using /bin/sh
job 2 at Sat Aug  8 02:15:00 2026
 ~ at 01:15:19 
> echo "systemctl poweroff" | at now + 60 minutes

```
