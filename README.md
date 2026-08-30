## Лабораторные работы по базам данных (PostgreSQL)

---

### ЛР1 «Изучение возможностей СУБД PostgreSQL»
**Цель:** знакомство с СУБД PostgreSQL и инструментами управления.

**Выполнено:**
- Работа в интерактивной оболочке **psql**: запуск, создание БД (`CREATE DATABASE test`), вывод списка БД (`\?;`), подключение к БД (`\c test;`), удаление БД (`DROP DATABASE test;`)
- Работа с графическим интерфейсом **pgAdmin4**
- Создание БД с именем `DB_Жданова_6301_2024` через pgAdmin

---

### ЛР2 «Операторы определения данных (создание таблиц)»
**Цель:** изучение DDL-операторов для создания и модификации структуры таблиц.

**Выполнено:**

**Таблица EMPLOYEES:**
```sql
CREATE TABLE EMPLOYEES (
    Num DECIMAL(4, 0) PRIMARY KEY,
    Fname VARCHAR(100) NOT NULL,
    Bday DATE,
    Gender CHAR(1) DEFAULT 'm' CHECK (Gender IN ('f', 'm')),
    Job VARCHAR(30) NOT NULL,
    WageRate DECIMAL(2, 1) DEFAULT 1 CHECK (WageRate BETWEEN 0.1 AND 1.5),
    Sdate DATE NOT NULL,
    Address VARCHAR(200) NOT NULL
);
```

**Таблица JOB_HISTORY:**
```sql
CREATE TABLE JOB_HISTORY (
    Num DECIMAL(4, 0),
    StartDate DATE NOT NULL,
    EndDate DATE,
    Job VARCHAR(30) NOT NULL,
    NumDepartment DECIMAL(2, 0) DEFAULT 1 CHECK (NumDepartment > 0 AND NumDepartment <= 30),
    FOREIGN KEY (Num) REFERENCES EMPLOYEES(Num)
);
```

**Дополнительно:**
- Добавлен столбец `Manager TEXT NOT NULL DEFAULT '1111'` в таблицу EMPLOYEES
- Изменен тип данных столбца Manager на `DECIMAL(6, 0)` с удалением DEFAULT
- Столбец Manager удален (`DROP COLUMN`)
- Вставлены записи через pgAdmin

---

### ЛР3 «Операторы манипулирования данными»
**Цель:** изучение DML-операторов и управление привилегиями.

**Выполнено:**

**DML-операции:**
- **INSERT** — вставка записей в таблицу EMPLOYEES (Jake, Bob, Ann)
- **DELETE** — удаление записей с `num >= 40`
- **UPDATE** — обновление имени сотрудника (`SET fname = 'Ekaterina' WHERE num = 2`)
- **INSERT** — заполнение таблицы JOB_HISTORY тремя записями

**Управление доступом (DCL):**
- Создание пользователя (`CREATE ROLE select_user LOGIN PASSWORD 'my_pass'`)
- Выдача прав на чтение таблицы (`GRANT SELECT ON Employees TO select_user`)
- Проверка прав
- Отзыв прав (`REVOKE SELECT ON TABLE employees FROM select_user`)
- Создание роли с правами на чтение и запись (`GRANT SELECT, INSERT ON Employees TO role1`)
- Назначение роли пользователю (`GRANT role1 TO select_user`)

---

### ЛР4 «Создание запросов и использование их результатов»
**Цель:** изучение операторов SELECT для выборки данных.

**Выполнены запросы:**

1. `SELECT * FROM employees WHERE gender = 'f'` — женщины-сотрудницы
2. `SELECT * FROM employees WHERE bday IS NULL` — сотрудники без даты рождения
3. `SELECT * FROM employees WHERE gender = 'm' AND wagerate > 1.0` — мужчины со ставкой > 1
4. `SELECT COUNT(*) FROM employees WHERE job = 'boss'` — количество сотрудников на должности
5. `SELECT job, COUNT(*) FROM employees GROUP BY job` — должности и количество сотрудников
6. `SELECT * FROM employees, job_history WHERE employees.num = job_history.num AND fname LIKE 'A%' AND numdepartment = 5` — сотрудники отдела №5, фамилия на 'А'
7. `SELECT * FROM employees, job_history WHERE employees.num = job_history.num AND job_history.startdate = '2014-01-01' AND job_history.enddate IS NULL` — сотрудники с должности с 2014
8. `SELECT MIN(wagerate), MAX(wagerate) FROM employees WHERE gender = 'f'` — мин/макс ставка среди женщин

---

### ЛР5 «Внешние объединения, вложенные запросы»
**Цель:** изучение сложных запросов с JOIN, подзапросами и агрегацией.

**Выполнено:**

**Создание таблицы DEPARTMENT:**
```sql
CREATE TABLE DEPARTMENT (
    NumDepartment NUMERIC(2) CHECK (NumDepartment > 0 AND NumDepartment <= 30) UNIQUE,
    Name VARCHAR(50)
);
```

