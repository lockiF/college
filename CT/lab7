# Лабораторна робота №7
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Створення віртуальних машин у середовищі Azure
**Мета:** Знайомство з хмарним провайдером Azure, створення віртуальних машин у середовищі Azure.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

#### 1.1. What are Azure Marketplace Images used for? How do they differ from Custom Images?
- **Purpose:** Marketplace Images are ready templates (OS and software) from Microsoft and partners, for example Ubuntu, Windows Server or WordPress. They let you create a VM in a few clicks.
- **Marketplace (public) images:** made by a provider, available to everyone, some paid by license.
- **Custom (private) images:** created by the user from his own VM (with own software and settings), stored in the user's subscription (Azure Compute Gallery), visible only to him or to those he shares with. They are used to create many identical VMs.

#### 1.2. What are Azure Managed Disks used for? Which disk types are supported?
- **Purpose:** Managed Disks are disks for VMs, where Azure itself manages storage, replication and encryption. The user only chooses the size and type.
- **Types:** Standard HDD (cheap, backups), Standard SSD (web servers, tests), Premium SSD (production), Premium SSD v2 and Ultra Disk (highest speed, big databases).

#### 1.3. What are Snapshots used for in Azure?
A Snapshot is a read-only copy of a disk at a certain moment. It is used for backup, restoring a VM, creating a new disk or VM, copying a disk to another region, and saving the state before risky changes such as an update.

#### 1.4. Main VM sizes and when to use them
- **General purpose** (B, D): balanced CPU and RAM. Web servers, small databases, tests.
- **Compute optimized** (F): strong CPU. Game servers, batch processing.
- **Memory optimized** (E, M): a lot of RAM. In-memory and big databases.
- **Storage optimized** (L): fast local disks. NoSQL and data warehouses.
- **GPU** (N): graphics, machine learning.
- **High performance compute** (H): scientific calculations.

#### 1.5. Operating systems available in Azure Free Account
| Family | Example | Description |
|---|---|---|
| **Linux** | Ubuntu Server 24.04 LTS | `B1s`: 1 vCPU, 1 GiB RAM, Standard SSD, `apt`, `systemd`, user `azureuser`, SSH access. |
| **Windows** | Windows Server 2022 Datacenter | `B1s`: 1 vCPU, 1 GiB RAM, 127 GiB disk, access by RDP, Desktop Experience. |
| **Mac** | – | Azure does not offer macOS virtual machines, so this family is not available (macOS in cloud is given by other providers, for example AWS EC2 Mac). |

---

## 2. Хід роботи (Work Progress)

### Практичне завдання №1. Віртуальна машина Linux

### 2.1. Створення Linux-ВМ
Увійшов у `portal.azure.com` → **Virtual machines** → **Create** → **Azure virtual machine** і задав параметри:
1. **Name:** `Linux-Kozub-Makar` (Azure не приймає кирилицю в назві).
2. **Region:** West Europe. **Availability options:** No infrastructure redundancy required.
3. **Image:** Ubuntu Server 24.04 LTS.
4. **Size:** `B1s`.
5. **Authentication type:** SSH public key, username `azureuser`, **Generate new key pair**, ім'я ключа `linux-key`.
6. **Inbound ports:** SSH (22), HTTP (80), HTTPS (443).
7. **Disks:** тип OS disk – Standard SSD.
8. **Advanced → Custom data:**

```bash
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
```

9. Натиснув **Review + create** → **Create**, завантажив файл ключа `linux-key.pem`.
10. Після розгортання відкрив **Go to resource**, скопіював **Public IP address** і відкрив у браузері `http://<Public IP>`. З'явилась сторінка **Welcome to nginx!**.

> **Опис скріншота 1 (Параметри ВМ):** сторінка *Create a virtual machine* з назвою, регіоном, образом Ubuntu, розміром `B1s`, SSH-ключем та відкритими портами 22/80/443.
> **Опис скріншота 2 (Custom data):** вкладка *Advanced* з полем *Custom data* і скриптом встановлення Nginx.
> **Опис скріншота 3 (Стан ВМ):** сторінка *Overview* ВМ зі статусом `Running` та Public IP (частково приховано).
> **Опис скріншота 4 (Nginx у браузері):** браузер з адресою `http://<Public IP>` та стартовою сторінкою **Welcome to nginx!**.

### 2.2. Перевірка ВМ в Azure CLI та підключення через SSH
```bash
az login
az vm list -d -o table
ssh -i linux-key.pem azureuser@<Public-IP>
```
Команда `az vm list -d -o table` показала мою ВМ зі станом `VM running` та Public IP. Після `ssh` підтвердив довіру до хоста (`yes`) і зайшов на сервер: запрошення стало `azureuser@Linux-Kozub-Makar:~$`.

> **Опис скріншота 5 (az vm list):** термінал з таблицею, де видно ВМ `Linux-Kozub-Makar`, `PowerState` – `VM running`.
> **Опис скріншота 6 (SSH):** термінал після `ssh`, де видно вітання Ubuntu та запрошення `azureuser@Linux-Kozub-Makar`.

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
> **Опис скріншота 8 (Результат):** сторінка за Public IP із текстом «Група РПЗ-34, Козуб Макар».

### 2.4. Зупинка та видалення Linux-ВМ
Через Portal: **Virtual machines** → моя ВМ → **Stop** (стан `Stopped (deallocated)`). Через CLI:

