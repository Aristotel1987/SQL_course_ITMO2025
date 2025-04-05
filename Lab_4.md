# Лабораторная работа № 4 # 

**Студент:** Выдумкин Илья Александрович 
**группа:** P 4150  
**дисциплина:** "Взаимодействие с базами данных"
**Дата:** 05.04.2025  
** Задание:
Лабораторная работа 4.
Для выполнения лабораторной работы необходимо:
·  Описать бизнес-правила вашей предметной области. Какие в вашей системе могут быть действия, требующие выполнения запроса в БД. Эти бизнес-правила будут использованы для реализации триггеров, функций, процедур, транзакций поэтому приниматься будут только достаточно интересные бизнес-правила
·  Добавить в ранее созданную базу данных триггеры для обеспечения комплексных ограничений целостности. Триггеров должно быть не менее трех
·  Реализовать функции и процедуры на основе описания бизнес-процессов, определенных при описании предметной области из пункта 1. Примеров не менее 3
ы·  Привести 3 примера выполнения транзакции. Это может быть, например, проверка корректности вводимых данных для созданных функций и процедур. Например, функция, которая вносит данные. Данные проверяются и в случае если они не подходят ограничениям целостности, транзакция должна откатываться
·  Необходимо произвести анализ использования созданной базы данных, выявить наиболее часто используемые объекты базы данных, виды запросов к ним. Результаты должны быть представлены в виде текстового описания
·  На основании полученного описания требуется создать подходящие индексы и доказать, что они будут полезны для представленных в описании случаев использования базы данных.

Отчёт по лабораторной работе должен содержать:
·  титульный лист;
·  текст задания;
·  код триггеров, функций, процедур, транзакций;
·  описание наиболее часто используемых сценариев при работе с базой данных;
·  описание индексов и обоснование их использования;

Защита лабораторной работы будет происходить следующим способом. Студент описывает бизнес-правила предметной области и функции своей системы. При этом демонстрирует реализацию этих условий в системе средствами СУБД. Например: «в моем бизнес процессе при добавлении пользователя в систему ему начисляются премиальные баллы. Для этого я организовал триггер, вот его код. При этом я написал функцию, которая проверяет, что клиент не в черном списке. Добавление клиента и работу триггера я обернул в транзакцию, поэтому если клиент в черном списке, то результат откатывается и возвращается сообщение о том, что пользователь в ЧС». И так по всем пунктам ЛР. То есть рассказывает о проделанной работе, что реализовал и почему. 

По ходу защиты могут быть заданы теоретические вопросы по следующим темам:
1.  Процедуры, функции. Отличия. Где применяются одни, где другие.
2.  Триггеры. Как работают, зачем нужны. На что способны. Как понять, что они нужны.
3.  Индексы. Виды индексов по структуре данных. Преимущество и недостатки индексов. Виды индексов: кластеризованный, некластеризованный, уникальный, покрывающий, композитный, полнотекстовый. Сколько весит индекс. Как хранятся в памяти физически и что из себя представляют логически?
4.  Транзакции. Как понять, что их нужно применять. Когда они нужны. Принцип ACID. Какую функциональность имеют транзакции.
## Бизнес-правила ##
В ходе выполнения Лабораторной работы по предметной области из предыдущих ЛР было решено реализовать следующие бизнес-правила:
1. Уведомление о начале отопительного сезона. При вводе новых наблюдений в таблицу наблюдений, база данных автоматически сообщает о том, что пора включать отопление.
2. При добавлении нового помещения для того же пользователя, пользователю выдается промокод на третье бесплатное помещение, для подключения к системе. То есть действуует акция 3 по цене 2х.
3. При добавлении наблюдений, при окончании отопительного сезона, производится расчет потенциальной экономии за прошедший отопительный сезон
Для реализации этих правил понадобилось создать 2 дополнителные таблицы: Таблица промо-кодов со сроком истечения действия 90 дней от текущей даты и таблица отопительных сезонов, с внешним ключом в качестве id помещения, датой начала отопительного сезона, датой окончания сезона и потенциальной экономией, рассчиттанной по базовому варианту общего снижения внутренней температуры до 20 градусов.
CREATE TABLE HeatingSeason (
    id SERIAL PRIMARY KEY,  
    room_id INT NOT NULL,   -- Идентификатор комнаты
    heating_start_date DATE NOT NULL,  -- Дата включения отопления
    heating_end_date DATE,  -- Дата окончания отопления
    Potential_economy20 FLOAT,  -- потенциальная экономия, при постоянном снижении температуры до 20 градцусов, в процентах от общих расходов
    FOREIGN KEY (room_id) REFERENCES Room(room_id)  -- Внешний ключ на таблицу Room
);
CREATE TABLE PromoCodes (
    id SERIAL PRIMARY KEY, 
    user_login VARCHAR NOT NULL, 
    promo_code TEXT NOT NULL, -- Текст промокода
    expiration_date DATE NOT NULL, -- Дата истечения промокода
    FOREIGN KEY (user_login)
        REFERENCES USERS(login) 
        ON DELETE CASCADE -- Удаление записей в PromoCodes, если удален пользователь
);
## Транзакции, поддерживающие комплексную целостность БД ##
Для реализации комплексной целостности данных была применена обработка добавления новых датчиков совместно с помещением и квартирой, то есть пользователь не сможет создать датчик без привязки к помещению.
BEGIN; -- Начинаем транзакцию
-- Шаг 1: Вставка новых датчиков
INSERT INTO Sensors (sensor_type, installation_date)
VALUES 
('temp', CURRENT_DATE), 
('Hum', CURRENT_DATE);

