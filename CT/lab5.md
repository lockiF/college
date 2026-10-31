# Лабораторна робота №5
**Виконав студент групи РПЗ-34 Козуб Макар**

**Дисципліна:** Хмарні технології та сервіси
**Тема:** Дослідження сервісів та компонентів AWS
**Мета:** Знайомство з популярним хмарним провайдером AWS, робота з різними сервісами у середовищі AWS.

---

## 1. Хід роботи (Work Progress)

### Практичне завдання №1. Огляд Storage-as-a-Service

#### 1. В чому суть концепції Storage-as-a-Service, коли її доцільно використовувати?
**Storage-as-a-Service (STaaS)** – це модель, коли провайдер надає місце для зберігання даних в оренду через Інтернет. Не треба купувати власні диски й сервери, а платиш тільки за використаний обсяг. Її доцільно використовувати для резервних копій, архівів, файлів сайтів, відео та фото, а також коли обсяг даних швидко росте.

#### 2. Приклади сервісів за моделлю Storage-as-a-Service
Amazon S3, Google Cloud Storage, Azure Blob Storage, Google Drive, Dropbox, Backblaze B2.

#### 3. Порівняйте поняття бакету та віртуальної машини на прикладі сервісів AWS
- **Бакет (Amazon S3):** контейнер для зберігання файлів (об'єктів). Немає операційної системи, на ньому не можна запускати програми. Розмір росте автоматично, оплата за гігабайти. Доступ через HTTP/API.
- **Віртуальна машина (Amazon EC2):** віртуальний сервер із ОС, процесором і пам'яттю. На ній можна запускати програми, але її треба налаштовувати й оновлювати самому. Оплата за час роботи.

Тобто бакет потрібен для зберігання даних, а віртуальна машина – для їх обробки.

#### 4. Для чого потрібні Versioning, Lifecycle та Replication
- **Versioning:** зберігає всі версії об'єкта. Захищає від випадкового видалення чи перезапису, можна повернути стару версію.
- **Lifecycle:** правила, які автоматично переносять об'єкти в дешевший клас зберігання (наприклад, Glacier) або видаляють їх через певний час. Це економить гроші.
- **Replication:** автоматично копіює об'єкти в інший бакет (в іншому регіоні або в тому самому). Потрібна для захисту від аварій, швидшого доступу та вимог закону. Для неї потрібно ввімкнути Versioning.

#### 5. Методи захисту від несанкціонованого доступу до бакетів
- **IAM policies:** права користувачів і ролей.
- **Bucket policies:** JSON-правила для конкретного бакету.
- **ACL:** простіші права на бакет або об'єкт.
- **Block Public Access:** забороняє випадкове відкриття бакету всім.
- **Шифрування:** SSE-S3, SSE-KMS (на сервері) та HTTPS (під час передачі).
- **Presigned URL:** тимчасове посилання на файл.
- **MFA Delete, CloudTrail та логи доступу:** захист від видалення й контроль дій.

---

### Практичне завдання №2. Огляд Function-as-a-Service (FaaS)

#### 6. В чому суть концепції FaaS, коли її доцільно використовувати?
**Function-as-a-Service (FaaS)** – це модель, у якій розробник пише невелику функцію, а провайдер сам запускає її, коли настає подія (запит, завантаження файлу, таймер). Сервери не потрібно налаштовувати, ресурси масштабуються автоматично, а оплата йде лише за час виконання функції. Доцільно для коротких задач за подіями: обробка файлів, API, сповіщення, завдання за розкладом. Для довгих постійних процесів вона не підходить (AWS Lambda працює до 15 хвилин).

#### 7. Поняття «Serverless-архітектури»
**Serverless-архітектура** – це підхід, коли розробник не керує серверами, а всім займається провайдер: виділяє ресурси, масштабує, оновлює. Фізичні сервери, звісно, існують, але ми їх не бачимо. Оплата йде за фактичне використання.

#### 8. Порівняйте serverless-архітектуру та FaaS
Serverless – це ширше поняття: сюди входять не лише функції, а й бази даних, сховища, черги, API (S3, DynamoDB, SQS, API Gateway). **FaaS** – це лише частина serverless, яка відповідає за запуск коду. Тобто будь-який FaaS є serverless, але не весь serverless є FaaS.

#### 9. Приклади serverless-рішень та FaaS-сервісів
- **FaaS:** AWS Lambda, Azure Functions, Google Cloud Functions, Cloudflare Workers.
- **Інші serverless-сервіси:** Amazon S3, DynamoDB, API Gateway, AWS Fargate, Aurora Serverless, Firebase.

#### 10. Приклад реальної задачі для FaaS на платформі AWS
**Задача:** автоматично створювати зменшені картинки (мініатюри) для фото, які користувачі завантажують на сайт.

Що потрібно зробити:
1. Створити два S3-бакети: для оригіналів і для мініатюр.
2. Створити IAM-роль для функції з правами читати й писати в S3 та писати логи в CloudWatch.
3. Створити функцію AWS Lambda (наприклад, на Python) і додати код зменшення картинки (бібліотеку Pillow підключити через Layer).
4. Додати **тригер S3** на подію `ObjectCreated` у бакеті з оригіналами.
5. Завантажити тестове фото й перевірити, що мініатюра з'явилась у другому бакеті, а логи є в CloudWatch.

---

### Практичне завдання №3. Робота з реляційними базами даних в AWS

#### 11. Можливості Amazon RDS. До якої хмарної моделі її можна віднести?
**Amazon RDS (Relational Database Service)** – керований сервіс реляційних баз даних. AWS сам робить резервні копії, оновлення, моніторинг, може створювати репліки та забезпечувати відмовостійкість (Multi-AZ). Розробнику залишається працювати лише з даними. RDS відноситься до моделі **PaaS** (також її називають DBaaS).

#### 12. Які бази даних підтримує Amazon RDS. Порівняйте їх
RDS підтримує: **MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server, IBM Db2** та **Amazon Aurora**.
- **MySQL:** простий і популярний, добре підходить для вебсайтів.
- **PostgreSQL:** більше можливостей (складні запити, JSON, розширення), безкоштовний.
- **MariaDB:** відкритий форк MySQL, сумісний із ним.
- **Oracle та SQL Server:** комерційні, потрібна ліцензія, використовуються в корпоративних системах.
- **IBM Db2:** корпоративна база, використовується у великих компаніях.
- **Amazon Aurora:** база від AWS, сумісна з MySQL і PostgreSQL, працює швидше, сховище росте автоматично.

#### 13. Порівняйте Amazon RDS (PaaS) та EC2 + MySQL вручну (IaaS)
- **RDS:** база створюється за кілька кліків. Резервні копії, оновлення та відмовостійкість робить AWS. Мінус: немає доступу до ОС сервера й менше налаштувань.
- **EC2 + MySQL:** треба самому створити сервер, встановити й налаштувати базу, робити копії та оновлення. Плюс: повний контроль над системою.

RDS доцільно брати для більшості проєктів, щоб менше адмініструвати. EC2 потрібен, коли треба повний контроль над ОС, особливі налаштування або версія бази, якої немає в RDS.

#### 14. Для чого потрібні масштабування, снепшоти, моніторинг та логування баз даних
- **Масштабування:** база витримує більше користувачів (більший інстанс, репліки для читання, збільшення диска).
- **Снепшоти:** резервні копії для відновлення даних після помилки чи збою.
- **Моніторинг:** показує навантаження (CPU, пам'ять, з'єднання), щоб помітити проблеми заздалегідь (Amazon CloudWatch).
- **Логування:** записи помилок і повільних запитів допомагають знайти причину проблем і перевірити, хто що робив.

#### 15. Порівняйте реляційні та NoSQL бази даних. Чи підтримує NoSQL сервіс Amazon RDS?
- **Реляційні:** дані в таблицях зі строгою схемою, мова SQL, зв'язки між таблицями, транзакції (ACID). Масштабуються переважно вертикально.
- **NoSQL:** гнучка схема, дані як документи, пари «ключ-значення», стовпці чи графи. Легко масштабуються горизонтально й дуже швидкі.

Amazon RDS **не підтримує** NoSQL, лише реляційні бази. Для NoSQL в AWS є окремі сервіси: DynamoDB, DocumentDB, Keyspaces.

---

### Практичне завдання №4. Робота з NoSQL базами даних в AWS

#### 16. Класифікація сучасних NoSQL баз даних. Де вони використовуються?
- **Ключ-значення:** Redis, DynamoDB. Використовують для кешу, сесій, кошиків.
- **Документні:** MongoDB, DocumentDB. Для каталогів товарів, профілів, контенту.
- **Стовпцеві (wide-column):** Cassandra, HBase. Для великих обсягів даних, IoT, логів.
- **Графові:** Neo4j, Amazon Neptune. Для соціальних мереж та рекомендацій.

#### 17. Можливості та застосування Amazon DynamoDB
**Amazon DynamoDB** – повністю керована serverless NoSQL база типу «ключ-значення» та документів. Має відповідь за мілісекунди, автоматичне масштабування, резервні копії, глобальні таблиці (Global Tables), кеш DAX та Streams. Застосовується для ігор (рейтинги), інтернет-магазинів (кошики), IoT, сесій користувачів.

#### 18. Основні поняття DynamoDB: Primary Key, WCU та RCU
- **Primary Key:** унікальний ідентифікатор запису. Це або лише **partition key**, або **partition key + sort key**.
- **RCU (Read Capacity Unit):** одиниця читання, одне строго узгоджене читання на секунду запису до 4 КБ.
- **WCU (Write Capacity Unit):** одиниця запису, один запис на секунду розміром до 1 КБ.

Ці одиниці задають пропускну здатність таблиці в режимі provisioned, а в режимі on-demand AWS рахує оплату за кожен запит.

#### 19. Порівняйте DynamoDB, DocumentDB та Keyspaces
- **DynamoDB:** serverless, ключ-значення та документи, власний API від AWS, будь-який масштаб.
- **Amazon DocumentDB:** документна база з сумісністю з **MongoDB**, зберігає JSON-документи, працює на кластері з інстансів.
- **Amazon Keyspaces:** serverless база, сумісна з **Apache Cassandra**, стовпцева, мова запитів CQL.

#### 20. Приклади задач та як розгорнути в AWS
- **DynamoDB:** кошик інтернет-магазину чи рейтинги гравців. Розгортання: консоль → DynamoDB → **Create table** → вказати назву й Primary Key → вибрати режим потужності (on-demand або provisioned).
- **DocumentDB:** каталог товарів у JSON або перенесення застосунку з MongoDB. Розгортання: консоль → DocumentDB → **Create cluster** → вибрати тип інстансу, створити користувача, вказати VPC та Security Group, підключитися через mongo shell.
- **Keyspaces:** запис даних з тисяч IoT-датчиків або міграція з Cassandra. Розгортання: консоль → Keyspaces → **Create keyspace** → створити таблицю через CQL-редактор.

---

## 2. Контрольні питання (Review Questions)

### 2.1. Give examples of tasks where Storage-as-a-Service is needed, using an online store and a game development company.
- **Online store:** storing product photos and videos, invoices and order documents, database backups and logs. Static site files can be hosted in Amazon S3 too, and S3 handles the load during sales.
- **Game development company:** storing game builds, textures, models and other assets, patches and updates for players to download, and backups of project files. Together with a CDN (CloudFront), players can download the game fast from any country.

### 2.2. Give examples of tasks where Function-as-a-Service (FaaS) is needed, using online banking and a mobile app development company.
- **Online banking:** checking every payment for fraud right after it happens, sending SMS or email notifications about transactions, generating monthly statements on a schedule, processing uploaded documents.
- **Mobile app development company:** the backend API for an app (API Gateway + Lambda), resizing avatars uploaded by users, sending push notifications, handling in-app purchase webhooks.

### 2.3. Give examples of tasks where DataBase-as-a-Service (DBaaS) is needed, using Internet of Things and artificial intelligence projects.
- **Internet of Things:** storing data from thousands of sensors (temperature, location). A managed database like DynamoDB scales automatically when new devices are added, and no one needs to administer servers.
- **Artificial intelligence:** storing user data, chat history, datasets and prediction results for an AI service. A managed database (for example, Amazon RDS for PostgreSQL) gives backups and high availability, so the team can focus on the model.

---

## 3. Висновки (Conclusions)

Під час виконання лабораторної роботи №5 я ознайомився з основними сервісами AWS: Amazon S3 (Storage-as-a-Service), AWS Lambda (Function-as-a-Service), Amazon RDS (реляційні бази даних) та Amazon DynamoDB, DocumentDB і Keyspaces (NoSQL бази даних). Я розібрався, чим бакет відрізняється від віртуальної машини, для чого потрібні Versioning, Lifecycle і Replication, а також як захистити дані в S3.

Також я зрозумів різницю між serverless і FaaS, між PaaS-моделлю (RDS) та ручним розгортанням бази на EC2 (IaaS), і між реляційними та NoSQL базами. Керовані сервіси економлять час, бо AWS бере на себе адміністрування, а розробник може зосередитися на самому проєкті.

**Conclusion (EN):** In this lab I studied the main AWS services: S3 for storage, Lambda for serverless functions, RDS for relational databases, and DynamoDB, DocumentDB and Keyspaces for NoSQL databases. I learned when to use each service and how managed services reduce administration work.
