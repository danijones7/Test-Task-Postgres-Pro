# 1. Установка PostgreSQL через Docker

Устанавливаем Docker с официального сайта 

<https://docs.docker.com/get-docker/>

Запускаем терминал и скачиваем образ PostgreSQL
```
docker pull postgres
```
Создаем и запускем контейнер PostgreSQL
```
docker run --name my_postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```
Проверяем, что контейнер запущен:
```
docker ps
```
Подключаемся к базе данных PostgreSQL внутри контейнера:
```
docker exec -it my_postgres psql -U postgres
```
Выйти из psql
```
\q
```

# 2. Создание БД academy

Создаем файл "create_academy_db" в указанной директории
```
echo "CREATE DATABASE academy;" > C:\Users\Sowic\Desktop\Jessie\GB\PGStart\create_academy_db.sql
```
Копируем созданный файл в контейнер PostgreSQL
```
docker cp C:\Users\Sowic\Desktop\Jessie\GB\PGStart\create_academy_db.sql my_postgres:/create_academy_db.sql
```
Выполненяем скрипта внутри контейнера:
```
docker exec -it my_postgres psql -U postgres -f /create_academy_db.sql
```
Подключаемся к базе данных 
```
docker exec -it my_postgres psql -U postgres -d academy
```


# 2. Скрипт для создания таблиц

```
CREATE TABLE Students (
    s_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
	start_year DATE NOT NULL
);
```
```
CREATE TABLE Courses (
    c_no SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
	hours INT NOT NULL
);
```
```
CREATE TABLE Exams (
	PRIMARY KEY (s_id, c_no),
	s_id INT NOT NULL,
    c_no INT NOT NULL,
    score INT CHECK (score >= 1 AND score <= 5),
    FOREIGN KEY (c_no) REFERENCES Courses(c_no),
    FOREIGN KEY (s_id) REFERENCES Students(s_id)
);
```
# 3: Вставка данных в таблицы

```
INSERT INTO Students (name, start_year) VALUES 
('Max Miles', '01.09.2000'), 
('Freddy McDowel', '01.09.2000'), 
('Mark Browns', '01.09.2000') ;
```
```
INSERT INTO Courses (title, hours) VALUES 
('Math', 40),
('Physics', 55), 
('Chemistry', 77),
('English Literature', 50);
```
```
INSERT INTO Exams (s_id, c_no, score) VALUES
(1, 1, 5),
(1, 2, 4),
(1, 3, 5),
(2, 1, 3),
(3, 2, 3),
(3, 3, 3);
```

# 4. Запросы к базе данных

1. Запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена:
```
SELECT students.s_id, students.name
FROM students 
LEFT JOIN exams ON students.s_id = exams.s_id
WHERE exams.s_id IS NULL;
```
или 
```
SELECT students.s_id, students.name
FROM Students
WHERE NOT EXISTS (
    SELECT 1
    FROM Exams
    WHERE Exams.s_id = Students.s_id
);
```

2. Запрос, который возвращает список студентов и количество сданных ими экзаменов. Только для студентов, у которых есть сданные экзамены:
```
SELECT st.s_id, st.name, COUNT(e.s_id) AS exams_count
FROM students st
JOIN exams e ON st.s_id = e.s_id
GROUP BY st.s_id, st.name
ORDER BY st.s_id;
```

3. Запрос для получения списка курсов со средним баллом по экзамену, отсортированный по убыванию среднего балла:
```
SELECT courses.c_no, courses.title, ROUND(AVG(exams.score), 2) AS average_score
FROM courses
JOIN exams ON courses.c_no = exams.c_no
GROUP BY courses.c_no, courses.title
ORDER BY average_score DESC;
```

# 5: Генерация данных
#### Напишем скрипт для генерации случайных данных с помощью Python:

Работаем в VSCode 
* Установиv расширение Python для VSCode
* Установим библиотеки psycopg2 и Faker
```
pip install psycopg2
pip install Faker
```

```python

import psycopg2
import random
from faker import Faker

# Подключение к базе данных PostgreSQL
conn = psycopg2.connect(
    dbname="academy",
    user="postgres",
    password="mysecretpassword",
    host="localhost",
    port="5432"
)
cur = conn.cursor()

# Создание объекта Faker для генерации случайных данных
fake = Faker()

# Функция для создания таблиц, если они еще не существуют


def create_tables():
    cur.execute("""
        CREATE TABLE IF NOT EXISTS Students (
            s_id SERIAL PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            start_year DATE NOT NULL
        )
    """)
    cur.execute("""
        CREATE TABLE IF NOT EXISTS Courses (
            c_no SERIAL PRIMARY KEY,
            title VARCHAR(100) NOT NULL,
            hours INTEGER NOT NULL
        )
    """)
    cur.execute("""
        CREATE TABLE IF NOT EXISTS Exams (
            s_id INTEGER NOT NULL,
            c_no INTEGER NOT NULL,
            score INTEGER CHECK (score >= 1 AND score <= 5),
            FOREIGN KEY (s_id) REFERENCES Students(s_id),
            FOREIGN KEY (c_no) REFERENCES Courses(c_no),
            PRIMARY KEY (s_id, c_no)
        )
    """)
    conn.commit()

# Функция для генерации и наполнения таблицы Students


def generate_students():
    cur.execute("SELECT COUNT(*) FROM Students")
    count = cur.fetchone()[0]
    if count < 10:  # Генерируем только если записей меньше 10
        for _ in range(10 - count):
            name = fake.name()
            start_year = fake.date_between(start_date='-5y', end_date='today')
            cur.execute(
                "INSERT INTO Students (name, start_year) VALUES (%s, %s)", (name, start_year))
        conn.commit()

# Функция для генерации и наполнения таблицы Courses


def generate_courses():
    cur.execute("SELECT COUNT(*) FROM Courses")
    count = cur.fetchone()[0]
    if count < 4:  # Генерируем только если записей меньше 4
        courses = [('Математика',), ('Физика',), ('История',), ('Литература',)]
        for course in courses[count:]:
            hours = random.randint(30, 90)
            cur.execute(
                "INSERT INTO Courses (title, hours) VALUES (%s, %s)", (course[0], hours))
        conn.commit()

# Функция для генерации и наполнения таблицы Exams


def generate_exams():
    cur.execute("SELECT COUNT(*) FROM Exams")
    count = cur.fetchone()[0]
    if count == 0:  # Генерируем только если таблица пуста
        cur.execute(
            "INSERT INTO Exams (s_id, c_no, score) SELECT s_id, c_no, %s FROM Students CROSS JOIN Courses", (random.randint(2, 5),))
        conn.commit()


# Вызываем функцию для создания таблиц
create_tables()

# Вызываем функции для генерации данных
generate_students()
generate_courses()
generate_exams()

# Закрытие соединения
cur.close()
conn.close()