```bash
az vm deallocate --name Linux-Kozub-Makar --resource-group <ResourceGroup_Name>
az vm delete --name Linux-Kozub-Makar --resource-group <ResourceGroup_Name> --yes
```

Після видалення також видалив resource group (`az group delete --name <ResourceGroup_Name>`), щоб прибрати диск, IP та мережу.

> **Опис скріншота 9 (Stop/Delete):** вікно Azure Portal зі статусом `Stopped (deallocated)` та термінал з виконаною командою `az vm delete`.

---

### Практичне завдання №2. Віртуальна машина Windows Server

### 2.5. Створення Windows-ВМ
**Virtual machines** → **Create** → **Azure virtual machine**:
1. **Name:** `Windows-Kozub` (для Windows назва ВМ – максимум 15 символів).
2. **Region:** West Europe.
3. **Image:** Windows Server 2022 Datacenter.
4. **Size:** `B1s`.
5. **Authentication:** Username `azureuser`, пароль (запам'ятав).
6. **Inbound ports:** RDP (3389).
7. Інше за замовчуванням → **Review + create** → **Create**. Після розгортання знайшов **Public IP address**.

> **Опис скріншота 10 (Параметри ВМ):** форма *Create a virtual machine* з назвою `Windows-Kozub`, образом Windows Server 2022, розміром `B1s` та відкритим RDP.
> **Опис скріншота 11 (Стан ВМ):** сторінка *Overview* зі статусом `Running` та Public IP.

### 2.6. Підключення через RDP та зміна імені
1. Відкрив **Remote Desktop Connection**, ввів Public IP, увійшов як `azureuser` зі своїм паролем.
2. Відкрив **Settings → System → About** і записав поточне ім'я пристрою (`Windows-Kozub`).
3. Натиснув **Rename this PC**, ввів **`WSERVER-Kozub`**, підтвердив і перезавантажив ВМ.
4. Після перезапуску знову підключився через RDP: у *Device name* стоїть `WSERVER-Kozub`.

> **Опис скріншота 12 (RDP):** вікно Windows Server у Remote Desktop Connection та сторінка *System → About* зі старою назвою.
> **Опис скріншота 13 (Rename):** діалог *Rename this PC* з новою назвою `WSERVER-Kozub`.
> **Опис скріншота 14 (Після перезапуску):** *System → About*, де *Device name* – `WSERVER-Kozub`.

### 2.7. Зупинка та видалення Windows-ВМ
```bash
az vm deallocate --name Windows-Kozub --resource-group <ResourceGroup_Name>
az vm delete --name Windows-Kozub --resource-group <ResourceGroup_Name> --yes
```
`az vm deallocate` зупиняє ВМ і припиняє оплату обчислень, `az vm delete` видаляє її повністю.

> **Опис скріншота 15 (Stop/Delete):** термінал з виконаними командами `az vm deallocate` та `az vm delete` і порожнім результатом `az vm list`.

---

## 3. Контрольні питання (Review Questions)

### 3.1. What role do SSH keys or passwords play when connecting to an Azure Virtual Machine?
They prove the identity of the user. With an SSH key, the public key is stored on the VM and the private key (`.pem`) stays with the user, so no password is sent over the network (safer). A password is used for Windows VMs (RDP), and it can also be used for Linux, but it is less secure.

### 3.2. Which command is used to connect to a Linux VM in Azure via SSH?
```bash
ssh -i linux-key.pem azureuser@<Public-IP>
```

### 3.3. How else can you connect to a Linux VM in Azure?
- **Azure Bastion:** connection from the browser through the portal, without a public IP and open port 22.
- **Serial console:** access when the network does not work.
- **Azure Cloud Shell** or `az ssh vm` command (needs the SSH extension).

### 3.4. How can you connect to a Windows VM in Azure?
Via **RDP**: in the portal choose *Connect → RDP*, download the RDP file (or enter the Public IP in Remote Desktop Connection) and sign in with the username and password. Port 3389 must be open in the NSG. Also **Azure Bastion** can be used.

### 3.5. What is the difference between Stop (Deallocate) and Delete?
- **Stop (Deallocate):** the VM is shut down and its compute resources are released, so you do not pay for the VM hours. The disks and settings remain (storage is still paid), and the VM can be started again. The Public IP can change if it is dynamic.
- **Delete:** the VM is removed permanently. The disk, Public IP and network interface can stay as separate resources if they are not deleted, so the whole resource group should be deleted to avoid costs.

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №7 я навчився створювати віртуальні машини в Azure. Було створено Linux-ВМ на базі Ubuntu Server з автоматичним встановленням Nginx через Custom data, виконано підключення через SSH та змінено стартову сторінку Nginx. Також створено Windows-ВМ на базі Windows Server, підключено її через RDP та змінено ім'я сервера.

Я розібрався з Marketplace Images, Managed Disks, Snapshots, розмірами ВМ, NSG та різницею між Stop (Deallocate) і Delete. Після роботи ВМ зупинялись і видалялись, щоб не було зайвих витрат.

**Conclusion (EN):** In this lab I created a Linux and a Windows virtual machine in Azure, installed Nginx with Custom data, connected via SSH and RDP, and stopped and deleted the machines with Azure CLI to avoid extra costs.
