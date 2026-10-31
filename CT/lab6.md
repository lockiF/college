# Лабораторна робота №6
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Реєстрація та знайомство з хмарним середовищем Azure
**Мета:** Реєстрація та знайомство з хмарним провайдером Azure, реєстрація акаунту та огляд його інтерфейсу.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

#### 1.1. What is a virtual machine and what are the features of Azure Virtual Machine?
A virtual machine (VM) is a software emulation of a physical server with its own OS, CPU, RAM and disk. Azure Virtual Machine is an IaaS service: you choose an image (Windows, Linux) and a size (series B, D, E, F), and Azure gives managed disks, a virtual network (VNet) and a firewall (NSG). It supports availability zones and scale sets. You pay for running time, and a deallocated VM does not pay for compute.

#### 1.2. What does Azure App Service provide?
Azure App Service is a PaaS service for hosting web apps, REST APIs and mobile backends without managing servers. It supports .NET, Java, Node.js, Python, PHP and containers. It has autoscaling, deployment slots, CI/CD (GitHub, Azure DevOps), custom domains, SSL and built-in monitoring.

#### 1.3. Main steps to create an App Service in Azure Portal
1. Portal → **Create a resource** → **Web App**.
2. Choose Subscription and Resource Group.
3. Enter a unique app name (URL `name.azurewebsites.net`).
4. Choose Publish (Code or Container), Runtime stack and Region.
5. Choose an App Service Plan (pricing tier).
6. **Review + create** → **Create**, then open the app URL.

#### 1.4. Features of "Access control (IAM)" in Azure
IAM in Azure is based on RBAC. A role assignment has three parts: who (user, group, service principal, managed identity), which role (Owner, Contributor, Reader, etc.) and the scope (management group, subscription, resource group or resource). Permissions are inherited from the higher scope to the lower one. In the IAM blade you can add role assignments and check access of a user.

#### 1.5. How are Deployment slots used for version management?
A deployment slot is a separate live copy of an App Service (for example, `staging`) with its own URL. A new version is deployed to the staging slot and tested, then **swapped** with production without downtime. If something goes wrong, a swap back works as a quick rollback. Slots need Standard tier or higher.

#### 1.6. Overview of a Storage Account and its key functions
A Storage Account is a container with a unique name for all Azure Storage data (Blob, File, Table, Queue). Key functions: redundancy options (LRS, ZRS, GRS, GZRS), access tiers (Hot, Cool, Cold, Archive), encryption at rest, access via keys, SAS tokens or Entra ID, and a REST endpoint for every service.

#### 1.7. Types of storage in Azure
- **Blob Storage:** object storage for files, images, video, backups.
- **File Storage:** network file shares (SMB/NFS), like a shared disk.
- **Table Storage:** simple NoSQL store of key-attribute data.
- **Queue Storage:** queue of messages for communication between app parts.

#### 1.8. Working with Azure Storage in C#
Install the NuGet package `Azure.Storage.Blobs` and take the connection string from *Storage account → Access keys*. Create a `BlobServiceClient`, get a container with `GetBlobContainerClient`, create it with `CreateIfNotExistsAsync`, then use `GetBlobClient` with `UploadAsync` and `DownloadToAsync` for files. Queues work in the same way with `QueueClient` (`SendMessageAsync`, `ReceiveMessageAsync`), and tables with `TableClient`.

#### 1.9. What is Serverless and how is it implemented in Azure Functions?
Serverless means that the developer does not manage servers: the provider scales resources automatically and charges only for execution. In Azure Functions, a function runs when a **trigger** fires (HTTP, Timer, Blob, Queue, Service Bus) and **bindings** connect it to other services. Hosting plans: Consumption, Flex Consumption and Premium. Functions can be written in C#, JavaScript, Python, Java, and tested locally in VS Code with Azure Functions Core Tools (`func start`).

#### 1.10. Authentication vs authorization. How are Azure RBAC and Entra ID related?
**Authentication** checks who you are (login, password, MFA). **Authorization** decides what you can do. **Microsoft Entra ID (Azure AD)** does authentication and gives the user a token. **Azure RBAC** does authorization: it checks the roles of this identity on the resource.

---

## 2. Хід роботи (Work Progress)