-- шаг 2 добавление помещения с новыми датчиками
INSERT INTO Room (room_type, room_area, ceiling_height, glazing_area, external_wall_area, temperature_sensor_id, humidity_sensor_id, login)
VALUES 
('Living_Room', 25.0, 3.0, 10.0, 12.0, 
    (SELECT sensor_id FROM Sensors WHERE sensor_type = 'temp' ORDER BY installation_date DESC LIMIT 1),
    (SELECT sensor_id FROM Sensors WHERE sensor_type = 'Hum' ORDER BY installation_date DESC LIMIT 1),
    'user1'
);

-- Шаг 3: Вставка квартиры с этим помещением
INSERT INTO Flat (flat_number, login, room_id, latitude, longitude)
VALUES 
(11, 'user1', 
    (SELECT room_id FROM Room 
    WHERE temperature_sensor_id = (SELECT sensor_id FROM Sensors WHERE sensor_type = 'temp' ORDER BY installation_date DESC LIMIT 1)
    AND humidity_sensor_id = (SELECT sensor_id FROM Sensors WHERE sensor_type = 'Hum' ORDER BY installation_date DESC LIMIT 1)
    ORDER BY room_id DESC LIMIT 1), 
    40.7128, 
    -74.0060
);
COMMIT;
Вторая транзакция предполагает вставку данных в таблицу наблюдений с поиском идентификаторов датчиков в таблице Room
BEGIN; 
INSERT INTO Observations (room_id, observation_time, temperature_sensor_id, temperature_reading, humidity_sensor_id, humidity_reading, outside_temperature)
SELECT 
    r.room_id,              -- ID комнаты
    NOW(),                  -- Текущее время
    r.temperature_sensor_id, -- ID датчика температуры из таблицы Room
    23,                     -- Внутренняя температура
    r.humidity_sensor_id,    -- ID датчика влажности из таблицы Room
    15,                   -- значение влажности, 
    10                      -- Внешняя температура
FROM 
    Room r
WHERE 
    r.room_id = 1;         
COMMIT;
Третья транзакция представляет собой обратную операцию к первой, представим, что пользователь подал заявление с требованием удалить всю информацию о нем.  Запишем все связанные удаления в одну транзакцию, для придания атамарности.
BEGIN;

