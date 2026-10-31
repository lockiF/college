# Лабораторна робота №8
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Реєстрація та знайомство з хмарним середовищем Google Cloud Platform
**Мета:** Реєстрація та знайомство з хмарним провайдером Google Cloud Platform (GCP), реєстрація акаунту та огляд його інтерфейсу.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

#### 1.1. What is a virtual machine and what are the features of Google Compute Engine?
A virtual machine is a software emulation of a physical server with its own OS, CPU, RAM and disk. Compute Engine is the IaaS service of GCP. Its features: predefined and custom machine types (E2, N2, C2, etc.), Persistent Disks, per-second billing, Spot VMs (cheap but can be stopped), live migration (VM keeps running during host maintenance), instance groups with autoscaling and a global network of Google.

#### 1.2. What does Google App Engine provide for deploying web applications?
App Engine is a PaaS service: the developer uploads code, and Google manages servers, scaling and updates. It supports Python, Java, Node.js, Go, PHP and Ruby. It has automatic scaling (also to zero in the Standard environment), versions, traffic splitting, built-in security and integration with other GCP services. There are two environments: Standard and Flexible (Docker containers).

#### 1.3. Main steps to create an App Engine application in Google Cloud Console
1. Create or select a **project** (with billing enabled).
2. Open **App Engine** → **Create application**.
3. Choose the **region** (it cannot be changed later).
4. Choose the language and environment (Standard or Flexible).
5. Write the app and `app.yaml`, then deploy it with `gcloud app deploy`.
6. Open the app with `gcloud app browse`.

#### 1.4. Features of Identity and Access Management (IAM) in GCP
IAM answers the question "who can do what on which resource". A **principal** (user, group, service account) gets a **role** (a set of permissions) on a resource. Role types: **Basic** (Owner, Editor, Viewer), **Predefined** (made by Google for a service) and **Custom**. Policies are inherited from top to bottom: Organization → Folder → Project → Resource. It is recommended to give roles to groups and follow the principle of least privilege.

#### 1.5. Using Versions in App Engine to manage updates
Every deployment creates a new **version** of the service, and old versions are kept. Several versions can work at the same time, and traffic can be split between them (for example 10% to the new one for testing, a canary release). If the new version has errors, traffic is moved back to the old version (quick rollback), and unused versions can be deleted.

#### 1.6. Overview of Cloud Storage and its key functions
Cloud Storage is an object storage for files, backups, media and data lakes. Data is stored in **buckets** (globally unique names) as objects. Key functions: location types (region, dual-region, multi-region), storage classes, object versioning, lifecycle rules, encryption by default, access control (IAM, signed URLs), high durability (99.999999999%) and a web/API access to any object.

#### 1.7. Storage classes in GCP
- **Standard:** frequently used data (websites, active apps), no minimum storage time.
- **Nearline:** data accessed about once a month, minimum 30 days.
- **Coldline:** data accessed about once a quarter, minimum 90 days.
- **Archive:** the cheapest, for data accessed less than once a year (archives, legal data), minimum 365 days.

All classes have the same speed of access (milliseconds); the colder the class, the cheaper the storage and the more expensive the access.

#### 1.8. Working with Cloud Storage in Python
Install the library `pip install google-cloud-storage` and log in with `gcloud auth application-default login`. Then create `storage.Client()`, get a bucket with `client.bucket("name")`, create an object with `bucket.blob("file.txt")` and use `blob.upload_from_filename()` to upload and `blob.download_to_filename()` to download. `client.list_blobs("name")` shows the objects in the bucket.

#### 1.9. What is Serverless and how is it implemented in Cloud Functions and Cloud Run?
Serverless means the developer does not manage servers: Google scales resources automatically and charges only for use. **Cloud Functions** (now called Cloud Run functions) runs a small function after an event (HTTP request, file in Cloud Storage, Pub/Sub message). **Cloud Run** runs any container (any language) as a web service or job, scales from zero to many instances and is used for bigger applications.

#### 1.10. Authentication vs authorization. How are IAM Roles and Service Accounts related?
**Authentication** checks who you are (login, password, 2-step verification). **Authorization** decides what you can do. In GCP, authorization is made by **IAM roles**: they give permissions to an identity. A **Service Account** is an identity for an application or VM: it is authenticated with a token or key and gets IAM roles, so the program can access GCP services without a human.

---

## 2. Хід роботи (Work Progress)

