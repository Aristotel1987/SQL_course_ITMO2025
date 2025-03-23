 # Лабораторная работа № 2 # 

**Студент:** Выдумкин Илья Александрович 
**группа:** P 4150  
**дисциплина:** "Взаимодействие с базами данных"
**Дата:** 20.03.2025  
** Задание:
Лабораторная работа 2. 
1.  Из описания предметной области, полученной в ходе выполнения ЛР 1, выделить сущности, их атрибуты и связи, отразить их в инфологической модели (она же концептуальная)
2.  Составить даталогическую (она же ER-диаграмма, она же диаграмма сущность-связь) модель. При описании типов данных для атрибутов должны использоваться типы из СУБД PostgreSQL.
3.  Реализовать даталогическую модель в PostgreSQL. При описании и реализации даталогической модели должны учитываться ограничения целостности, которые характерны для полученной предметной области
4.  Заполнить созданные таблицы тестовыми данными.
Для построения моделей можно использовать plantUML
Отчёт по лабораторной работе должен содержать:
·  Имя студента
·  группа
·  дата выполнения задания
·  Именование дисциплины
·  текст задания;
·  описание предметной области;
·  список сущностей и их классификацию (стержневая, ассоциация, характеристика);
·  инфологическая модель (ER-диаграмма в расширенном виде - с атрибутами, ключами...);
·  даталогическая модель (должна содержать типы атрибутов, вспомогательные таблицы для отображения связей "многие-ко-многим");
·  реализация даталогической модели на SQL;
·  выводы по работе;
# Описание предметной области #
Система мониторинга и расчета показателей микроклимата состоит из датчиков 2х типов( температуры и влажности) и внешних источников погоды. Каждый датчик принадлежит определенному помещени и характеризуется id. Помещения характеризуются: площадью, высотой потолков, площадью остекления и внешних стен. Пользователи обладают набором помещений, расположенном в определенном географическом месте, характеризуемом координатами широты и долготы. Модули расчета производят регулярный мониторинг микроклимата, собирая данные с датчиков и запрашивая данные о погоде из внешних источников. Мониторинг осуществляется каждые 4 часа. Каждая ссессия сбора информации о микроклимате, собирается в таблицу наблюдений по конкретному пользователю.  Каждый пользователь характеризуется именем, фамилией, логином в системе и паролем.

# Список сущностей и их классификация #
1.**Помещение** ( стержневая сущность)
    - ID помещения(INTEGER, PRIMARY KEY)
    - тип помещения (VARCHAR)
    - площадь помещения(FLOAT)
    - высота потолков (FLOAT)
    - площадь остекления (FLOAT)
    - площадь внешних стен(FLOAT)
 - id  датчика температуры(INTEGER, FOREIGN KEY)
    - id  датчика влажности(INTEGER, FOREIGN KEY)
    -логин пользователя (INTEGER, FOREIGN KEY)
2. **Пользователь** ( стержневая сущность)
    - Фамилия (VARCHAR)
   - Имя (VARCHAR)
   - Контактная информация (VARCHAR)
   - Дата регистрации (DATE)
   - Логин в системе (VARCHAR, PRIMARY KEY), 
   - Пароль в системе (VARCHAR)
3. **Квартира** (ассоциативная сущность)
    - id  квартиры(INTEGER)
    - логин пользователя(VARCHAR, FOREIGN KEY)
    - ID помещения(INTEGER)
    - географическая широта(FLOAT)
    - Географическая долгота(FLOAT)
4.**. Датчики** ( Стержневая сущность)
    - ID датчика (INTEGER, PRIMARY KEY)
    - тип датчика (VARCHAR)
    - Дата установки (DATE)
5. Наблюдения (Стержневая сущность)
    -id наблюдения (INTEGER, PRIMARY KEY)
    - id  помещения(INTEGER, FOREIGN KEY)
    - время наблюдения (DATETIME)
    - показания датчика температуры (FLOAT)
    - показания датчика влажности (FLOAT)
    - температура за окном (FLOAT)

