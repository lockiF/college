# Лабораторна робота №3
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Реєстрація та знайомство з хмарним середовищем AWS
**Мета:** Реєстрація та знайомство з популярним хмарним провайдером AWS, реєстрація акаунту та огляд його інтерфейсу.

---

## 1. Завдання попередньої підготовки (Prerequisite Tasks)

### Відповіді на питання (Answers to Questions):

#### 1.1. Core Concepts and Terms (from Video 1)
- **Global Infrastructure:**
  - **AWS Region:** A geographic area where AWS has several data center groups (e.g., `eu-central-1` – Frankfurt, `us-east-1` – N. Virginia).
  - **Availability Zone (AZ):** One or more isolated data centers inside a Region with their own power and network. Using several AZs gives high availability.
  - **Edge Locations:** Points of presence used by Amazon CloudFront (CDN) and Route 53 to deliver content to users with lower latency.
- **Service Categories and Core Services:**
  - **Compute:** Amazon EC2 (virtual servers), AWS Lambda (serverless functions), AWS Fargate, Elastic Beanstalk.
  - **Storage:** Amazon S3 (object storage), Amazon EBS (block storage for EC2), Amazon EFS (file storage), S3 Glacier (archive).
  - **Databases:** Amazon RDS (relational DB), Amazon Aurora, Amazon DynamoDB (NoSQL), ElastiCache.
  - **Networking:** Amazon VPC (virtual network), Route 53 (DNS), CloudFront (CDN), Elastic Load Balancing.
  - **Security & Management:** AWS IAM, Amazon CloudWatch (monitoring), AWS CloudTrail (audit), AWS Budgets, CloudFormation.
- **Pricing & Billing:** *Pay-as-you-go* model (you pay only for what you use), *AWS Free Tier* with free limits for new accounts, *Shared Responsibility Model* (AWS secures the cloud, the customer secures what is in the cloud).

#### 1.2. Multi-Factor Authentication (MFA) and Configuration Options
- **Purpose of MFA:** It adds a second authentication factor on top of the password. Even if the password is stolen, an attacker can't log in without the second factor. For the root user MFA is a must, because root has unlimited access and can see billing.
- **How to set up:** Console → account name → *Security credentials* → *Multi-factor authentication (MFA)* → *Assign MFA device* → enter device name and choose type → scan QR code → enter two consecutive codes → *Add MFA*.
- **MFA Options in AWS (from Video 2):**
  - **Authenticator App (Virtual MFA):** TOTP apps on a phone (Google Authenticator, Authy, Microsoft Authenticator).
  - **Passkey / FIDO2 Security Key:** Physical key (e.g., YubiKey) or built-in biometrics (Windows Hello, Touch ID).
  - **Hardware TOTP Token:** A separate physical device that shows one-time codes.

#### 1.3. Amazon Budgets: Purpose and Setup Options
- **Purpose:** A service that tracks AWS costs and usage and sends alerts when the actual or forecasted spend goes over a defined threshold. It protects from unexpected bills.
- **Setup Options (from Video 2):**
  - **Zero Spend Budget:** Sends a notification as soon as costs exceed $0.01 (good for staying inside Free Tier).
  - **Monthly Cost Budget:** A monthly limit (e.g., $10) with alerts at a chosen percentage (e.g., 80%).
  - **Usage Budget:** Tracks usage of specific resources (e.g., EC2 hours, GB in S3).
  - **Savings Plans / Reservation Budget:** Tracks coverage and utilization of Savings Plans or Reserved Instances.
  - Additional settings: period (daily / monthly / quarterly / yearly), alert type (*Actual* or *Forecasted*), notifications by email or Amazon SNS.

#### 1.4. Purpose of IAM Policy and Managing User Permissions
- **Purpose:** An IAM Policy is a JSON document that defines what is allowed (`Allow`) or denied (`Deny`) for which actions (`Action`) on which resources (`Resource`). A new user has no permissions by default.
- **Permission Setup:** Create user groups by role (developers, readonly, admins), attach policies to the groups and add users to them. Follow the Principle of Least Privilege – give only the permissions that are really needed.

#### 1.5. Options for Attaching IAM Policies When Creating a New User
1. **Add user to group:** The user inherits the policies of an existing group (best practice).
2. **Copy permissions:** Copies group membership and policies from another existing user.
3. **Attach policies directly:** Attaches AWS managed or custom policies right to the user (harder to manage when there are many users).

