# Лабораторна робота №9
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Створення віртуальних машин у середовищі Google Cloud Platform
**Мета:** Знайомство з хмарним провайдером Google Cloud Platform (GCP), створення віртуальних машин у середовищі GCP.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

#### 1.1. What is Google Compute Engine (GCE) used for?
Compute Engine is the IaaS service of GCP for creating and running virtual machines on Google's infrastructure. The user chooses the OS, CPU, memory and disks, and gets full control over the VM. It supports predefined and custom machine types, per-second billing, instance groups with autoscaling and Spot VMs.

#### 1.2. What are Machine Images? How do Google's standard images differ from Custom Images?
- **Purpose:** an image is a template of the boot disk (OS and software) used to create a VM.
- **Standard (public) images:** prepared and updated by Google or partners (Debian, Ubuntu, Windows Server, etc.), available to everyone. Some (Windows, RHEL) have a license fee.
- **Custom images:** created by the user from his own disk or snapshot, with own software and settings. They are private to the project and are used to create many identical VMs.

#### 1.3. What are Persistent Disks used for? What types exist?
Persistent Disks are network block disks for VMs. Data stays after the VM is stopped or deleted (if the disk is kept), and the disk can be resized and backed up with snapshots.
- **Standard (`pd-standard`):** HDD, cheapest and slowest, for big sequential data and backups.
- **Balanced (`pd-balanced`):** SSD with balanced price and speed, default for most tasks.
- **SSD (`pd-ssd`):** fast SSD for databases and high I/O.
- **Extreme (`pd-extreme`):** highest speed with set IOPS for heavy databases.

#### 1.4. What are Snapshots in Google Cloud? When should they be used?
A Snapshot is a backup copy of a Persistent Disk. The first one is full, the next ones are incremental. They are used for backups, restoring a disk or VM, creating a new disk in another zone or region, and before risky changes or before deleting a VM.

#### 1.5. Main Machine Types and when to use them
- **General purpose** (`e2`, `n2`): balanced CPU and RAM. `e2` is cheaper, for web servers and tests; `n2` is faster, for business apps.
- **Compute optimized** (`c2`, `c3`; also `n2-highcpu` with 1 GB RAM per vCPU): strong CPU for games, encoding, calculations.
- **Memory optimized** (`m`; also `n2-highmem` with 8 GB RAM per vCPU): in-memory and big databases.
- **Storage optimized** (`z3`): fast local disks.
- **GPU / accelerator** (`a2`, `g2`): machine learning and graphics.

#### 1.6. Operating systems available in the Google Cloud Free Tier
The Free Tier gives one `e2-micro` VM per month (only in the US regions `us-west1`, `us-central1`, `us-east1`) with a 30 GB standard disk.
- **Linux (free):** Debian 12, Ubuntu 22.04 LTS / 24.04 LTS, Rocky Linux.
- **Windows:** Windows Server 2019 / 2022 are available as images, but they have a license fee and are not free (they are paid from the trial credits).

#### 1.7. What are Firewall rules in GCP and how do they affect access to a VM?
Firewall rules are part of the VPC network. They allow or deny incoming (ingress) and outgoing (egress) traffic by protocol, port, IP range and network tag or service account. By default all incoming traffic is denied except the rules created by default (SSH, RDP, ICMP). The checkboxes *Allow HTTP/HTTPS traffic* add a tag to the VM and a rule that opens ports 80 and 443 for it.

#### 1.8. SSH vs RDP: difference and use cases
- **SSH:** secure text terminal (port 22), authentication by keys. Used for Linux servers and administration through the command line.
- **RDP:** remote graphical desktop (port 3389), login and password. Used for Windows servers and work with the graphical interface.

---

## 2. Хід роботи (Work Progress)

### Практичне завдання №1. Віртуальна машина Linux

### 2.1. Створення Linux-ВМ
Увійшов у `console.cloud.google.com` → **Compute Engine** → **VM instances** → **Create instance** і задав параметри:
1. **Name:** `linux-kozub-makar` (GCP приймає лише малі літери, цифри та дефіси).
2. **Region / Zone:** `us-central1` / `us-central1-a` (e2-micro безкоштовна лише в US-регіонах).
3. **Machine type:** `e2-micro`.
4. **Boot disk → Change:** Ubuntu 22.04 LTS.
5. **Firewall:** позначив **Allow HTTP traffic** та **Allow HTTPS traffic**.
6. **Advanced options → Management → Automation → Startup script:**