**Запросы:**
1. `SELECT fname FROM Employees WHERE sdate >= CURRENT_DATE - INTERVAL '2 years'` — сотрудники, принятые за последние 2 года
2. `SELECT COUNT(*) FROM employees, job_history WHERE employees.num = job_history.num AND employees.wagerate > 1 AND job_history.enddate IS NULL AND job_history.numdepartment = (SELECT numdepartment FROM department WHERE name = 'Отдел снабжения')` — кол-во сотрудников со ставкой > 1 в отделе снабжения
3. `SELECT employees.* FROM employees JOIN job_history ON employees.num = job_history.Num GROUP BY employees.num HAVING COUNT(job_history.job) = 1` — сотрудники, не менявшие должность
4. `SELECT employees.Fname FROM employees JOIN job_history ON employees.num = job_history.num WHERE EXTRACT(YEAR FROM startdate) = EXTRACT(YEAR FROM CURRENT_DATE) GROUP BY fname HAVING COUNT(job_history.job) > 1` — меняли должность в текущем году несколько раз
5. `SELECT department.name FROM job_history JOIN employees ON job_history.num = employees.num JOIN department ON job_history.numdepartment = department.numdepartment GROUP BY department.name HAVING COUNT(job_history.job) = (SELECT MAX(max_count) FROM (SELECT COUNT(*) AS max_count FROM job_history GROUP BY job))` — отдел, где чаще всего меняли должности
6. `SELECT employees.fname FROM employees JOIN job_history ON employees.num = job_history.num WHERE job_history.job = (SELECT job FROM employees WHERE num = 2)` — сотрудники, занимавшие должность заданного сотрудника
7. `SELECT DISTINCT employees.fname FROM employees JOIN job_history ON employees.num = job_history.num WHERE job_history.numdepartment IN (SELECT numdepartment FROM department WHERE name IN ('Отдел снабжения', 'Отдел плановый'))` — сотрудники отделов снабжения и планового

---

### ЛР6 «Создание и использование представлений SQL»
**Цель:** изучение механизма VIEW для упрощения доступа к данным.

**Выполнено:**

1. **Простое представление:**
```sql
CREATE VIEW EMPLOYEES_VIEW AS
SELECT Num, Fname, Bday, Gender, Job, WageRate, Sdate, Address FROM EMPLOYEES
```
Вставка нового сотрудника через представление

2. **Представление с CHECK OPTION:**
```sql
CREATE VIEW EMPLOYEES_MENVIEW AS
SELECT * FROM EMPLOYEES
WHERE Gender = 'm' AND Sdate > '2014-01-01'
WITH CHECK OPTION;
```
Предотвращает вставку строк, не соответствующих условию

3. **Представление на основе представления:**
```sql
CREATE VIEW EMPLOYEES_MENVIEW2 AS
SELECT * FROM EMPLOYEES_MENVIEW
WHERE Job = 'IT'
WITH CHECK OPTION;
```

4. `CREATE VIEW FNAME_NUMDEPARTMENT5 AS SELECT employees.fname FROM employees JOIN job_history ON employees.num = job_history.num WHERE job_history.numdepartment = 5` — фамилии сотрудников отдела №5

5. **Представление с подзапросом:**
```sql
CREATE VIEW MaxDepartment AS
SELECT DEPARTMENT.NumDepartment, DEPARTMENT.Name AS EmployeeCount
FROM DEPARTMENT
JOIN JOB_HISTORY ON DEPARTMENT.NumDepartment = JOB_HISTORY.NumDepartment
GROUP BY DEPARTMENT.NumDepartment, DEPARTMENT.Name
HAVING COUNT(JOB_HISTORY.NumDepartment) = (
    SELECT MAX(Ecount) FROM (SELECT COUNT(*) AS Ecount FROM JOB_HISTORY GROUP BY NumDepartment)
);
```

6. **Обновление через представление:**
```sql
CREATE VIEW VIEWMAX AS
SELECT EMPLOYEES.* FROM EMPLOYEES
WHERE EMPLOYEES.WageRate < 0.5 AND Num IN (SELECT Num FROM JOB_HISTORY WHERE NumDepartment = 5);
UPDATE VIEW_MAX SET WageRate = WageRate + 0.5
```

---

## Индивидуальное домашнее задание (ИДЗ)

**Тема:** Моделирование предметной области «Сессия»

### Описание предметной области
Сессия — период сдачи экзаменов и зачетов студентами по итогам семестра.

### Выделенные сущности
| Сущность | Атрибуты |
|----------|----------|
| **Студенты** | ФИО, номер группы, номер зачетной книжки, направление подготовки, форма обучения |
| **Преподаватели** | id, ФИО, кафедра, должность, email |
| **Экзамены** | id, название предмета, тип (зачет/дифф.зачет/экзамен), дата, время |
| **Аудитории** | id, номер кабинета, номер корпуса, вместимость |
| **Оценки** | id, оценка (числовая), номер зачетной книжки, название предмета |

### Бизнес-правила
- К каждому экзамену привязан только один преподаватель
- В аудитории не может быть более одного экзамена в одно время

### Категории пользователей
| Роль | Функциональность |
|------|------------------|
| **Студент** | просмотр расписания, просмотр своих оценок и успеваемости |
| **Преподаватель** | выставление оценок, просмотр списка студентов, формирование отчетов |
| **Администратор** | управление данными (CRUD), настройка расписания, генерация отчетов |