#### 1.6. Writing and Applying Access Control via IAM Policies
A policy is a JSON document with statements:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```
`Effect` – Allow or Deny, `Action` – API actions, `Resource` – ARN of the resource, `Condition` – optional conditions (IP address, MFA, time). An explicit `Deny` always overrides `Allow`. To apply: *IAM → Policies → Create policy* (Visual editor or JSON) → then *Attach policies* to a user, group or role.

#### 1.7. Switching IAM Roles for Secure Access
Switching roles lets a user temporarily assume a role and get short-living credentials (via AWS STS) instead of permanent keys. Steps: create a role with a trust policy → attach permissions to it → give the user the `sts:AssumeRole` permission → in the Console click the profile menu → **Switch role** → enter Account ID, Role name and display name → **Switch Role**. To go back, choose *Switch back*.

#### 1.8. Permissions Boundaries for IAM Roles and Users
A **Permissions Boundary** sets the maximum permissions that a user or role can have. Effective permissions are the intersection of the identity policy and the boundary. So even with `AdministratorAccess` the user can't do anything outside the boundary. Setup: *IAM → Users (or Roles) → Permissions → Set permissions boundary → choose policy*. The boundary itself doesn't give any permissions.

#### 1.9. Tracking and Analyzing User Actions with AWS CloudTrail
**AWS CloudTrail** records API calls and actions made through the Console, CLI and SDKs: who did it, what, when, and from which IP. In *Event history* the last 90 days are available and can be filtered by user name, event name or resource type. For long-term storage a **Trail** is created, which saves logs to an S3 bucket (and optionally to CloudWatch Logs).

#### 1.10. IAM Policy Simulator Purpose
The **IAM Policy Simulator** is a tool for testing and troubleshooting IAM policies. It shows whether an action is allowed or denied for a given user, group or role and which policy caused it, without executing the real action.

---

## 2. Хід роботи (Work Progress)

### 2.1. Реєстрація акаунту AWS Free Tier
Відкрив сайт `aws.amazon.com/free` і натиснув **Create a Free Account**. Далі виконав кроки:
1. Ввів email root-користувача та назву акаунту, підтвердив email кодом з листа.
2. Задав пароль для root-користувача.
3. Заповнив контактну інформацію (тип акаунту **Personal**, ПІБ, телефон, адреса).
4. Ввів дані банківської картки. AWS робить тимчасове утримання невеликої суми (~$1) для підтвердження особи, потім кошти повертаються.
5. Підтвердив номер телефону через SMS-код.
6. Обрав план підтримки **Basic support – Free**.

Після активації акаунту увійшов у AWS Management Console.

> **Опис скріншота 1 (AWS Management Console):**
> На скріншоті показано головну сторінку AWS Management Console. Зверху розташована панель пошуку сервісів, вибір регіону (`Europe (Frankfurt) eu-central-1`) та меню акаунту. Посередині віджети *Recently visited*, *Welcome to AWS* та *Cost and usage*. Особисті дані (email, Account ID) приховано.

---

### 2.2. Налаштування MFA для root-акаунту
Для захисту root-акаунту налаштував двофакторну автентифікацію через застосунок Google Authenticator:
1. Клік на ім'я акаунту (правий верхній кут) → **Security credentials**.
2. Розділ **Multi-factor authentication (MFA)** → **Assign MFA device**.
3. Device name: `root-mfa`, тип: **Authenticator app** → Next.
4. Відсканував QR-код у Google Authenticator.
5. Ввів два послідовні 6-значні коди → **Add MFA**.

Для перевірки вийшов з акаунту і зайшов знову, після пароля система запитала код MFA.

> **Опис скріншота 2 (MFA Configuration):**
> На скріншоті показано сторінку *Security credentials* з розділом *Multi-factor authentication (MFA)*. У таблиці відображено пристрій `root-mfa` з типом `Virtual` та статусом `Assigned`.

---

### 2.3. Створення бюджету "Zero Spend Budget"
1. У пошуку Console ввів **Billing and Cost Management** і відкрив сервіс.
2. Ліве меню → **Budgets** → **Create budget**.
3. Обрав **Use a template (simplified)** → шаблон **Zero spend budget**.
4. Budget name: `My Zero-Spend Budget`, вказав свою email-адресу для сповіщень.
5. Натиснув **Create budget**.

> **Опис скріншота 3 (AWS Budgets):**
> На скріншоті показано сторінку *Budgets*. У списку є бюджет `My Zero-Spend Budget` з параметрами: Budget amount – `$0.01`, Current spend – `$0.00`, Status – `OK`.

---

### 2.4. Створення IAM Group та користувачів у IAM
1. У пошуку Console знайшов **IAM** → **User groups** → **Create group**.
2. Назва групи: `developers`, прикріпив політику `PowerUserAccess` → **Create user group**.
3. **Users** → **Create user**. Увімкнув *Provide user access to the AWS Management Console* → *I want to create an IAM user*, задав пароль.
4. На кроці *Set permissions* обрав **Add user to group** → `developers`.
5. Повторив для трьох користувачів: `makar.kozub` (я), `[ім'я одногрупника 1]`, `[ім'я одногрупника 2]`.

> **Опис скріншота 4 (IAM User Group `developers`):**
> На скріншоті показано сторінку групи `developers`. На вкладці *Users* відображено трьох користувачів (`makar.kozub` та двох одногрупників), на вкладці *Permissions* прикріплено політику `PowerUserAccess`.

---

### 2.5. Налаштування через AWS IAM Identity Center
1. У пошуку знайшов **IAM Identity Center** → **Enable** (автоматично створюється AWS Organization).
2. **Groups** → **Create group** → `developers`.
3. **Users** → **Add user**: заповнив username, email, ім'я та прізвище → у кроці *Add user to groups* обрав `developers`. Повторив для трьох користувачів, як у п. 2.4.
4. **Groups** → **Create group** → `readonly`.
5. Відкрив групу `readonly` → **Add users to group** → додав користувача `makar.kozub` (він уже є в `developers`).
6. Перейшов у **Users** → `makar.kozub` → вкладка **Groups**, щоб побачити членство.

> **Опис скріншота 5 (IAM Identity Center User Groups):**
> На скріншоті показано картку користувача `makar.kozub` в IAM Identity Center. На вкладці *Groups* відображено дві групи: `developers` та `readonly`.

---

### 2.6. Встановлення AWS CLI
Завантажив MSI-інсталятор AWS CLI v2 з офіційного сайту `aws.amazon.com/cli`, запустив і пройшов кроки встановлення (Next → Install → Finish). Перезапустив термінал і перевірив версію:

```bash
aws --version
```

> **Опис скріншота 6 (AWS CLI Version):**
> На скріншоті показано вікно PowerShell з виконаною командою `aws --version`. Результат: рядок виду `aws-cli/2.x.x Python/3.x.x Windows/10 exe/AMD64`, що підтверджує успішне встановлення.

---

### 2.7. Конфігурація AWS CLI
Для доступу створив Access Key: IAM → Users → `makar.kozub` → *Security credentials* → **Create access key** → use case *Command Line Interface (CLI)*. Далі виконав команду:

```bash
aws configure
```

Ввів параметри:
- `AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE`
- `AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
- `Default region name [None]: eu-central-1`
- `Default output format [None]: json`

