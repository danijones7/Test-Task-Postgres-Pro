# 1. Установка PostgreSQL через Docker

* Создать и запустить контейнер PostgreSQL с помощью следующих команд:

### Создаем и запускаем контейнер PostgreSQL
```
docker run --name academy-db -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```
### Подключаемся к контейнеру PostgreSQL
```
docker exec -it academy-db psql -U postgres
```
### Создаем базу данных academy
```
CREATE DATABASE academy;
```
### Подключитесь к базе данных academy:
```
\c academy
```
# 2. Скрипт для создания таблиц

```
CREATE TABLE students (
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
    FOREIGN KEY (s_id) REFERENCES students(s_id)
);
```
# 3: Вставка данных в таблицы

```
INSERT INTO Students (name, start_year) VALUES 
('John', '01.09.2000'), 
('Bill', '01.09.2000'), 
('Lev', '01.09.2000') ;
```
```
INSERT INTO Courses (title, hours) VALUES 
('Math', 40),
('Physics', 55), 
('Chemistry', 77);
```
```
INSERT INTO exams (course_id, student_id, score) VALUES
(1, 1, 85),
(1, 2, 90),
(2, 2, 80),
(3, 3, 70);
```

# 4. Запросы к базе данных

1. Запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена:
```
SELECT st.s_id, st.name
FROM students st
LEFT JOIN exams e ON st.s_id = e.s_id
WHERE e.s_id IS NULL;
```

2. Запрос, который возвращает список студентов и количество сданных ими экзаменов. Только для студентов, у которых есть сданные экзамены:
```
SELECT st.s_id, st.name, COUNT(e.s_id) AS exams_count
FROM students st
JOIN exams e ON st.s_id = e.s_id
GROUP BY st.s_id, st.name
HAVING COUNT(e.s_id) > 0;
```

3. Запрос для получения списка курсов со средним баллом по экзамену, отсортированный по убыванию среднего балла:
```
SELECT co.c_no, co.title, ROUND(AVG(e.score)::numeric, 2) AS avg_score
FROM courses co
JOIN exams e ON co.c_no = e.c_no
GROUP BY co.c_no, co.title
ORDER BY avg_score DESC;
```

# 5: Генерация данных
Скрипт для генерации данных с помощью Python:

import psycopg2
from faker import Faker

#### Настройки подключения
conn = psycopg2.connect(
    dbname="academy",
    user="postgres",
    password="mysecretpassword",
    host="localhost"
)

fake = Faker()

### Функция для генерации данных
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