-- Сначала удаляем все наблюдения, связанные с комнатами пользователя
DELETE FROM Observations 
WHERE room_id IN (SELECT room_id FROM Room WHERE login = 'user2');
-- Удаляем все записи из таблицы HeatingSeason, связанные с комнатами пользователя
DELETE FROM HeatingSeason 
WHERE room_id IN (SELECT room_id FROM Room WHERE login = 'user2');
-- Удаляем комнаты, связанные с пользователем
DELETE FROM Room 
WHERE login = 'user2';
-- Удаляем датчики, которые были установлены в удаленных комнатах
DELETE FROM Sensors 
WHERE sensor_id IN (SELECT temperature_sensor_id FROM Room WHERE login = 'user2' 
                    UNION 
                    SELECT humidity_sensor_id FROM Room WHERE login = 'user2');
-- Удаляем квартиры пользователя
DELETE FROM Flat 
WHERE login = 'user2';
-- Наконец, удаляем самого пользователя
DELETE FROM USERS 
WHERE login = 'user2';
COMMIT;

## Функции и триггеры, обеспечивающие выполнение Бизнес-задач ##
### Бизнес-требование 2: создание промокодов ###
Первая и третье бизнес-требование связаны с расчетами средних температур, теплопотерь и возможной экономии, потому начнем наше описание со второго бизнес-требования. 
Сначала создадим триггерную функцию, которая, при добавлении новой записи в таблицу Room, будет проверять количество уже имеющихся комнат у пользователя и, если оно кратно 2, добавлять промокод таблицу Promocodes :
create FUNCTION add_promo_code_trigger()
RETURNS TRIGGER AS
$$
DECLARE
    room_count INT;
BEGIN
    SELECT COUNT(*) INTO room_count
    FROM Room
    WHERE login = NEW.login;

    IF room_count % 2 = 0 THEN
        INSERT INTO Promocodes (user_login, promo_code, expiration_date)
        VALUES (NEW.login, 'PROMO2025', CURRENT_DATE + INTERVAL '90 days');
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
Теперь создадим сам триггер на добавление помещения в таблицу Room, нам нужен тип AFTER,  так как считаем число комнат с учетом свежедобавленной.
CREATE TRIGGER after_room_insert
AFTER INSERT ON Room
FOR EACH ROW
EXECUTE FUNCTION add_promo_code_trigger();
Проверим работу триггера, выполнив транзакцию 1 из предыдущего раздела. Теперь при выборке таблицы промокодов видим:
home=# select * from Promocodes ;
 id | user_login | promo_code | expiration_date
----+------------+------------+-----------------
  2 | user1      | PROMO2025  | 2025-07-02
(1 ёЄЁюър)
### Бизнес-требование 1: Уведомление о начале отопительного сезона ###
В соответствии с действующими нормами, отопительный сезон начинается, когда среднесуточная температура за последние 5 суток опускается ниже 8 градусов Цельсия.Таким образом, для анализа необходимости включения отопления, нужно создать триггер на добавление наблюдений в таблицу Observations. Так как предполагается, что пользователь может менять частоту сбора информации, а также для более корректного усреднения температур, наложим условие, чтобы это наблюдения по данному помещению было первым в текущих сутках, то есть его дата не совпадала с предыдущим наблюдением. Так как необходимо выводить сообщение пользователю, а триггерная функция может возвращать объект только типа tRIGGER, напишем отдельную функцию, которая будет считать среднесуточную температуру за последние 5 суток для сегодня и вчера, сравнивать их и при переходе, через 8 градусов,  выдавать сообщение и записывать начало отопительного сезона в таблицу 
CREATE FUNCTION check_heating_status(v_room_id INT)
RETURNS TEXT AS $$
DECLARE
    avg_temp_today FLOAT;
    avg_temp_yesterday FLOAT;
    current_date DATE := CURRENT_DATE;
