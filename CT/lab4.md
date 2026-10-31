# Лабораторна робота №4
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Створення віртуальних машин у середовищі AWS
**Мета:** Знайомство з популярним хмарним провайдером AWS, створення віртуальних машин у середовищі AWS.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

### Відповіді на питання (Answers to Questions):

#### 1.1. What are AMIs used for? What is the difference between private and public AMIs?
- **Purpose:** An **Amazon Machine Image (AMI)** is a template for launching an EC2 instance. It contains the operating system, preinstalled software and settings, and the disk configuration. One AMI can be used to launch many identical instances.
- **Private AMI:** Visible and usable only by the account that owns it (or by accounts it was explicitly shared with). Used for company images with custom software.
- **Public AMI:** Available to all AWS accounts. Created by AWS (Amazon Linux, Windows Server), by vendors or by the community. It is easy to start with, but community images should be checked before use.

#### 1.2. What is EBS used for? Which EBS types are supported in AWS?
- **Purpose:** **Amazon EBS (Elastic Block Store)** is block storage for EC2, like a virtual hard disk. Data stays on the volume even when the instance is stopped. A volume lives in one Availability Zone and is attached to an instance in the same AZ.
- **Types:**
  - **General Purpose SSD:** `gp3`, `gp2` – balanced price and speed, used for most tasks.
  - **Provisioned IOPS SSD:** `io2`, `io1` – high speed for big databases.
  - **Throughput Optimized HDD:** `st1` – big sequential data (logs, big data).
  - **Cold HDD:** `sc1` – rarely used data, lowest price.
  - **Magnetic:** `standard` – old generation.

#### 1.3. What are Snapshots used for?
A **Snapshot** is a backup of an EBS volume at a certain moment, stored in Amazon S3. The first snapshot is full, the next ones are incremental (only changed blocks). Snapshots are used to restore data, create a new volume (also in another AZ or Region), copy data between Regions and create an AMI.

#### 1.4. Main instance types in AWS and when to use each of them
- **General Purpose** (`T`, `M`): balanced CPU, memory and network. For web servers, small databases, test environments.
- **Compute Optimized** (`C`): powerful CPU. For game servers, video encoding, scientific calculations.
- **Memory Optimized** (`R`, `X`): a lot of RAM. For in-memory databases (Redis), big data analytics.
- **Storage Optimized** (`I`, `D`): fast local disks with high I/O. For NoSQL databases, data warehouses.
- **Accelerated Computing** (`P`, `G`): GPU. For machine learning, graphics, rendering.

#### 1.5. Operating systems in AMI templates of the AWS Free Tier (Windows / Linux / Mac)
In the console, the AMIs with the label **Free tier eligible** are free to use (with a free instance type such as `t2.micro`).

| Family | Example | Hardware and software configuration |
|---|---|---|
| **Linux** | Amazon Linux 2023 | `t2.micro`: 1 vCPU, 1 GiB RAM, 8 GiB `gp3` root disk. Linux optimized for AWS, `dnf` package manager, `systemd`, user `ec2-user`. Free tier eligible. |
| **Windows** | Windows Server 2025 Base | `t2.micro`: 1 vCPU, 1 GiB RAM, 30 GiB root disk. Windows Server with Desktop Experience, access via RDP, user `Administrator`. Free tier eligible. |
| **Mac** | macOS (EC2 Mac) | Runs on real Apple Mac mini hardware (`mac2.metal`, Apple M1, 16 GiB RAM). It is **not** in the Free Tier: it works only on a Dedicated Host with a minimum of 24 hours. |

---

## 2. Хід роботи (Work Progress)

### Практичне завдання №1. Віртуальна машина Linux

### 2.1. Створення Linux-інстансу
Відкрив AWS Management Console → **EC2** → **Launch instance** і задав параметри:
1. **Name:** `Linux_Козуб_Макар`.
2. **AMI:** Amazon Linux 2023 (*Free tier eligible*).
3. **Instance type:** `t2.micro`.
4. **Key pair:** **Create new key pair** → ім'я `linux-key`, тип RSA, формат `.pem`. Файл ключа зберіг на комп'ютері.
5. **Network settings:** залишив за замовчуванням, у Security Group дозволив **SSH, HTTP та HTTPS**.
6. **Storage:** `8 GiB` (gp3).
7. **Advanced details → User data:** додав команди для встановлення Nginx:

```bash
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable nginx
systemctl start nginx
```

8. Натиснув **Launch instance** і відкрив список інстансів. Через хвилину стан став `Running`, а *Status check* – `2/2 checks passed`.
9. Скопіював **Public IPv4 address** і відкрив її в браузері за адресою `http://<Public IP>`. З'явилася сторінка **Welcome to nginx!**, тобто вебсервер працює.

