Создаем Структуру
```
CREATE TABLE public.user (
	id int,
	login varchar,
	name varchar,
	surname varchar,
	pos_id int
);
CREATE TABLE public.departments (
	dep_id int,
	dep_name varchar
);
CREATE TABLE public.positions (
	pos_id int,
	dep_id int,
	pos_name varchar
);
```
Заполняем
```
INSERT INTO public.user
VALUES (1, 'alex_p', 'Alex', 'Popov', 201);
INSERT INTO public.user
VALUES (2, 'aleksey_r', 'Aleksey', 'Remizov', 201);
INSERT INTO public.user
VALUES (3, 'dmitr_o', 'Dmitriy', 'Osin', 202);
INSERT INTO public.user
VALUES (4, 'oleg_g', 'Oleg', 'Groshev', 202);
INSERT INTO public.user
VALUES (5, 'mih_t', 'Mikhail', 'Kostikov', 203);
INSERT INTO public.user
VALUES (6, 'rita_l', 'Margorita', 'Lisova', 203);
INSERT INTO public.user
VALUES (7, 'jon_d', 'Evgeniy', 'Dolin', 204);
INSERT INTO public.user
VALUES (8, 'alexander_e', 'Alexander', 'Efimov', 204);
INSERT INTO public.user
VALUES (9, 'yar_b', 'Yaroslav', 'Belov', 205);
INSERT INTO public.user
VALUES (10, 'olga_y', 'Olga', 'Yalshova', 205);
INSERT INTO public.user
VALUES (11, 'tim_b', 'Timofey', 'Bolishov', 206);
INSERT INTO public.user
VALUES (12, 'kos_t', 'Konstantin', 'Tikhov', 206);
INSERT INTO public.user
VALUES (13, 'ant_p', 'Anton', 'Paverin', 207);
INSERT INTO public.user
VALUES (14, 'lex_a', 'Aleksey', 'Averin', 207);

INSERT INTO public.departments
VALUES (101, 'IT');
INSERT INTO public.departments
VALUES (121, 'Seles');
INSERT INTO public.departments
VALUES (150, 'Logistics');
INSERT INTO public.departments
VALUES (180, 'HR');

INSERT INTO public.positions
VALUES (201, 101, 'Sysadmin');
INSERT INTO public.positions
VALUES (202, 121, 'Manager_sales');
INSERT INTO public.positions
VALUES (203, 121, 'CreationDirector');
INSERT INTO public.positions
VALUES (204, 150, 'Driver');
INSERT INTO public.positions
VALUES (205, 150, 'Logist_manager');
INSERT INTO public.positions
VALUES (206, 180, 'Recruiter');
INSERT INTO public.positions
VALUES (207, 180, 'Buhgalter');
```