```bash
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
```

7. Натиснув **Create**, дочекався зеленої позначки біля ВМ, скопіював **External IP** і відкрив у браузері `http://<External IP>`. З'явилась сторінка **Welcome to nginx!**.

> **Опис скріншота 1 (Параметри ВМ):** сторінка *Create an instance* з назвою, регіоном, `e2-micro`, образом Ubuntu 22.04 та позначками Allow HTTP/HTTPS.
> **Опис скріншота 2 (Startup script):** розділ *Management → Automation* зі скриптом встановлення Nginx.
> **Опис скріншота 3 (Список ВМ):** *VM instances* зі створеною ВМ, зеленою позначкою та External IP (частково приховано).
> **Опис скріншота 4 (Nginx у браузері):** сторінка **Welcome to nginx!** за External IP.

### 2.2. Перевірка ВМ у Cloud Shell та підключення по SSH
У Cloud Shell (кнопка `>_`) виконав:

```bash
gcloud compute instances list
gcloud compute ssh linux-kozub-makar --zone=us-central1-a
```

Перша команда показала мою ВМ зі статусом `RUNNING` та External IP. Друга створила SSH-ключ (при першому запуску) і підключила мене до сервера: запрошення змінилось на `<user>@linux-kozub-makar:~$`.

> **Опис скріншота 5 (instances list):** Cloud Shell з таблицею, де видно ВМ `linux-kozub-makar`, зону, тип `e2-micro` та статус `RUNNING`.
> **Опис скріншота 6 (SSH):** Cloud Shell після `gcloud compute ssh` із запрошенням `linux-kozub-makar`.

### 2.3. Зміна сторінки Nginx
Головний конфіг Nginx – `/etc/nginx/sites-available/default`, а стартова сторінка – `/var/www/html/index.nginx-debian.html`. Відкрив сторінку в редакторі:

```bash
sudo nano /var/www/html/index.nginx-debian.html
```

Замінив заголовок `Welcome to nginx!` на **Група РПЗ-34, Козуб Макар**, зберіг (`Ctrl+O`, `Enter`, `Ctrl+X`) і перезапустив Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Оновив сторінку в браузері: замість стандартного заголовка відображається інформація про команду.

> **Опис скріншота 7 (Редагування):** редактор `nano` з відкритим файлом, де заголовок замінено на «Група РПЗ-34, Козуб Макар».
> **Опис скріншота 8 (Результат):** сторінка за External IP із текстом «Група РПЗ-34, Козуб Макар».

### 2.4. Зупинка та видалення Linux-ВМ
```bash
gcloud compute instances stop linux-kozub-makar --zone=us-central1-a
gcloud compute instances delete linux-kozub-makar --zone=us-central1-a --quiet
```
- **Stop:** ВМ вимикається, статус `TERMINATED`, але диск і налаштування зберігаються, її можна запустити знову. Обчислення не оплачуються, а диск оплачується.
- **Delete:** ВМ видаляється назавжди разом із завантажувальним диском (за замовчуванням), `--quiet` пропускає запит на підтвердження.

> **Опис скріншота 9 (Stop/Delete):** Cloud Shell з виконаними командами `stop` та `delete` і порожнім результатом `gcloud compute instances list`.

---

### Практичне завдання №2. Віртуальна машина Windows Server