BEGIN
    -- Вычисляем среднесуточную температуру за последние 5 суток (включая сегодняшний день)
    SELECT AVG(o.outside_temperature) INTO avg_temp_today
    FROM Observations o
    WHERE o.room_id = v_room_id
    AND o.observation_time >= current_date - INTERVAL '6 days';

    -- Вычисляем среднесуточную температуру за предыдущие 5 суток (общая дата вчерашнего дня)
    SELECT AVG(o.outside_temperature) INTO avg_temp_yesterday
    FROM Observations o
    WHERE o.room_id = v_room_id
    AND o.observation_time >= current_date - INTERVAL '7 days'
    AND o.observation_time < current_date- INTERVAL '1 days';

    -- Проверяем, получены ли значения средней температуры
    IF avg_temp_today IS NULL THEN
        RETURN NULL;  -- Убираем возврат, если сегодня нет данных
    END IF;

    -- Сравниваем среднесуточные температуры
    IF (avg_temp_today < 8 AND avg_temp_yesterday < 8) OR (avg_temp_today >= 8 AND avg_temp_yesterday >= 8) THEN
        RETURN NULL;  -- Убираем возврат, если температуры с одной стороны от 8 градусов
    END IF;

    -- Возвращаем сообщения на основе температуры
    IF avg_temp_today < 8 AND avg_temp_yesterday >= 8 THEN
        -- Рекомендуем включить отопление
        INSERT INTO HeatingSeason (room_id, heating_start_date, heating_end_date, Potential_economy20 ) 
        VALUES (v_room_id, current_date, NULL, NULL);

        RETURN 'Рекомендуется включить отопление.';
        
    ELSIF avg_temp_today >= 8 AND avg_temp_yesterday < 8 THEN
        -- Рекомендуем выключить отопление, обновляем запись
        UPDATE HeatingSeason 
        SET heating_end_date = current_date 
        WHERE room_id = v_room_id AND heating_end_date IS NULL;

        RETURN 'Рекомендуется выключить отопление.';
        
    ELSE
        -- Если значение avg_temp_yesterday отсутствует, ничего не возвращаем
        RETURN NULL;
    END IF;

END;
$$ LANGUAGE plpgsql;
Теперь создаем триггерную функцию,  вводя условие на первое наблюдение в сутках:
CREATE FUNCTION before_insert_observation()
RETURNS TRIGGER AS
$$
DECLARE
    last_observation_date TIMESTAMP;
BEGIN
    -- Получаем дату последней записи для данной комнаты
    SELECT MAX(observation_time) INTO last_observation_date
    FROM Observations
    WHERE room_id = NEW.room_id;
    -- Проверяем, совпадает ли дата последней записи с текущей
    IF last_observation_date IS NULL OR DATE(last_observation_date) <> CURRENT_DATE THEN
        -- Вызываем функцию check_heating_status для данной комнаты
        PERFORM check_heating_status(NEW.room_id);
    END IF;

    RETURN NEW;  -- Возвращаем новую запись
END;
$$ LANGUAGE plpgsql;

-- Создаем триггер
CREATE TRIGGER trg_before_insert_observation
BEFORE INSERT ON Observations
FOR EACH ROW
EXECUTE FUNCTION before_insert_observation()

Для проверки работы функции, ббыли загружены соответствующие данные в таблицу наблюдений. После чего, триггер сработал и сделал запись в таблицу отопительных сезонов:
 id | room_id | heating_start_date | heating_end_date | potential_economy20
----+---------+--------------------+------------------+---------------------
  1 |       1 | 2025-04-03         |                  |
(1 ёЄЁюър)

### Бизнес-задача 3. Расчет экономии за отопительный сезон ###
Для расчета потенциальной экономии, при снижении температуры до 20 градусов, также как и в предыдущем случае, напишем отдельную функцию, которая будет вызываться из триггерной функции, триггер привяжем на обновление атрибута  heating_end_date
CREATE FUNCTION calculate_energy_savings(
    p_room_id INT,
    p_heat_start DATE,
    p_heat_end DATE
)
RETURNS VOID AS $$
DECLARE
    v_sum_temp_above_20 FLOAT := 0; -- Для суммы температур свыше 20
    v_sum_temp_difference FLOAT := 0; -- Для суммы разностей температур
    v_energy_savings FLOAT := 0; -- Для хранения результата расчета