Перевірив доступ командою `aws sts get-caller-identity`.

> **Опис скріншота 7 (AWS Configure):**
> На скріншоті показано термінал, де після `aws configure` послідовно введено Access Key ID, Secret Access Key (частково замасковано), регіон `eu-central-1` та формат виводу `json`. Нижче результат `aws sts get-caller-identity` з ARN користувача `makar.kozub`.

---

### 2.8. Перевірка запущених віртуальних машин
Для перегляду EC2-інстансів використав команду:

```bash
aws ec2 describe-instances
```

> **Опис скріншота 8 (EC2 Describe Instances):**
> На скріншоті показано виконання команди `aws ec2 describe-instances`. У відповідь повернуто `{ "Reservations": [] }`, тобто запущених віртуальних машин у регіоні `eu-central-1` немає.

---

### 2.9. Довідник з використання AWS CLI (Cheat Sheet)

#### 1. Структура команд
```bash
aws <service-name> <command-name> [options/parameters]
```

#### 2. Довідка (Help)
- `aws help` – загальна довідка
- `aws ec2 help` – довідка по сервісу
- `aws ec2 describe-instances help` – довідка по команді

#### 3. Базові команди

##### Налаштування та ідентифікація:
```bash
aws configure                    # налаштування ключів і регіону
aws configure list               # поточна конфігурація
aws sts get-caller-identity      # хто я зараз
```

