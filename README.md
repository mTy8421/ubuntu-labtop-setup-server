# 💻 Ubuntu Laptop Setup for Home Server & Desktop

คู่มือการตั้งค่า Ubuntu สำหรับนำแล็ปท็อป (Laptop) มาปรับแต่งเป็น Home Server หรือเครื่อง Desktop สำหรับงาน Development พร้อมเทคนิคการจัดการพลังงานและหน้าจอ

---

## 📑 สารบัญ (Table of Contents)

- [1. การตั้งค่าระบบ Server บน Laptop](#1-การตั้งค่าระบบ-server-บน-laptop)
  - [1.1 ปิดฝาพับโน้ตบุ๊กไม่ให้เครื่องดับ/Sleep (Lid Close Settings)](#11-ปิดฝาพับโน้ตบุ๊กไม่ให้เครื่องดับsleep-lid-close-settings)
  - [1.2 พักหน้าจออัตโนมัติเพื่อถนอมจอภาพ (Console Blanking)](#12-พักหน้าจออัตโนมัติเพื่อถนอมจอภาพ-console-blanking)
  - [1.3 ปิดระบบ Sleep / Suspend โดยสิ้นเชิง (ทางเลือกเสริม)](#13-ปิดระบบ-sleep--suspend-โดยสิ้นเชิง-ทางเลือกเสริม)
  - [1.4 ปิดโหมดประหยัดพลังงานของ Wi-Fi](#14-ปิดโหมดประหยัดพลังงานของ-wi-fi)
- [2. การตั้งค่าการเชื่อมต่อระยะไกล (SSH Setup)](#2-การตั้งค่าการเชื่อมต่อระยะไกล-ssh-setup)
  - [2.1 ติดตั้งและเปิดใช้งาน OpenSSH Server](#21-ติดตั้งและเปิดใช้งาน-openssh-server)
  - [2.2 ตรวจสอบ IP Address และทดสอบการเชื่อมต่อ](#22-ตรวจสอบ-ip-address-และทดสอบการเชื่อมต่อ)
  - [2.3 การตั้งค่า Firewall (UFW)](#23-การตั้งค่า-firewall-ufw)
- [3. การตั้งค่าสำหรับ Ubuntu Desktop & Development](#3-การตั้งค่าสำหรับ-ubuntu-desktop--development)
  - [3.1 อัปเดตระบบ](#31-อัปเดตระบบ)
  - [3.2 ติดตั้ง Timeshift สำหรับสำรองข้อมูล](#32-ติดตั้ง-timeshift-สำหรับสำรองข้อมูล)
  - [3.3 ติดตั้งเครื่องมือและชุดพัฒนา (Development Tools)](#33-ติดตั้งเครื่องมือและชุดพัฒนา-development-tools)
  - [3.4 แนะนำเครื่องมือมอนิเตอร์และบริหารระบบ](#34-แนะนำเครื่องมือมอนิเตอร์และบริหารระบบ)
- [4. เอกสารที่เกี่ยวข้อง](#4-เอกสารที่เกี่ยวข้อง)

---

## 1. การตั้งค่าระบบ Server บน Laptop

เมื่อนำโน้ตบุ๊กมาทำเป็น Server ส่วนใหญ่มักต้องการปิดฝาพับ (Clamshell mode) เพื่อประหยัดพื้นที่ โดยไม่ให้เครื่องตัดเข้าโหมด Suspend หรือปิดการทำงาน

### 1.1 ปิดฝาพับโน้ตบุ๊กไม่ให้เครื่องดับ/Sleep (Lid Close Settings)

แก้ไขไฟล์คอนฟิกของ `systemd-logind`:

```bash
sudo nano /etc/systemd/logind.conf
```

ค้นหาและแก้ไข (หรือเพิ่ม) บรรทัดต่อไปนี้:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

> **คำอธิบาย:**
> - ค่าเดิมตามปกติคือ `HandleLidSwitch=suspend`
> - การตั้งค่าเป็น `ignore` จะสั่งให้ระบบไม่ต้องทำอะไรเมื่อฝาพับปิดลง ทำให้ Server ยังคงทำงานต่อไปได้ตามปกติ

สั่งรีสตาร์ตเซอร์วิสเพื่อปรับใช้การตั้งค่า:

```bash
sudo systemctl restart systemd-logind.service
```

> [!NOTE]
> หากเปิดใช้งานบน Desktop Environment การรีสตาร์ตเซอร์วิสนี้อาจทำให้หน้าจอ GUI ล็อกเอาต์หรือรีโหลดได้ แนะนำให้ทำผ่าน SSH หรือบันทึกงานไว้ก่อน

---

### 1.2 พักหน้าจออัตโนมัติเพื่อถนอมจอภาพ (Console Blanking)

สำหรับเครื่องที่เปิด TTY/Console ทิ้งไว้ สามารถตั้งเวลาให้หน้าจอดับลงอัตโนมัติเมื่อไม่มีการใช้งาน เพื่อประหยัดพลังงานและป้องกันปัญหาจอเบิร์น (Screen Burn-in):

แก้ไขการตั้งค่า GRUB:

```bash
sudo nano /etc/default/grub
```

แก้ไขหรือเพิ่มพารามิเตอร์ `consoleblank=300` (ตัวเลข 300 คือ 300 วินาที หรือ 5 นาที):

```bash
GRUB_CMDLINE_LINUX="consoleblank=300"
```

> **คำแนะนำ:** หากมีค่าเดิมอยู่แล้ว สามารถเว้นวรรคแล้วใส่ต่อท้ายได้ เช่น:
> ```bash
> GRUB_CMDLINE_LINUX_DEFAULT="quiet splash consoleblank=300"
> ```

จากนั้นสั่งอัปเดต GRUB และรีบูตเครื่อง:

```bash
sudo update-grub
sudo reboot
```

---

### 1.3 ปิดระบบ Sleep / Suspend โดยสิ้นเชิง (ทางเลือกเสริม)

เพื่อป้องกันไม่ให้ Ubuntu ตัดเข้าโหมด Sleep ไม่ว่าจะจากเหตุผลใด สามารถ Mask เป้าหมายของ systemd ได้โดยตรง:

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

> หากต้องการเปิดกลับมาใช้งานในภายหลัง ให้ใช้คำสั่ง:
> ```bash
> sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
> ```

---

### 1.4 ปิดโหมดประหยัดพลังงานของ Wi-Fi

หากโน้ตบุ๊กเชื่อมต่อผ่าน Wi-Fi บางครั้งตัวการ์ด Wi-Fi จะตัดเข้าสู่โหมด Power Saving ส่งผลให้ SSH หลุดหรือไม่สามารถเชื่อมต่อเข้ามาได้ แนะนำให้ปิดโหมดประหยัดพลังงาน:

แก้ไขไฟล์คอนฟิกของ NetworkManager:

```bash
sudo nano /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
```

เปลี่ยนค่า `wifi.powersave` จาก `3` (เปิดใช้งาน) เป็น `2` (ปิดใช้งาน):

```ini
[connection]
wifi.powersave = 2
```

รีสตาร์ต NetworkManager:

```bash
sudo systemctl restart NetworkManager
```

---

## 2. การตั้งค่าการเชื่อมต่อระยะไกล (SSH Setup)

### 2.1 ติดตั้งและเปิดใช้งาน OpenSSH Server

ติดตั้งแพ็กเกจ SSH และเปิดการทำงานอัตโนมัติตอนบูตเครื่อง:

```bash
sudo apt update
sudo apt install -y openssh-server

# ตรวจสอบสถานะการทำงาน
sudo systemctl status ssh

# เปิดให้เซอร์วิสเริ่มทำงานอัตโนมัติทุกครั้งที่เปิดเครื่อง
sudo systemctl enable --now ssh
```

---

### 2.2 ตรวจสอบ IP Address และทดสอบการเชื่อมต่อ

ตรวจสอบหมายเลข IP ของโน้ตบุ๊ก:

```bash
hostname -I
# หรือ
ip a
```

จากนั้นทดสอบเชื่อมต่อจากเครื่องคอมพิวเตอร์เครื่องอื่นในวงแลนเดียวกัน:

```bash
ssh <ชื่อผู้ใช้>@<IP_ADDRESS_ของเซิร์ฟเวอร์>
```

---

### 2.3 การตั้งค่า Firewall (UFW)

อนุญาตให้พอร์ต SSH สามารถเชื่อมต่อผ่าน Firewall ได้:

```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

---

## 3. การตั้งค่าสำหรับ Ubuntu Desktop & Development

### 3.1 อัปเดตระบบ

ควรอัปเดตคลังแพ็กเกจและซอฟต์แวร์ของระบบให้เป็นปัจจุบันก่อนติดตั้งโปรแกรมอื่นๆ:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 3.2 ติดตั้ง Timeshift สำหรับสำรองข้อมูล

Timeshift เป็นโปรแกรมสำหรับสร้าง System Restore Points ช่วยให้ย้อนสถานะระบบกลับมาได้หากเกิดข้อผิดพลาด:

```bash
sudo apt install -y timeshift
```

---

### 3.3 ติดตั้งเครื่องมือและชุดพัฒนา (Development Tools)

ติดตั้งเครื่องมือพื้นฐานสำหรับงานพัฒนาและคอมไพล์โปรแกรม (C/C++, Go, Build tools):

```bash
sudo apt install -y \
  git \
  wget \
  curl \
  build-essential \
  make \
  cmake \
  gcc \
  clang \
  golang \
  fzf \
  libdrm-dev \
  libgtk-3-dev
```

---

### 3.4 แนะนำเครื่องมือมอนิเตอร์และบริหารระบบ

เครื่องมืออำนวยความสะดวกสำหรับเซิร์ฟเวอร์ที่ควรมีติดเครื่อง:

```bash
sudo apt install -y htop btop tmux net-tools
```

- **btop / htop**: ใช้ดูการใช้งาน CPU, RAM, Disk และอุณหภูมิของแล็ปท็อปแบบเรียลไทม์
- **tmux**: รันเซสชันค้างไว้ใน background เพื่อให้สคริปต์/โปรแกรมทำงานต่อเนื่องแม้ปิดการเชื่อมต่อ SSH

---

## 4. เอกสารที่เกี่ยวข้อง

- [Setup Nerd Fonts in Chrome OS terminal](chromebooksetup.md) - คู่มือตั้งค่าฟอนต์ Nerd Fonts สำหรับ Crostini Terminal บน Chrome OS
