# Ubuntu Basic setup (server)

- [laptop lid](#setup-close-laptop-lid)
- [ssh](#ubuntu-setup-ssh)

## setup close laptop lid

ใช้ nano สำหรับแก้ไขการ config เดิมของ ubuntu

```bash
sudo nano /etc/systemd/logind.conf
```

จะเพิ่มหรือแก้ไข ก็ได้

```bash
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
```

> default เดิม ของ HandleLidSwitch คือ HandleLidSwitch=suspend

ทำการ reboot ระบบ loggin เพื่อใช้งาน

```bash
sudo sudo systemctl restart systemd-logind.service
```

เสริมสำหรับตั้งค่า line console

```bash
sudo nano /etc/default/grub
```

```bash
GRUB_CMDLINE_LINE="consoleblank=300"
```

> default เดิม ของ GRUB_CMDLINE_LINE คือ GRUB_CMDLINE_LINE=""

```bash
sudo update-grub
```

## ubuntu setup ssh

```bash
sudo apt install openssh-server
sudo systemctl status ssh
sudo ufw allow ssh
```