Реализовать прямое соединение двух или более таблиц
```
dbtest=# select pos_name,name,surname from public.user u inner join public.positions p on p.pos_id =u.pos_id;
     pos_name     |    name    | surname
------------------+------------+----------
 Sysadmin         | Aleksey    | Remizov
 Sysadmin         | Alex       | Popov
 Manager_sales    | Oleg       | Groshev
 Manager_sales    | Dmitriy    | Osin
 CreationDirector | Margorita  | Lisova
 CreationDirector | Mikhail    | Kostikov
 Driver           | Alexander  | Efimov
 Driver           | Evgeniy    | Dolin
 Logist_manager   | Olga       | Yalshova
 Logist_manager   | Yaroslav   | Belov
 Recruiter        | Konstantin | Tikhov
 Recruiter        | Timofey    | Bolishov
 Buhgalter        | Aleksey    | Averin
 Buhgalter        | Anton      | Paverin
(14 rows)

```
Реализовать левостороннее (или правостороннее) соединение двух или более таблиц
```
dbtest=# select pos_name,name,surname from public.user u right join public.positions p on p.pos_id =u.pos_id;
     pos_name     |    name    | surname
------------------+------------+----------
 Sysadmin         | Aleksey    | Remizov
 Sysadmin         | Alex       | Popov
 Manager_sales    | Oleg       | Groshev
 Manager_sales    | Dmitriy    | Osin
 CreationDirector | Margorita  | Lisova
 CreationDirector | Mikhail    | Kostikov
 Driver           | Alexander  | Efimov
 Driver           | Evgeniy    | Dolin
 Logist_manager   | Olga       | Yalshova
 Logist_manager   | Yaroslav   | Belov
 Recruiter        | Konstantin | Tikhov
 Recruiter        | Timofey    | Bolishov
 Buhgalter        | Aleksey    | Averin
 Buhgalter        | Anton      | Paverin
(14 rows)

таблицы простые то результат получается такой же
```
Реализовать кросс соединение двух или более таблиц
```
dbtest=# select * from public.user u cross join public.departments d;
 id |    login    |    name    | surname  | pos_id | dep_id | dep_name
----+-------------+------------+----------+--------+--------+-----------
  1 | alex_p      | Alex       | Popov    |    201 |    101 | IT
  2 | aleksey_r   | Aleksey    | Remizov  |    201 |    101 | IT
  3 | dmitr_o     | Dmitriy    | Osin     |    202 |    101 | IT
  4 | oleg_g      | Oleg       | Groshev  |    202 |    101 | IT
  5 | mih_t       | Mikhail    | Kostikov |    203 |    101 | IT
  6 | rita_l      | Margorita  | Lisova   |    203 |    101 | IT
  7 | jon_d       | Evgeniy    | Dolin    |    204 |    101 | IT
  8 | alexander_e | Alexander  | Efimov   |    204 |    101 | IT
  9 | yar_b       | Yaroslav   | Belov    |    205 |    101 | IT
 10 | olga_y      | Olga       | Yalshova |    205 |    101 | IT
 11 | tim_b       | Timofey    | Bolishov |    206 |    101 | IT
 12 | kos_t       | Konstantin | Tikhov   |    206 |    101 | IT
 13 | ant_p       | Anton      | Paverin  |    207 |    101 | IT
 14 | lex_a       | Aleksey    | Averin   |    207 |    101 | IT
  1 | alex_p      | Alex       | Popov    |    201 |    121 | Seles
  2 | aleksey_r   | Aleksey    | Remizov  |    201 |    121 | Seles
  3 | dmitr_o     | Dmitriy    | Osin     |    202 |    121 | Seles
  4 | oleg_g      | Oleg       | Groshev  |    202 |    121 | Seles
  5 | mih_t       | Mikhail    | Kostikov |    203 |    121 | Seles
  6 | rita_l      | Margorita  | Lisova   |    203 |    121 | Seles
  7 | jon_d       | Evgeniy    | Dolin    |    204 |    121 | Seles
  8 | alexander_e | Alexander  | Efimov   |    204 |    121 | Seles
  9 | yar_b       | Yaroslav   | Belov    |    205 |    121 | Seles
 10 | olga_y      | Olga       | Yalshova |    205 |    121 | Seles
 11 | tim_b       | Timofey    | Bolishov |    206 |    121 | Seles
 12 | kos_t       | Konstantin | Tikhov   |    206 |    121 | Seles
 13 | ant_p       | Anton      | Paverin  |    207 |    121 | Seles
 14 | lex_a       | Aleksey    | Averin   |    207 |    121 | Seles
  1 | alex_p      | Alex       | Popov    |    201 |    150 | Logistics
  2 | aleksey_r   | Aleksey    | Remizov  |    201 |    150 | Logistics
  3 | dmitr_o     | Dmitriy    | Osin     |    202 |    150 | Logistics
  4 | oleg_g      | Oleg       | Groshev  |    202 |    150 | Logistics
  5 | mih_t       | Mikhail    | Kostikov |    203 |    150 | Logistics
  6 | rita_l      | Margorita  | Lisova   |    203 |    150 | Logistics
  7 | jon_d       | Evgeniy    | Dolin    |    204 |    150 | Logistics
  8 | alexander_e | Alexander  | Efimov   |    204 |    150 | Logistics
  9 | yar_b       | Yaroslav   | Belov    |    205 |    150 | Logistics
 10 | olga_y      | Olga       | Yalshova |    205 |    150 | Logistics
 11 | tim_b       | Timofey    | Bolishov |    206 |    150 | Logistics
 12 | kos_t       | Konstantin | Tikhov   |    206 |    150 | Logistics
 13 | ant_p       | Anton      | Paverin  |    207 |    150 | Logistics
 14 | lex_a       | Aleksey    | Averin   |    207 |    150 | Logistics
  1 | alex_p      | Alex       | Popov    |    201 |    180 | HR
  2 | aleksey_r   | Aleksey    | Remizov  |    201 |    180 | HR
  3 | dmitr_o     | Dmitriy    | Osin     |    202 |    180 | HR
  4 | oleg_g      | Oleg       | Groshev  |    202 |    180 | HR
  5 | mih_t       | Mikhail    | Kostikov |    203 |    180 | HR
  6 | rita_l      | Margorita  | Lisova   |    203 |    180 | HR
  7 | jon_d       | Evgeniy    | Dolin    |    204 |    180 | HR
  8 | alexander_e | Alexander  | Efimov   |    204 |    180 | HR
  9 | yar_b       | Yaroslav   | Belov    |    205 |    180 | HR
 10 | olga_y      | Olga       | Yalshova |    205 |    180 | HR
 11 | tim_b       | Timofey    | Bolishov |    206 |    180 | HR
 12 | kos_t       | Konstantin | Tikhov   |    206 |    180 | HR
 13 | ant_p       | Anton      | Paverin  |    207 |    180 | HR
 14 | lex_a       | Aleksey    | Averin   |    207 |    180 | HR
(56 rows)

получаем все возможные комбинации из 2х таблиц
```
Реализовать полное соединение двух или более таблиц
```
dbtest=# select * from public.user u full join public.positions p on p.pos_id =u.pos_id;
 id |    login    |    name    | surname  | pos_id | pos_id | dep_id |     pos_name
----+-------------+------------+----------+--------+--------+--------+------------------
  2 | aleksey_r   | Aleksey    | Remizov  |    201 |    201 |    101 | Sysadmin
  1 | alex_p      | Alex       | Popov    |    201 |    201 |    101 | Sysadmin
  4 | oleg_g      | Oleg       | Groshev  |    202 |    202 |    121 | Manager_sales
  3 | dmitr_o     | Dmitriy    | Osin     |    202 |    202 |    121 | Manager_sales
  6 | rita_l      | Margorita  | Lisova   |    203 |    203 |    121 | CreationDirector
  5 | mih_t       | Mikhail    | Kostikov |    203 |    203 |    121 | CreationDirector
  8 | alexander_e | Alexander  | Efimov   |    204 |    204 |    150 | Driver
  7 | jon_d       | Evgeniy    | Dolin    |    204 |    204 |    150 | Driver
 10 | olga_y      | Olga       | Yalshova |    205 |    205 |    150 | Logist_manager
  9 | yar_b       | Yaroslav   | Belov    |    205 |    205 |    150 | Logist_manager
 12 | kos_t       | Konstantin | Tikhov   |    206 |    206 |    180 | Recruiter
 11 | tim_b       | Timofey    | Bolishov |    206 |    206 |    180 | Recruiter
 14 | lex_a       | Aleksey    | Averin   |    207 |    207 |    180 | Buhgalter
 13 | ant_p       | Anton      | Paverin  |    207 |    207 |    180 | Buhgalter
(14 rows)

результат не отличается от прямого, всему виной простая структура
```
Реализовать запрос, в котором будут использованы разные типы соединений
```
dbtest=# select pos_name,name,surname,d.* from public.user u inner join public.positions p on p.pos_id = u.pos_id cross join public.departments d;
     pos_name     |    name    | surname  | dep_id | dep_name
------------------+------------+----------+--------+-----------
 Sysadmin         | Aleksey    | Remizov  |    101 | IT
 Sysadmin         | Aleksey    | Remizov  |    121 | Seles
 Sysadmin         | Aleksey    | Remizov  |    150 | Logistics
 Sysadmin         | Aleksey    | Remizov  |    180 | HR
 Sysadmin         | Alex       | Popov    |    101 | IT
 Sysadmin         | Alex       | Popov    |    121 | Seles
 Sysadmin         | Alex       | Popov    |    150 | Logistics
 Sysadmin         | Alex       | Popov    |    180 | HR
 Manager_sales    | Oleg       | Groshev  |    101 | IT
 Manager_sales    | Oleg       | Groshev  |    121 | Seles
 Manager_sales    | Oleg       | Groshev  |    150 | Logistics
 Manager_sales    | Oleg       | Groshev  |    180 | HR
 Manager_sales    | Dmitriy    | Osin     |    101 | IT
 Manager_sales    | Dmitriy    | Osin     |    121 | Seles
 Manager_sales    | Dmitriy    | Osin     |    150 | Logistics
 Manager_sales    | Dmitriy    | Osin     |    180 | HR
 CreationDirector | Margorita  | Lisova   |    101 | IT
 CreationDirector | Margorita  | Lisova   |    121 | Seles
 CreationDirector | Margorita  | Lisova   |    150 | Logistics
 CreationDirector | Margorita  | Lisova   |    180 | HR
 CreationDirector | Mikhail    | Kostikov |    101 | IT
 CreationDirector | Mikhail    | Kostikov |    121 | Seles
 CreationDirector | Mikhail    | Kostikov |    150 | Logistics
 CreationDirector | Mikhail    | Kostikov |    180 | HR
 Driver           | Alexander  | Efimov   |    101 | IT
 Driver           | Alexander  | Efimov   |    121 | Seles
 Driver           | Alexander  | Efimov   |    150 | Logistics
 Driver           | Alexander  | Efimov   |    180 | HR
 Driver           | Evgeniy    | Dolin    |    101 | IT
 Driver           | Evgeniy    | Dolin    |    121 | Seles
 Driver           | Evgeniy    | Dolin    |    150 | Logistics
 Driver           | Evgeniy    | Dolin    |    180 | HR
 Logist_manager   | Olga       | Yalshova |    101 | IT
 Logist_manager   | Olga       | Yalshova |    121 | Seles
 Logist_manager   | Olga       | Yalshova |    150 | Logistics
 Logist_manager   | Olga       | Yalshova |    180 | HR
 Logist_manager   | Yaroslav   | Belov    |    101 | IT
 Logist_manager   | Yaroslav   | Belov    |    121 | Seles
 Logist_manager   | Yaroslav   | Belov    |    150 | Logistics
 Logist_manager   | Yaroslav   | Belov    |    180 | HR
 Recruiter        | Konstantin | Tikhov   |    101 | IT
 Recruiter        | Konstantin | Tikhov   |    121 | Seles
 Recruiter        | Konstantin | Tikhov   |    150 | Logistics
 Recruiter        | Konstantin | Tikhov   |    180 | HR
 Recruiter        | Timofey    | Bolishov |    101 | IT
 Recruiter        | Timofey    | Bolishov |    121 | Seles
 Recruiter        | Timofey    | Bolishov |    150 | Logistics
 Recruiter        | Timofey    | Bolishov |    180 | HR
 Buhgalter        | Aleksey    | Averin   |    101 | IT
 Buhgalter        | Aleksey    | Averin   |    121 | Seles
 Buhgalter        | Aleksey    | Averin   |    150 | Logistics
 Buhgalter        | Aleksey    | Averin   |    180 | HR
 Buhgalter        | Anton      | Paverin  |    101 | IT
 Buhgalter        | Anton      | Paverin  |    121 | Seles
 Buhgalter        | Anton      | Paverin  |    150 | Logistics
 Buhgalter        | Anton      | Paverin  |    180 | HR
(56 rows)

```
Сделать комментарии на каждый запрос
```
-выполнено
```
К работе приложить структуру таблиц, для которых выполнялись соединения
```
-в самом первом пункте
```
Что бы как то внести разнообразие ещё пример (вывод сотрудников и должностей по департаментам c сортировкой по фамилии)
```
dbtest=# SELECT u.surname,u.name,p.pos_name, d.dep_name FROM public.user u
JOIN public.positions p ON u.pos_id = p.pos_id
JOIN public.departments d ON p.dep_id = d.dep_id order by u.surname;
 surname  |    name    |     pos_name     | dep_name
----------+------------+------------------+-----------
 Averin   | Aleksey    | Buhgalter        | HR
 Belov    | Yaroslav   | Logist_manager   | Logistics
 Bolishov | Timofey    | Recruiter        | HR
 Dolin    | Evgeniy    | Driver           | Logistics
 Efimov   | Alexander  | Driver           | Logistics
 Groshev  | Oleg       | Manager_sales    | Seles
 Kostikov | Mikhail    | CreationDirector | Seles
 Lisova   | Margorita  | CreationDirector | Seles
 Osin     | Dmitriy    | Manager_sales    | Seles
 Paverin  | Anton      | Buhgalter        | HR
 Popov    | Alex       | Sysadmin         | IT
 Remizov  | Aleksey    | Sysadmin         | IT
 Tikhov   | Konstantin | Recruiter        | HR
 Yalshova | Olga       | Logist_manager   | Logistics
(14 rows)
```