### 2.1. Реєстрація Free Trial у GCP
Відкрив `cloud.google.com` → **Get started for free** і виконав кроки:
1. Увійшов у Google-акаунт.
2. Вибрав країну та погодився з умовами сервісу.
3. Вказав тип акаунту **Individual**, ім'я та адресу.
4. Підтвердив номер телефону та ввів дані банківської картки (для перевірки, автоматичного списання після пробного періоду немає).
5. Натиснув **Start free** і отримав $300 кредитів на 90 днів.

> **Опис скріншота 1 (Реєстрація):** форма *Get started for free* з кроками реєстрації. Особисті дані заблюрено.
> **Опис скріншота 2 (Результат):** вікно з повідомленням про активацію Free Trial та кредити $300.

### 2.2. Огляд Google Cloud Console
Відкрив `console.cloud.google.com`. Основні розділи інтерфейсу:
- **Project selector** (зверху ліворуч): вибір проєкту.
- **Search**: пошук сервісів і ресурсів.
- **Navigation menu** (☰): усі сервіси (Compute Engine, Cloud Storage, IAM & Admin, Billing).
- **Dashboard**: інформація про проєкт, використання ресурсів та білінг.
- **Cloud Shell** (кнопка `>_`): термінал у браузері.
- **Billing**: витрати, кредити, бюджети.

> **Опис скріншота 3 (Dashboard):** головна панель керування з вибором проєкту, пошуком і картками *Project info*, *Resources*, *Billing*.
> **Опис скріншота 4 (Меню навігації):** відкрите меню з розділами сервісів.
> **Опис скріншота 5 (Billing):** сторінка *Billing Overview* із кредитами $300.

### 2.3. Двофакторна автентифікація (2-Step Verification)
1. Відкрив `myaccount.google.com` → **Security** → **2-Step Verification** → **Get started**.
2. Підтвердив вхід паролем.
3. Вибрав **Authenticator app** → **Set up authenticator**.
4. Відсканував QR-код у Google Authenticator.
5. Ввів 6-значний код і натиснув **Verify** → **Turn on**.

Для перевірки вийшов з акаунту й зайшов знову: після пароля система запитала код.

> **Опис скріншота 6 (2-Step Verification):** сторінка *2-Step Verification* зі статусом **On** та методом Authenticator app.

### 2.4. Бюджет «Zero Spend Budget»
1. У консолі відкрив **Billing** → **Budgets & alerts** → **Create budget**.
2. Name: `Zero Spend Budget`, вибрав свій проєкт.
3. Amount: *Specified amount*, `$0` (якщо консоль не приймає 0, вказав мінімальне значення, напр. $1).
4. Додав пороги сповіщень (100% Actual) та увімкнув надсилання на email адміністраторів білінгу.
5. Натиснув **Finish**.

> **Опис скріншота 7 (Budget):** сторінка *Budgets & alerts* із бюджетом `Zero Spend Budget` та сповіщенням на email.

### 2.5. Група developers
1. Відкрив **IAM & Admin → Groups** (група створюється через Google Groups / Cloud Identity).
2. **Create group**, назва `developers`.
3. Додав у групу трьох користувачів: мене та двох одногрупників.
4. У **IAM → Grant access** можна видати групі роль (наприклад, *Viewer*) на проєкт.

> **Опис скріншота 8 (Group developers):** сторінка групи `developers` зі списком із трьох учасників.

### 2.6. Група readonly
1. **IAM & Admin → Groups → Create group**, назва `readonly`.
2. Додав до неї одного користувача з `developers` (себе).
3. Відкрив сторінку користувача й подивився в розділі **Groups**, де він перебуває.

> **Опис скріншота 9 (Groups користувача):** список груп користувача: `developers` та `readonly`.

### 2.7. Встановлення Google Cloud SDK
Завантажив інсталятор з `cloud.google.com/sdk/docs/install`, пройшов кроки (Next → Install → Finish) і перезапустив термінал.

```bash
gcloud --version
```

> **Опис скріншота 10 (gcloud --version):** PowerShell з версіями `Google Cloud SDK`, `bq`, `core`, `gsutil`.

### 2.8. Авторизація в gcloud
```bash
gcloud auth login
```
Відкрилась сторінка браузера, де я вибрав свій Google-акаунт і дозволив доступ. У терміналі з'явилось повідомлення `You are now logged in as [...]`.

> **Опис скріншота 11 (gcloud auth login):** вікно браузера із запитом доступу та термінал із повідомленням про успішний вхід (email приховано).

### 2.9. Перевірка віртуальних машин
```bash
gcloud compute instances list
```
Команда повернула `Listed 0 items.`, тобто запущених віртуальних машин немає.

> **Опис скріншота 12 (instances list):** термінал з виконаною командою та результатом `Listed 0 items.`

### 2.10. Довідник з використання gcloud (Cheat Sheet)