> **Опис скріншота 1 (Параметри запуску інстансу):**
> На скріншоті показано форму *Launch an instance*: ім'я `Linux_Козуб_Макар`, AMI Amazon Linux 2023, тип `t2.micro`, ключ `linux-key`, дозволені SSH / HTTP / HTTPS, диск 8 GiB.

> **Опис скріншота 2 (User data):**
> На скріншоті показано розділ *Advanced details* з полем *User data*, де вписано команди встановлення Nginx.

> **Опис скріншота 3 (Стан інстансу):**
> На скріншоті показано список інстансів EC2. Інстанс `Linux_Козуб_Макар` має стан `Running`, *Status check* – `2/2 checks passed`, праворуч видно Public IPv4 address. Особисті дані приховано.

> **Опис скріншота 4 (Nginx у браузері):**
> На скріншоті показано браузер з адресою `http://<Public IP>` і стандартну сторінку **Welcome to nginx!**.

---

### 2.2. Перегляд інстансів у AWS CLI та підключення через SSH
У терміналі перевірив, чи є запущені віртуальні машини:

```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]" --output table
```

У відповіді є один інстанс зі станом `running` та його публічною IP-адресою.

Далі підключився до інстансу через SSH з використанням ключа:

```bash
ssh -i "linux-key.pem" ec2-user@<Public IP>
```

При першому підключенні підтвердив довіру до хоста (`yes`). З'явилося вітання Amazon Linux, тобто підключення успішне.

> **Опис скріншота 5 (AWS CLI describe-instances):**
> На скріншоті показано результат команди `aws ec2 describe-instances` у вигляді таблиці. Видно інстанс зі станом `running` та його Public IP.

> **Опис скріншота 6 (SSH-підключення):**
> На скріншоті показано термінал після команди `ssh -i "linux-key.pem" ec2-user@<Public IP>`. Рядок запрошення змінився на `[ec2-user@ip-172-31-xx-xx ~]$`, що підтверджує вхід на сервер.

---

### 2.3. Зміна сторінки Nginx (інформація про команду)
Через SSH-з'єднання познайомився з файлами Nginx: головний конфігураційний файл – `/etc/nginx/nginx.conf`, а стартова сторінка – `/usr/share/nginx/html/index.html`. Щоб на сторінці було видно інформацію про команду, відкрив її у редакторі:

```bash
sudo nano /usr/share/nginx/html/index.html
```

Замінив заголовок `Welcome to nginx!` на текст: **Група РПЗ-34, Козуб Макар**. Зберіг файл (`Ctrl+O`, `Enter`, `Ctrl+X`) і перевірив конфігурацію та перезапустив Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Оновив сторінку в браузері. Замість стандартного заголовка тепер відображається інформація про команду.

> **Опис скріншота 7 (Редагування файлу):**
> На скріншоті показано редактор `nano` з відкритим файлом `index.html`, де заголовок замінено на «Група РПЗ-34, Козуб Макар».

> **Опис скріншота 8 (Результат у браузері):**
> На скріншоті показано сторінку за IP-адресою інстансу. Замість `Welcome to nginx!` відображається «Група РПЗ-34, Козуб Макар».

---

### 2.4. Завершення роботи з інстансом
Щоб не було зайвих витрат, у списку EC2 виділив інстанс і вибрав **Instance state → Terminate (delete) instance**, підтвердив дію. Стан змінився на `shutting-down`, а потім на `terminated`.

> **Опис скріншота 9 (Terminate instance):**
> На скріншоті показано список інстансів, де інстанс `Linux_Козуб_Макар` має стан `Terminated`.

---

### Практичне завдання №2. Віртуальна машина Windows

### 2.5. Створення Windows-інстансу
У **EC2** → **Launch instance** задав параметри:
1. **Name:** `Windows_Козуб_Макар`.
2. **AMI:** Windows Server 2025 Base (*Free tier eligible*).
3. **Instance type:** `t2.micro`.
4. **Key pair:** **Create new key pair** → ім'я `windows-key`, тип RSA, формат `.pem` (ключ потрібен, щоб отримати пароль Administrator).
5. **Network settings:** у Security Group дозволив **RDP (порт 3389)**.
6. Натиснув **Launch instance** і дочекався стану `Running` та `2/2 checks passed`.

> **Опис скріншота 10 (Параметри запуску Windows-інстансу):**
> На скріншоті показано форму *Launch an instance*: ім'я `Windows_Козуб_Макар`, AMI Windows Server 2025 Base, тип `t2.micro`, ключ `windows-key`, дозволений RDP.

> **Опис скріншота 11 (Стан Windows-інстансу):**
> На скріншоті показано список інстансів EC2, де інстанс `Windows_Козуб_Макар` має стан `Running` та `2/2 checks passed`.

