
Проверка установки docker 
docker --version

Установка PostgreSQL через Docker
Шаг 1: Запуск контейнера PostgreSQL

docker run --name postgres-db -e POSTGRES_PASSWORD=goodluck777 -d postgres

Шаг 2: Проверка работы PostgreSQL
docker ps -a

Шаг 1: Установка PostgreSQL через Docker
1. Установите Docker, если он еще не установлен. Инструкции можно найти на официальном сайте Docker.
2. Создайте и запустите контейнер PostgreSQL с помощью следующих команд:
bash
Copy code
# Создаем и запускаем контейнер PostgreSQL
docker run --name academy-db -e POSTGRES_PASSWORD=mysecretpassword -d postgres
3. Подключитесь к контейнеру PostgreSQL:
bash
Copy code
# Подключаемся к контейнеру PostgreSQL
docker exec -it academy-db psql -U postgres
Шаг 2: Создание базы данных и таблиц
Создайте базу данных academy:
sql
Copy code
-- Создаем базу данных academy
CREATE DATABASE academy;
Подключитесь к базе данных academy:
bash
Copy code
# Подключаемся к базе данных academy
\c academy
Создайте таблицы по приложенной схеме. Допустим, схема выглядит следующим образом:
Таблица students (id, name)
Таблица courses (id, name)
Таблица exams (id, course_id, student_id, grade)
sql
Copy code
-- Скрипт для создания таблиц
CREATE TABLE Students (
    s_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
	start_year DATE NOT NULL
);

CREATE TABLE Courses (
    c_no SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
	hours TIME NOT NULL
);

CREATE TABLE Exams (
	PRIMARY KEY (s_id, c_no),
	s_id INT NOT NULL,
    c_no INT NOT NULL,
    score INT CHECK (score >= 1 AND score <= 5),
    FOREIGN KEY (c_no) REFERENCES Courses(c_no),
    FOREIGN KEY (s_id) REFERENCES students(s_id)
);

Шаг 3: Вставка данных в таблицы
sql
Copy code
-- Вставка данных в таблицы
INSERT INTO Students (name) VALUES ('Alice'), ('Bob'), ('Charlie');

INSERT INTO courses (name) VALUES ('Math'), ('Physics'), ('Chemistry');

INSERT INTO exams (course_id, student_id, grade) VALUES
(1, 1, 85),
(1, 2, 90),
(2, 2, 80),
(3, 3, 70);
Шаг 4: Запросы к базе данных
1. Запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена:
sql
Copy code
-- Запрос для получения студентов, которые не сдали ни одного экзамена
SELECT s.id, s.name
FROM students s
LEFT JOIN exams e ON s.id = e.student_id
WHERE e.student_id IS NULL;
2. Запрос, который возвращает список студентов и количество сданных ими экзаменов. Только для студентов, у которых есть сданные экзамены:
sql
Copy code
-- Запрос для получения студентов и количества сданных ими экзаменов
SELECT s.id, s.name, COUNT(e.id) AS exams_count
FROM students s
JOIN exams e ON s.id = e.student_id
GROUP BY s.id, s.name
HAVING COUNT(e.id) > 0;
3. Запрос для получения списка курсов со средним баллом по экзамену, отсортированный по убыванию среднего балла:
sql
Copy code
-- Запрос для получения списка курсов со средним баллом по экзамену
SELECT c.id, c.name, AVG(e.grade) AS avg_grade
FROM courses c
JOIN exams e ON c.id = e.course_id
GROUP BY c.id, c.name
ORDER BY avg_grade DESC;
Шаг 5: Генерация данных
Скрипт для генерации данных с помощью Python:
python
Copy code
import psycopg2
from faker import Faker

# Настройки подключения
conn = psycopg2.connect(
    dbname="academy",
    user="postgres",
    password="mysecretpassword",
    host="localhost"
)

fake = Faker()

# Функция для генерации данных
def generate_data():
    cur = conn.cursor()

    # Вставка студентов
    for _ in range(100):
        cur.execute("INSERT INTO students (name) VALUES (%s)", (fake.name(),))

    # Вставка курсов
    courses = ['Math', 'Physics', 'Chemistry', 'Biology', 'History']
    for course in courses:
        cur.execute("INSERT INTO courses (name) VALUES (%s)", (course,))

    # Вставка экзаменов
    cur.execute("SELECT id FROM students")
    student_ids = [row[0] for row in cur.fetchall()]
    
    cur.execute("SELECT id FROM courses")
    course_ids = [row[0] for row in cur.fetchall()]

    for student_id in student_ids:
        for course_id in course_ids:
            cur.execute(
                "INSERT INTO exams (course_id, student_id, grade) VALUES (%s, %s, %s)",
                (course_id, student_id, fake.random_int(min=0, max=100))
            )

    conn.commit()
    cur.close()

generate_data()
conn.close()
Запустите этот скрипт для заполнения таблиц случайными данными.
Таким образом, вы установили PostgreSQL через Docker, создали базу данных и таблицы, заполнили их данными и написали необходимые запросы.