#### 1. Структура команд
```bash
gcloud <група> <підгрупа> <команда> [параметри]
```

#### 2. Довідка
- `gcloud help` – загальна довідка
- `gcloud compute --help` – довідка по групі
- `gcloud compute instances create --help` – довідка по команді

#### 3. Базові команди
```bash
gcloud auth login                       # авторизація
gcloud auth list                        # список акаунтів
gcloud config list                      # поточна конфігурація
gcloud projects create my-lab-project   # створити проєкт
gcloud projects list                    # список проєктів
gcloud config set project <PROJECT_ID>  # вибрати проєкт
gcloud compute instances list           # список ВМ
gcloud compute instances create my-vm --zone=europe-west1-b   # створити ВМ
gcloud compute instances stop my-vm --zone=europe-west1-b     # зупинити ВМ
gcloud compute instances delete my-vm --zone=europe-west1-b   # видалити ВМ
gcloud storage buckets list             # список бакетів
gcloud iam service-accounts list        # список service accounts
```

#### 4. Корисні параметри
- `--project` – вказати проєкт
- `--zone`, `--region` – зона або регіон
- `--format=json | table | value(...)` – формат виводу
- `--filter` – фільтрація результату

---

## 3. Контрольні питання (Review Questions)

### 3.1. How does TOTP authentication work? Examples with Google Authenticator and 2-Step Verification.
**TOTP (Time-based One-Time Password)** is a one-time password calculated from a shared secret key and the current time. When 2-Step Verification is set up, Google shows a QR code and the Google Authenticator app saves the secret. Every 30 seconds the app calculates a new 6-digit code, and Google calculates the same code and compares it. After the password, the user must enter the current code, so a stolen password alone is not enough.

### 3.2. Describe the JSON format: where it is used and its features. Is it used in GCP?
**JSON (JavaScript Object Notation)** is a lightweight text format for data exchange. It is built from key-value pairs `{ "key": "value" }` and arrays `[1, 2, 3]`. Types: string, number, boolean, null, object, array. Keys use double quotes, comments are not allowed. It is used in REST APIs, config files and NoSQL databases. In GCP it is used a lot: REST API requests and responses, `gcloud --format=json` output, service account key files, IAM policies, Deployment Manager configs and Firestore documents.

### 3.3. What main APIs does Google Cloud Platform provide?
- **Service REST APIs:** manage resources and data, for example Compute Engine API, Cloud Storage JSON API, BigQuery API, IAM API.
- **gRPC APIs and client libraries:** fast access from code (Python, Java, Go, C#, Node.js).
- **Machine learning APIs:** Vision API, Speech-to-Text, Translation, Vertex AI.
- **API management tools:** API Gateway, Cloud Endpoints and Apigee for publishing and protecting your own APIs.

Every API must be enabled in the project, and requests are authorized with OAuth 2.0 tokens or service accounts.

### 3.4. How can Google Cloud SDK (CLI) be installed on different operating systems?
- **Windows:** the official installer (`GoogleCloudSDKInstaller.exe`).
- **macOS:** `brew install --cask google-cloud-sdk` or the archive from the website.
- **Linux:** the package from Google's apt/yum repository (`sudo apt install google-cloud-cli`), `snap install google-cloud-cli --classic`, or the archive.
- **Without installation:** Cloud Shell in the browser.

After installation run `gcloud init` and check with `gcloud --version`.

### 3.5. Basic gcloud commands with explanations
- `gcloud auth login` – sign in to the Google account.
- `gcloud projects list` – show all projects.
- `gcloud config set project <PROJECT_ID>` – choose the working project.
- `gcloud compute instances list` – show virtual machines.
- `gcloud compute instances create my-vm --zone=europe-west1-b` – create a VM in the zone.
- `gcloud storage buckets list` – show Cloud Storage buckets.

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №8 я ознайомився з хмарною платформою Google Cloud Platform. Було зареєстровано акаунт Free Trial, ознайомлено з інтерфейсом Google Cloud Console, налаштовано двофакторну автентифікацію (2-Step Verification) та створено бюджет Zero Spend Budget для контролю витрат.

Також я створив групи `developers` і `readonly` та розібрався з ролями IAM. Встановив і налаштував Google Cloud SDK (gcloud), перевірив за його допомогою наявність віртуальних машин та склав коротку довідку з базових команд. Окремо розібрався з основними сервісами GCP: Compute Engine, App Engine, Cloud Storage, Cloud Functions та Cloud Run.

**Conclusion (EN):** In this lab I created and secured a Google Cloud account (2-Step Verification, Budgets), managed groups and roles with IAM, and used the gcloud CLI to manage resources from the command line.