### 2.1. Реєстрація FREE Azure Account
Відкрив `portal.azure.com` → **Start free** і виконав кроки:
1. Увійшов у Microsoft-акаунт (або створив новий).
2. Заповнив профіль: країна, ім'я, email, телефон.
3. Підтвердив номер телефону кодом з SMS.
4. Ввів дані банківської картки (для перевірки особи, гроші не списуються).
5. Погодився з умовами й натиснув **Sign up**.

Після створення отримав кредити $200 на 30 днів та безкоштовні ресурси, і зайшов у Azure Portal.

> **Опис скріншота 1 (Реєстрація):**
> На скріншоті показано форму *Sign up* з полями профілю та кроком перевірки телефону. Особисті дані заблюрено.

> **Опис скріншота 2 (Azure Portal):**
> На скріншоті показано головну сторінку Azure Portal: пошук зверху, меню зліва, розділ *Azure services* та *Recent resources*.

### 2.2. Налаштування MFA
1. Відкрив `mysignins.microsoft.com/security-info` → **Add sign-in method**.
2. Вибрав **Authenticator app** → встановив Microsoft Authenticator на телефон.
3. Натиснув **Next**, відсканував QR-код у застосунку.
4. Підтвердив вхід тестовим повідомленням.

Для перевірки вийшов з акаунту й зайшов знову: після пароля прийшов запит на підтвердження в Authenticator.

> **Опис скріншота 3 (MFA):**
> На скріншоті показано сторінку *Security info*, де в списку методів входу є *Microsoft Authenticator*.

### 2.3. Бюджет «Zero Spend Budget»
1. У пошуку Portal відкрив **Cost Management + Billing** → **Budgets** → **Add**.
2. Scope: моя підписка. Name: `Zero Spend Budget`.
3. Reset period: Monthly. Amount: `$0` (якщо портал не приймає 0, вказав мінімальне значення, напр. $1).
4. В *Alert conditions* додав оповіщення: 100% від бюджету (Actual), у *Alert recipients* вказав свій email.
5. Натиснув **Create**.

> **Опис скріншота 4 (Budget):**
> На скріншоті показано сторінку *Budgets* із бюджетом `Zero Spend Budget`, його сумою та оповіщенням на email.

### 2.4. Група developers та користувачі
1. Portal → **Microsoft Entra ID** → **Users** → **New user** → **Create new user**. Створив трьох користувачів: мене та двох одногрупників.
2. **Groups** → **New group**: тип *Security*, назва `developers`.
3. У **Members** додав трьох створених користувачів → **Create**.

> **Опис скріншота 5 (Group developers):**
> На скріншоті показано групу `developers` зі списком із трьох учасників у вкладці *Members*.

### 2.5. Група readonly
1. **Groups** → **New group** → тип *Security*, назва `readonly`.
2. Додав до неї одного користувача з `developers` (себе).
3. Відкрив **Users** → мій користувач → **Groups**, щоб побачити членство.

> **Опис скріншота 6 (Groups користувача):**
> На скріншоті показано вкладку *Groups* користувача, де відображено дві групи: `developers` та `readonly`.

### 2.6. Встановлення Azure CLI
Завантажив MSI-інсталятор Azure CLI з офіційного сайту Microsoft, пройшов кроки встановлення (Next → Install → Finish), перезапустив термінал і перевірив версію:

```bash
az --version
```

> **Опис скріншота 7 (az --version):**
> На скріншоті показано PowerShell з результатом команди `az --version`: версія `azure-cli` та Python.

### 2.7. Вхід через Azure CLI
```bash
az login
```
Відкрилось вікно браузера, де я увійшов у свій акаунт Azure. У терміналі з'явився JSON з інформацією про підписку.

> **Опис скріншота 8 (az login):**
> На скріншоті показано термінал після `az login` з JSON-даними підписки (ID приховано) та вікно браузера з повідомленням про успішний вхід.

### 2.8. Перевірка віртуальних машин
```bash
az vm list --output table
```
Команда повернула порожній результат, тобто запущених віртуальних машин немає.

> **Опис скріншота 9 (az vm list):**
> На скріншоті показано виконання `az vm list --output table`. Результат порожній.

### 2.9. Довідник з використання Azure CLI (Cheat Sheet)