### 2.5. Створення Windows-ВМ
**Compute Engine** → **VM instances** → **Create instance**:
1. **Name:** `windows-kozub`. **Region / Zone:** `us-central1` / `us-central1-a` (як для Linux).
2. **Machine type:** `e2-micro` (якщо консоль не дозволяє або ВМ працює дуже повільно, вибрав `e2-medium`, бо Windows потребує більше пам'яті).
3. **Boot disk → Change:** Operating system – Windows Server, версія Windows Server 2022 Datacenter.
4. **Firewall:** RDP (3389) відкритий правилом `default-allow-rdp` у мережі `default`.
5. Натиснув **Create**.

> **Опис скріншота 10 (Параметри ВМ):** форма *Create an instance* з назвою `windows-kozub`, типом машини та образом Windows Server 2022.
> **Опис скріншота 11 (Список ВМ):** *VM instances* з Windows-ВМ, зеленою позначкою та External IP.

### 2.6. Підключення через RDP та зміна імені
1. У списку ВМ відкрив стрілку біля **RDP** → **Set Windows password**, ввів ім'я користувача `kozub` і отримав пароль (скопіював).
2. Відкрив **Remote Desktop Connection**, ввів External IP і увійшов з отриманими даними.
3. Відкрив **Settings → System → About** і записав поточне ім'я пристрою (`windows-kozub`).
4. Натиснув **Rename this PC**, ввів **`WSERVER-Kozub`**, підтвердив і перезавантажив ВМ.
5. Після перезапуску знову підключився через RDP: у *Device name* відображається `WSERVER-Kozub`.

> **Опис скріншота 12 (Пароль):** вікно *Reset Windows password* з іменем користувача та згенерованим паролем (замасковано).
> **Опис скріншота 13 (RDP):** вікно Windows Server у Remote Desktop Connection та *System → About* зі старою назвою.
> **Опис скріншота 14 (Rename):** діалог *Rename this PC* з новою назвою `WSERVER-Kozub`.
> **Опис скріншота 15 (Після перезапуску):** *System → About*, де *Device name* – `WSERVER-Kozub`.

### 2.7. Завершення роботи з Windows-ВМ
```bash
gcloud compute instances stop windows-kozub --zone=us-central1-a
gcloud compute instances delete windows-kozub --zone=us-central1-a --quiet
```

> **Опис скріншота 16 (Stop/Delete):** Cloud Shell з виконаними командами та порожнім списком ВМ.

---

## 3. Контрольні питання (Review Questions)

### 3.1. What role do SSH keys and passwords play when connecting to VMs in GCP?
They prove the identity of the user. With SSH keys the public key is stored in the VM or project metadata (or managed by OS Login), and the private key stays with the user, so no password is sent (safer). Passwords are used for Windows VMs (RDP): the password is generated with *Set Windows password*.

### 3.2. Which command connects to a Linux VM through the gcloud CLI?
```bash
gcloud compute ssh <vm-name> --zone=<zone-name>
```

### 3.3. What connection methods are available for a Linux VM in GCP?
- The **SSH** button in the console (browser window).
- `gcloud compute ssh` from Cloud Shell or a local terminal.
- A standard `ssh -i key user@external-ip` with your own key.
- **IAP tunnel** (`--tunnel-through-iap`): connection without an external IP.

### 3.4. How can you connect to a Windows VM in GCP?
Via **RDP**: set the Windows password (*Set Windows password* or `gcloud compute reset-windows-password`), then connect with Remote Desktop Connection to the External IP, or use the RDP button in the console. Port 3389 must be open in the firewall.

### 3.5. What is the difference between Stop and Delete?
- **Stop:** the VM is shut down (status `TERMINATED`), but it still exists with its disk and settings. It can be started again, and you pay only for disks and reserved IPs.
- **Delete:** the VM is removed permanently, and the boot disk is deleted too by default.

### 3.6. What is a Startup script used for?
A Startup script is a script (metadata key `startup-script`) that runs as root every time the VM boots. It automates setup: installing software (for example Nginx), updating the system and configuring services, so no manual work is needed.

### 3.7. How can incoming traffic to a VM be allowed or denied in GCP?
With **VPC firewall rules** (*VPC network → Firewall*) or the checkboxes *Allow HTTP/HTTPS traffic*. A rule sets direction, action (allow/deny), protocol and port, IP range and target tags:
```bash
gcloud compute firewall-rules create allow-http --allow=tcp:80 --target-tags=http-server
```

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №9 я навчився створювати віртуальні машини в Google Compute Engine. Було створено Linux-ВМ на базі Ubuntu з автоматичним встановленням Nginx через Startup script, виконано підключення через `gcloud compute ssh` та змінено стартову сторінку Nginx. Також створено Windows-ВМ на базі Windows Server, підключено її через RDP та змінено ім'я сервера.

Я розібрався з образами, Persistent Disks, Snapshots, типами машин, правилами Firewall та різницею між Stop і Delete. Після роботи ВМ зупинялись і видалялись командами gcloud, щоб не було зайвих витрат.

**Conclusion (EN):** In this lab I created a Linux and a Windows virtual machine in Google Compute Engine, installed Nginx with a startup script, connected via SSH and RDP, and stopped and deleted the machines with gcloud to avoid extra costs.
