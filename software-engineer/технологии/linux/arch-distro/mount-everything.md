## mount-everything
```bash
lsblk -f

sudo mkdir /mnt/d1
sudo mkdir /mnt/sd
sudo mkdir /mnt/wb
sudo mkdir /mnt/do

sudo pacman -S ntfs-3g
sudo mount -t ntfs-3g /dev/sda1 /run/media/a/sd
sudo mount -t ntfs-3g /dev/nvme0n1p8 /mnt/di
sudo mount -t ntfs-3g /dev/nvme0n1p3 /mnt/bd

sudo nano /etc/fstab

# IF raea-only filesystem error
# --> use ntfs-3g, ex.:
/dev/disk/by-id/usb-Generic_MassStorageClass_000000002402-0:0-part1 /mnt/usb-Generic_MassStorageClass_000000002402-0:0-part1 ntfs-3g nosuid,nodev,nofail,x-gvfs-show 0 0

# sudo pacman -S davfs2
# echo "https://webdav.cloud.mail.ru amt779@bk.ru crwyOZdqkz0GeKCGN186" | sudo tee -a /etc/davfs2/secrets
# sudo chmod 600 /etc/davfs2/secrets
# sudo chown root:root /etc/davfs2/secrets
# sudo mount -t davfs https://webdav.cloud.mail.ru /mnt/do

```