##### EC2:
```bash
aws ec2 describe-instances
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name]" --output table
aws ec2 start-instances --instance-ids i-xxxxxxxx
aws ec2 stop-instances --instance-ids i-xxxxxxxx
```

##### S3:
```bash
aws s3 ls                                # список бакетів
aws s3 mb s3://my-bucket-name            # створити бакет
aws s3 cp myfile.txt s3://my-bucket-name/  # завантажити файл
```

##### IAM:
```bash
aws iam list-users
aws iam list-groups
```

#### 4. Корисні опції
- `--region` – вказати регіон
- `--profile` – використати іменований профіль
- `--output json | text | table` – формат виводу
- `--query` – фільтрація результату

Конфігурація зберігається у файлах `~/.aws/credentials` (ключі) та `~/.aws/config` (регіон, формат).

---

## 3. Контрольні питання (Review Questions)

### 3.1. What is the essence of TOTP authentication? Provide examples.
**TOTP (Time-based One-Time Password)** is an algorithm that generates a one-time password from a shared secret key and the current time.
- **Core Concept:** When MFA is set up, the server and the app get the same secret (via QR code). Every 30 seconds the app calculates a new 6-digit code from the secret and the current time. The server calculates the same code and compares it with the entered one. Each code is valid for a short time only.
- **Examples:** Google Authenticator, Authy, Microsoft Authenticator, 2FAS. It is used in AWS, Google, GitHub, etc.

### 3.2. Describe the JSON format: where is it used, and what are its features? Is JSON used in AWS?
**JSON (JavaScript Object Notation)** is a lightweight text format for data exchange, easy to read for humans and easy to parse for machines.
- **Features:** Based on key-value pairs (`{ "key": "value" }`) and arrays (`[1, 2, 3]`). Supported types: string, number, boolean, null, object, array. Keys and strings use double quotes, comments are not allowed. It doesn't depend on a programming language.
- **Usage:** REST APIs, config files (`package.json`), NoSQL databases, logs.
- **Usage in AWS:** Yes. JSON is used for IAM policies, CloudFormation templates, AWS CLI output, Lambda events, S3 bucket policies, CloudTrail logs and DynamoDB items.

### 3.3. What types of Amazon APIs exist? Briefly describe them.
In Amazon API Gateway there are three types:
1. **REST API:** Full-featured API with API keys, usage plans, caching, request validation and different authorizers.
2. **HTTP API:** A simpler, faster and cheaper version for proxying requests to Lambda or HTTP backends.
3. **WebSocket API:** Two-way communication in real time (chats, games, live dashboards).

Also AWS service APIs are divided into **Control Plane APIs** (managing resources, e.g., creating an S3 bucket) and **Data Plane APIs** (working with data, e.g., `PutObject`, `GetItem`). All requests are signed with Signature Version 4.

### 3.4. How can AWS CLI be installed across different Operating Systems?
- **Windows:** Run the official MSI installer, or `winget install Amazon.AWSCLI`.
- **macOS:** Use the PKG installer, or `brew install awscli`.
- **Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
After installation check with `aws --version`.

### 3.5. Review basic AWS CLI commands. Provide examples.
- `aws configure` – configure credentials, region and output format.
- `aws sts get-caller-identity` – show the current identity.
- `aws s3 ls` – list S3 buckets.
- `aws s3 cp file.txt s3://my-bucket/` – upload a file to S3.
- `aws ec2 describe-instances` – list EC2 instances.
- `aws iam list-users` – list IAM users.

---

## 4. Висновки (Conclusions)

Під час виконання лабораторної роботи №3 я ознайомився з хмарною платформою Amazon Web Services (AWS). Було зареєстровано акаунт AWS Free Tier, захищено його за допомогою двофакторної автентифікації (MFA) та налаштовано бюджет Zero Spend Budget в AWS Budgets для контролю витрат.

Також я попрацював з сервісами керування доступом **AWS IAM** та **AWS IAM Identity Center**: створив групи `developers` і `readonly`, користувачів та розібрався з політиками, ролями і permissions boundary. Встановив і сконфігурував **AWS CLI**, за допомогою якого перевірив наявність віртуальних машин та склав коротку довідку з базових команд.

**Conclusion (EN):** In this lab I created and secured an AWS account (MFA, Budgets), learned how to manage users and permissions with IAM and IAM Identity Center, and used AWS CLI to manage resources from the command line.
