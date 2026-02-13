# СРС Лекція №2

### 1. Завдання
1. A process is basically a program in execution.
2. An address space is a list of memory locations from 0 to some maximum, which the process can read and write.
3. In many OS all the information about each process, other than the contents of its own address space, is stored in an operating system table (or process table).
4. If a process can create one or more other processes (referred to as child processes) and these processes in turn can create child processes, we quickly arrive at the process tree structure.
5. To provide a place to keep files, most operating systems have the concept of a directory as a way of grouping files together.
6. Process hierarchies are typically short-lived, generally minutes at most, whereas the directory hierarchy may exist for years.
7. Before a file can be read, it must be located on the disk and opened, and after it has been read it should be closed, so calls are provided to do these things.

---

### 2. Переклад
1. Процес — це, по суті, програма, що виконується.
2. Адресний простір — це список комірок пам'яті від 0 до певного максимуму, які процес може зчитувати та куди може записувати дані.
3. У багатьох ОС уся інформація про кожен процес, окрім вмісту його власного адресного простору, зберігається в таблиці операційної системи (або таблиці процесів).
4. Якщо процес може створювати один або кілька інших процесів (званих дочірніми процесами), а ці процеси, своєю чергою, можуть створювати власні дочірні процеси, ми швидко приходимо до структури дерева процесів.
5. Щоб забезпечити місце для зберігання файлів, більшість операційних систем мають концепцію каталогу (директорії) як способу групування файлів разом.
6. Ієрархії процесів, як правило, є короткочасними (зазвичай тривають щонайбільше кілька хвилин), тоді як ієрархія каталогів може існувати роками.
7. Перш ніж файл можна буде прочитати, його потрібно знайти на диску та відкрити, а після прочитання його слід закрити; для виконання цих дій передбачені спеціальні системні виклики.