#### 1. Структура команд
```bash
az <група> <підгрупа> <команда> [параметри]
```

#### 2. Довідка
- `az --help` – загальна довідка
- `az vm --help` – довідка по групі
- `az find "vm create"` – пошук команд

#### 3. Базові команди
```bash
az login                          # вхід в акаунт
az logout                         # вихід
az account show                   # поточна підписка
az account list --output table    # список підписок
az group create --name LabGroup --location westeurope   # створити resource group
az group list --output table      # список resource groups
az group delete --name LabGroup   # видалити resource group
az vm list --output table         # список віртуальних машин
az vm start --name VM1 --resource-group LabGroup    # запустити ВМ
az vm deallocate --name VM1 --resource-group LabGroup   # зупинити ВМ
az storage account list --output table   # список storage account
az ad user list --output table    # список користувачів Entra ID
az ad group list --output table   # список груп
```

#### 4. Корисні параметри
- `--output json | table | tsv` – формат виводу
- `--query` – фільтрація результату (JMESPath)
- `--subscription` – вибір підписки
- `--resource-group` (`-g`) – назва групи ресурсів

---

## 3. Контрольні питання (Review Questions)

### 3.1. What is the essence of TOTP authentication? Examples with Microsoft Authenticator and Azure MFA.
**TOTP (Time-based One-Time Password)** is a one-time password calculated from a shared secret key and the current time. When MFA is set up, Azure shows a QR code and the app saves the secret. Every 30 seconds the app calculates a new 6-digit code, and the server calculates the same code and compares it. **Microsoft Authenticator** can show TOTP codes and also sends push notifications for approval. In **Azure MFA** the user enters the password first and then the code or approves the push, so a stolen password alone is not enough.

### 3.2. Describe the JSON format: where it is used and its features. Is it used in Azure?
**JSON (JavaScript Object Notation)** is a lightweight text format for data exchange. It is built from key-value pairs `{ "key": "value" }` and arrays `[1, 2, 3]`. Types: string, number, boolean, null, object, array. Keys use double quotes, comments are not allowed. It is used in REST APIs, config files and NoSQL databases. In Azure it is used a lot: ARM templates, Azure CLI output, REST API requests, Azure Policy, Cosmos DB documents and Azure Functions configuration (`host.json`).

### 3.3. What types of Azure API exist?
1. **Azure Resource Manager (ARM) REST API:** management of resources (create VM, group, storage). Portal, CLI and PowerShell use it.
2. **Service (data plane) REST APIs:** work with the data inside services, for example Blob Storage or Cosmos DB.
3. **Microsoft Graph API:** access to Entra ID (users, groups) and Microsoft 365 data.
4. **Azure API Management:** a service for publishing and protecting your own APIs.

All of them use tokens from Entra ID (OAuth 2.0).

### 3.4. How can Azure CLI be installed on different operating systems?
- **Windows:** MSI installer, or `winget install Microsoft.AzureCLI`.
- **macOS:** `brew install azure-cli`.
- **Linux (Debian/Ubuntu):** `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash`.
- **Any OS:** Docker image or Azure Cloud Shell in the browser (no installation).

Check the installation with `az --version`.

### 3.5. Basic Azure CLI commands with examples
- `az login` – sign in to the account.
- `az account show` – show the current subscription.
- `az group create --name LabGroup --location westeurope` – create a resource group.
- `az vm list --output table` – list virtual machines.
- `az storage account list` – list storage accounts.
- `az ad user list` – list Entra ID users.

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №6 я ознайомився з хмарною платформою Microsoft Azure. Було зареєстровано акаунт Azure Free, налаштовано багатофакторну автентифікацію через Microsoft Authenticator та створено бюджет Zero Spend Budget у Cost Management для контролю витрат.

Також я попрацював з Microsoft Entra ID: створив користувачів і групи `developers` та `readonly`. Встановив і налаштував Azure CLI, перевірив за його допомогою наявність віртуальних машин та склав коротку довідку з базових команд. Окремо розібрався з основними сервісами Azure: Virtual Machines, App Service, Storage Account, Azure Functions та RBAC.

**Conclusion (EN):** In this lab I created and secured an Azure account (MFA, Budgets), managed users and groups in Microsoft Entra ID, and used Azure CLI to manage resources from the command line.