---

### 2.6. Підключення через Remote Desktop та зміна імені сервера
Перевірив запущені машини командою:

```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name]" --output table
```

Для підключення через RDP:
1. Вибрав інстанс → **Connect** → вкладка **RDP client** → **Get password**.
2. Завантажив файл `windows-key.pem`, натиснув **Decrypt password** і скопіював пароль користувача `Administrator`.
3. Завантажив RDP-файл (**Download remote desktop file**), відкрив його в **Remote Desktop Connection** і ввів пароль.

Після входу відкрив **Settings → System → About**. Там видно інформацію про систему (ім'я пристрою, процесор Intel Xeon, ~1 GB RAM, 64-bit Windows). Натиснув **Rename this PC** і задав нове ім'я **`Windows-Kozub`** (для імені комп'ютера краще латиниця, до 15 символів). Перезавантажив віртуальну машину, знову підключився через RDP і перевірив: ім'я змінилося на `Windows-Kozub`.

> **Опис скріншота 12 (Отримання пароля):**
> На скріншоті показано вікно *Connect to instance* → *RDP client* з кнопкою *Get password* та розшифрованим паролем (замасковано).

> **Опис скріншота 13 (Remote Desktop):**
> На скріншоті показано вікно Windows Server у Remote Desktop Connection та сторінку *System → About* з інформацією про систему.

> **Опис скріншота 14 (Зміна імені сервера):**
> На скріншоті показано діалог *Rename this PC* з введеним ім'ям `Windows-Kozub`.

> **Опис скріншота 15 (Ім'я після перезавантаження):**
> На скріншоті показано сторінку *System → About* після перезавантаження. У полі *Device name* відображається `Windows-Kozub`.

---

### 2.7. Завершення роботи з інстансом
Закрив RDP-сеанс і в EC2 вибрав **Instance state → Terminate (delete) instance** для інстансу `Windows_Козуб_Макар`, підтвердив дію. Стан став `terminated`.

> **Опис скріншота 16 (Terminate instance):**
> На скріншоті показано список інстансів, де інстанс `Windows_Козуб_Макар` має стан `Terminated`.

---

## 3. Контрольні питання (Review Questions)

### 3.1. What role does a Key Pair play when connecting to an EC2 instance?
A **Key Pair** is a pair of keys: the public key is stored on the instance and the private key (`.pem` file) is stored by the user. When connecting, the user proves that he has the private key, so no password is needed. For Linux it is used for SSH login, for Windows it is used to decrypt the `Administrator` password.

### 3.2. What command is used to connect to an EC2 instance via SSH?
```bash
ssh -i "key.pem" ec2-user@<Public IP>
```
`-i` is the path to the private key, `ec2-user` is the default user for Amazon Linux (`ubuntu` for Ubuntu).

### 3.3. How can you connect to a Linux EC2 instance?
- Via **SSH** from a terminal (OpenSSH, PuTTY) with the key pair.
- Via **EC2 Instance Connect** – a browser-based terminal in the AWS Console (button *Connect*).
- Via **AWS Systems Manager Session Manager** – a connection without opening port 22.

### 3.4. How can you connect to a Windows EC2 instance?
Via **RDP (Remote Desktop Protocol)**: in the console choose *Connect → RDP client*, get the password with the `.pem` key, download the RDP file and open it in Remote Desktop Connection. The Security Group must allow port 3389. Also, **AWS Systems Manager Fleet Manager** can be used.

### 3.5. What is the difference between Stop and Terminate for an EC2 instance?
- **Stop:** the instance is shut down, but it still exists. The EBS disk and its data are kept, and the instance can be started again. You do not pay for the instance hours, but you pay for the EBS storage.
- **Terminate:** the instance is deleted permanently and cannot be restored. The root EBS volume is deleted too (by default).

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №4 я ознайомився з сервісом **Amazon EC2** та навчився створювати віртуальні машини в AWS. Було запущено Linux-інстанс на базі Amazon Linux 2023 з автоматичним встановленням вебсервера Nginx через User Data, виконано підключення до нього через SSH та змінено стартову сторінку Nginx. Також створено Windows-інстанс на базі Windows Server 2025, підключено його через Remote Desktop та змінено ім'я сервера.

Я розібрався з поняттями AMI, EBS, Snapshots, типами інстансів, Security Groups та Key Pair. Також дізнався різницю між станами Stop і Terminate, тому після завершення роботи видаляв інстанси, щоб не було зайвих витрат.

**Conclusion (EN):** In this lab I launched a Linux and a Windows virtual machine in AWS EC2, installed Nginx with User Data, connected to the instances via SSH and RDP, and terminated them after the work to avoid extra costs.