# Концептуальная модель #
@startuml
entity Помещение {
}
entity Пользователь {
}
entity Квартира{
}
entity Датчик{
}
entity Наблюдение{
}
Помещение --o{ Датчики : "имеет"
Пользователь --o{ Квартира : "владеет"
Помещение --o{ Наблюдения : "содержит"
Датчики --|> Наблюдения : "предоставляет"
@enduml

# Инфологическая модель #

@startuml
entity "Помещение" {
  + ID помещения : INTEGER <<PK>>
  + тип помещения : VARCHAR
  + площадь помещения : FLOAT
  + высота потолков : FLOAT
  + площадь остекления : FLOAT
  + площадь внешних стен : FLOAT
  + id датчика температуры : INTEGER <<FK>>
  + id датчика влажности : INTEGER <<FK>>
}

entity "Пользователь" {
  + Фамилия : VARCHAR
  + Имя : VARCHAR
  + Контактная информация : VARCHAR
  + Дата регистрации : DATE
  + Логин в системе : VARCHAR <<PK>>
  + Пароль в системе : VARCHAR
}

entity "Квартира" {
  + id квартиры : INTEGER
  + логин пользователя : VARCHAR <<FK>>
  + географическая широта : FLOAT
  + географическая долгота : FLOAT
}

entity "Датчики" {
  + ID датчика : INTEGER <<PK>>
  + тип датчика : VARCHAR
  + Дата установки : DATE
}

entity "Наблюдения" {
  + id наблюдения : INTEGER <<PK>>
  + id помещения : INTEGER <<FK>>
  + время наблюдения : DATETIME
  + показания датчика температуры : FLOAT
  + показания датчика влажности : FLOAT
  + температура за окном : FLOAT
}

Помещение "1" -- "0..*" Датчики : "имеет"
Пользователь "1" -- "0..*" Квартира : "владеет"
Помещение "1" -- "0..*" Наблюдения : "содержит"
Датчики "0..*" -- "0..*" Наблюдения : "предоставляет"
@enduml

# Реализация на языке SQL #
CREATE DATABASE Teplotech;

\c Teplotech



CREATE TABLE User (login VARCHAR PRIMARY KEY, last_name VARCHAR NOT NULL, first_name VARCHAR NOT NULL, contact_info VARCHAR, registration_date DATE NOT NULL, password VARCHAR NOT NULL CHECK (length(password) >= 6));

CREATE TABLE Flat (flat_id SERIAL PRIMARY KEY, flat_number INTEGER NOT NULL, login VARCHAR NOT NULL, room_id INTEGER NOT NULL, latitude FLOAT CHECK (latitude >= -90 AND latitude <= 90), longitude FLOAT CHECK (longitude >= -180 AND longitude <= 180), UNIQUE (flat_number, login), FOREIGN KEY (login) REFERENCES User(login), FOREIGN KEY (room_id) REFERENCES Room(room_id));

CREATE TABLE Sensors (sensor_id SERIAL PRIMARY KEY, sensor_type VARCHAR NOT NULL CHECK (sensor_type IN ('temp', 'Hum')), installation_date DATE NOT NULL);

CREATE TABLE Room (room_id SERIAL PRIMARY KEY, room_type VARCHAR NOT NULL CHECK (room_type IN ('Kitchen', 'Living_Room', 'Bath_Room', 'Bedroom')), room_area FLOAT NOT NULL CHECK (room_area > 0), ceiling_height FLOAT NOT NULL CHECK (ceiling_height > 0), glazing_area FLOAT NOT NULL CHECK (glazing_area >= 0), external_wall_area FLOAT NOT NULL CHECK (external_wall_area >= 0), temperature_sensor_id INTEGER, humidity_sensor_id INTEGER, login VARCHAR NOT NULL, FOREIGN KEY (temperature_sensor_id) REFERENCES Sensors(sensor_id), FOREIGN KEY (humidity_sensor_id) REFERENCES Sensors(sensor_id , FOREIGN KEY (login) REFERENCES User(login));

CREATE TABLE Observations (observation_id SERIAL PRIMARY KEY, room_id INTEGER NOT NULL, observation_time TIMESTAMP NOT NULL, temperature_sensor_id INTEGER NOT NULL, temperature_reading FLOAT NOT NULL, humidity_sensor_id INTEGER NOT NULL, humidity_reading FLOAT NOT NULL, outside_temperature FLOAT, FOREIGN KEY (room_id) REFERENCES Room(room_id), FOREIGN KEY (temperature_sensor_id) REFERENCES Sensors(sensor_id), FOREIGN KEY (humidity_sensor_id) REFERENCES Sensors(sensor_id));

INSERT INTO USERS (login, last_name, first_name, contact_info, registration_date, password) VALUES ('user1', 'Smith', 'John', 'john.smith@example.com', '2023-01-01', 'password1'), ('user2', 'Doe', 'Jane', 'jane.doe@example.com', '2023-01-02', 'password2'), ('user3', 'Johnson', 'Emily', 'emily.johnson@example.com', '2023-01-03', 'password3');

INSERT INTO Flat (flat_number, login, room_id, latitude, longitude) VALUES (101, 'user1', 1, 40.7128, -74.0060), (102, 'user2', 2, 34.0522, -118.2437), (103, 'user3', 3, 51.5074, -0.1278);

INSERT INTO Sensors (sensor_type, installation_date) VALUES ('temp', '2023-01-01'), ('Hum', '2023-01-02'), ('temp', '2023-01-03');

INSERT INTO Room (room_type, room_area, ceiling_height, glazing_area, external_wall_area, temperature_sensor_id, humidity_sensor_id, login) VALUES ('Kitchen', 20.5, 2.5, 5.0, 15.0, 1, 2, 'user1'), ('Living_Room', 25.0, 2.8, 10.0, 20.0, 2, 3, 'user2'), ('Bath_Room', 10.0, 2.4, 2.0, 7.0, NULL, 3, 'user3');
INSERT INTO Flat (flat_number, login, room_id, latitude, longitude) VALUES (101, 'user1', 1, 40.7128, -74.0060), (102, 'user2', 2, 34.0522, -118.2437), (103, 'user3', 3, 51.5074, -0.1278);

INSERT INTO Observations (room_id, observation_time, temperature_sensor_id, temperature_reading, humidity_sensor_id, humidity_reading, outside_temperature) VALUES (1, '2023-01-10 10:00:00', 1, 22.5, 2, 55.0, 5.0), (2, '2023-01-11 11:00:00', 2, 21.0, 3, 60.0, 6.0), (3, '2023-01-12 12:00:00', 1, 23.0, 2, 58.0, 4.5);
# Проверка таблиц запросами SELECT #
SELECT * FROM USERS;
 user_login | last_name | first_name |      contact_info       | registration_date |   password    
------------+-----------+------------+-------------------------+-------------------+---------------
 user1      | Ivanov    | Ivan       | ivanov@example.com      | 2023-01-15        | password123
 user2      | Petrov    | Petr       | petrov@example.com      | 2023-02-20        | password456
 user3      | Sidorov   | Sidor      | sidorov@example.com     | 2023-03-10        | password789
 user4      | Smith     | John       | john.smith@example.com  | 2023-01-15        | password123
 user5      | Doe       | Jane       | jane.doe@example.com    | 2023-02-20        | secret456
 user6      | Brown     | Chris      | chris.brown@example.com | 2023-03-05        | mypassword789
(6 строк)
(END)

SELECT * FROMROOMS; 
 room_id |  room_type  | room_area | ceiling_height | glazing_area | external_walls_area | temperature_sensor_id | humidity_sensor_id | user_login 
---------+-------------+-----------+----------------+--------------+---------------------+-----------------------+--------------------+------------
       1 | Living room |      25.5 |            2.8 |          4.5 |                  10 |                     1 |                  2 | user1
       2 | Bedroom     |        15 |            2.5 |            3 |                   8 |                     1 |                  2 | user2
       3 | Kitchen     |        20 |            2.7 |          2.5 |                   9 |                     1 |                  2 | user3
(3 строки)

(END)

select * from flats;
 flat_id | flat_number | user_login | room_id | latitude | longitude 
---------+-------------+------------+---------+----------+-----------
       1 |         101 | user1      |       1 |  55.7558 |   37.6173
       2 |         102 | user2      |       2 |  55.7619 |   37.6173
       3 |         103 | user3      |       3 |   55.759 |   37.6193
(3 строки)

SELECT * FROM OBSERVATIONS;
 
observation_id | room_id |  observation_time   | temperature_reading | humidity_reading | outside_temperature 
----------------+---------+---------------------+---------------------+------------------+---------------------
              1 |       1 | 2023-09-01 12:00:00 |                22.5 |               45 |                  25
              2 |       2 | 2023-09-01 12:05:00 |                  20 |               50 |                  24
              3 |       3 | 2023-09-01 12:10:00 |                  21 |               55 |                  26
(3 строки)

(END)

select * from sensors;
 sensor_id |    sensor_type     | installation_date 
-----------+--------------------+-------------------
         1 | Temperature Sensor | 2023-01-01
         2 | Humidity Sensor    | 2023-02-01
         3 | Temperature Sensor | 2023-03-01
(3 строки)

# Выводы #
В ходе выполнения задания на основе описания предметной области была построена и реализована даталогическая модель  на языке SQL. В ходе реализации студент столкнулся с нарушением целостности - закольцеванием таблицы Помещений и таблицы Датчиков, взаимные ссылки по внешнним ключам не позволяли добавлять данные без нарушения целостности. В результате было принято решение исключить ссылку на id помещения в таблице Датчиков. Кроме того студент понимает, что структура может быть дополнительно доработана, в частности добавлены дополнительные ограничения на типы датчиков, ограничения значения температуры в таблице наблюдений. Необходимость существование таблицы квартир, остается под вопросом, первоначально она задумывалась, как небольшая таблица, которая агрегирует данные, необходимые для  регулярного программного сбора данных с датчиков внутри помещений и запроса фактической погоды по координатам. Однако, разное количество датчиков в квартирах, предполагало хранение списков ссылок на эти датчики, что не всегда удобно в обработке. Было принято решение просто делать по одной записи на одно помещение с указанием датчиков и координат. В целом такая реализация может быть реализована просто добавлением координат в таблицу помещений. Однако, анализ Концептуальной модели привел к тому, что возможна ситуация, когда у одного пользователя несколько квартир, и потому данная таблица вполне может иметь смысл в бизнес логике самого приложения. 