BEGIN
    -- Вычисляем сумму температур внутри помещения выше 20 градусов
    SELECT SUM(CASE WHEN temperature_reading > 20 THEN (temperature_reading - 20) ELSE 0 END) 
    INTO v_sum_temp_above_20
    FROM Observations
    WHERE room_id = p_room_id
      AND observation_time BETWEEN p_heat_start AND p_heat_end;

    -- Вычисляем сумму разностей внутренней и наружной температур
    SELECT SUM(temperature_reading - outside_temperature) 
    INTO v_sum_temp_difference
    FROM Observations
    WHERE room_id = p_room_id
      AND observation_time BETWEEN p_heat_start AND p_heat_end;

    -- Расчет экономии, чтобы избежать деления на ноль
    IF v_sum_temp_difference > 0 THEN
        v_energy_savings := v_sum_temp_above_20 / v_sum_temp_difference;
    ELSE
        v_energy_savings := NULL; -- Если B = 0, экономию не можем рассчитать
    END IF;
    -- Обновляем запись в таблице HeatingSeason
    UPDATE HeatingSeason
    SET Potential_economy20 = v_energy_savings
    WHERE room_id = p_room_id
      AND heating_start_date = p_heat_start
      AND heating_end_date = p_heat_end;
END;
$$ LANGUAGE plpgsql;
Триггерная функция:
CREATE FUNCTION trigger_calculate_energy_savings()
RETURNS TRIGGER AS $$
BEGIN 
    PERFORM calculate_energy_savings(NEW.room_id, NEW.heating_start_date, NEW.heating_end_date);
    RETURN NEW;  
END;
$$ LANGUAGE plpgsql
Сам триггер:
CREATE TRIGGER update_energy_savings
AFTER UPDATE OF heating_end_date ON HeatingSeason
FOR EACH ROW 
EXECUTE FUNCTION trigger_calculate_energy_savings();
Для проверки функции переделаем наблюдения за прошлые несколько дней, чтобы окончание отопительного сезона произошло сегодня. Проверим расчет по таблице HeationgSeason
home=# select * from HeatingSeason;
 id | room_id | heating_start_date | heating_end_date | potential_economy20
----+---------+--------------------+------------------+---------------------
  1 |       1 | 2025-04-03         | 2025-04-05       | 0.16129032258064516
  
(1 ёЄЁюъш)
Таким образом потенциальная экономия за тестовый период в 3 дня, составила 16% от общих затрат на отопление, такой большой результат связан с тем, что на улице была относительно высокая температура(5 градусов), а значит снижение на 3 температуры градусавнутри помещения давало больший относительный вклад в снижение теплопотерь.

## Индексы ## 
Основными операциями в нашей системе являются:
1. Добавление наблюдений в таблицу наблюдений.
2. Запрос данных по конкретной комнате из таблицы наблюдений.
3. Сбор статистики по квартире.
В таблицах USERS и seensors поиск по первичному ключу вполне отвечает основным бизнес-целям. В таблице Room кроме использования первичных ключей, часто возникает необходимость поиска по логину пользователя, добавим индекс нна этот атрибут.
CREATE INDEX idx_login ON Room(login);
Для ускорения сбора информации о погоде при регулярных наблюдениях, добавим индекс в таблицу Flat на столбец room_id:
CREATE INDEX idx_room_id ON Flat(room_id);
Для быстрой аналитики по существующим наблюдениям добавим индекс на room_id в таблице Observations
CREATE INDEX idx_room_id_observ ON Observations(room_id);
Теперь рассмотрим пошагово, как внедрение индексов ускоряет стандартные операции:
1. Добавление наблюдений в таблицу наблюдений.
В таблице Room по room_id(первичный ключ), программа находит идентификаторы датчиков, чтобы запросить с них показания. В таблице Flat  программа по room_id находит координаты помещения, здесь, добавленный индекс ускоряет работу программы. 
2. Запрос данных по конкретной комнате из таблицы наблюдений.
Программа собирает данные по room_id в таблице Observations,  к которому мы добавили индекс, для дальнейшего анализа и отображения. 
3. Сбор статистики по квартире.
Когда пользователь хочет посмотреть статистику, он в первую очередь смотрит список своих помещений, для быстрого отображения этого списка, мы добавили индекс на login в таблице Room.