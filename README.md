# SQL_Murder_Mystery
##  Разгадка тайны убийства 

Вы выступаете в роли детектива. Вам поручили расследовать давнее нераскрытое дело об убийство. Полицейский отчёт о преступлении был похищен и вам необходимо вновь по крупицам собрать все факты. Единственно что вы знаете, что трагедия произошло 15 января 2018 года в SQL City. Ключи к разгадке преступления кроются в бесчисленных архивах полиции. У вас есть доступ к полицейской базе данных (https://mystery.knightlab.com). Воспользуйтесь SQL и найти убийцу. 

База данных полиции:

![1](https://github.com/Irina-Smol/SQL_Murder_Mystery/assets/112115002/87738d3b-252b-48c9-aa67-d352b8bbbbe1)

## Решение

### Prompt (Подсказка)

> Произошло преступление, и детективу нужна ваша помощь. Детектив дал вам протокол с места преступления, но вы его как-то потеряли. Вы смутно помните, что преступление было убийством, которое произошло где-то 15 января 2018 года и произошло в SQL City. Начните с извлечения соответствующего отчета о месте преступления из базы данных полицейского управления.

> A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

### Data Format Check (Проверка формата данных)
```SQL  
SELECT date FROM crime_scene_report limit 5;
```  
Данные хранятся в формате ГГГГММДД

### Crime scene report (Отчет о месте преступления)

Найти отчет о месте преступления, если известен тип (убийство) и город (SQL City). Предположим, что это было где-то в январе 2018 года

```SQL
SELECT * FROM crime_scene_report   
WHERE type = 'murder' AND city = 'SQL City' AND date between 20180101 and 20180131;
```  
> На кадрах с камер видеонаблюдения видно, что было 2 свидетеля. Первый свидетель живет в последнем доме на "Северо-Западном докторе". Второй свидетель по имени Аннабель живет где-то на Франклин-авеню.

> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

### Witness reports (Отчеты свидетелей)

Начнем со свидетеля, который живет на Нортвестерн Драйв. Мы знаем, что они живут в «последнем доме», который предположительно имеет самый большой номер дома на этой улице. Мы можем легко найти это, сначала отфильтровав только людей, которые живут на Северо-Западном проезде, затем упорядочив эти результаты по номеру дома в порядке убывания и показав только первый результат:

```SQL 
SELECT * FROM person 
WHERE address_street_name = 'Northwestern Dr'   
ORDER BY address_number DESC   
LIMIT 1;
```  
![2](https://github.com/Irina-Smol/SQL_Murder_Mystery/assets/112115002/0e118282-9bf2-4080-8f55-d2c4822c1392)

Найдем Аннабель

```SQL
SELECT * FROM person 
WHERE name like 'Annabel%'    
AND address_street_name = 'Franklin Ave';
```
![3](https://github.com/Irina-Smol/SQL_Murder_Mystery/assets/112115002/efd3ee3a-6c20-43d3-96d6-507cc55e03c2)

> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

Возьмем их интервью:
```SQL
SELECT person.name, interview.transcript 
FROM interview   
JOIN person ON person.id = interview.person_id    
WHERE person_id in (14887, 16371);
```

![4](https://github.com/Irina-Smol/SQL_Murder_Mystery/assets/112115002/9587f3ac-283c-4a35-96dd-01fc5c0b46aa)

> Морти - Я услышал выстрел, а затем увидел выбегающего человека. У него была сумка «Get Fit Now Gym». Членский номер на сумке начинался с «48Z». Эти сумки есть только у золотых участников. Мужчина сел в машину с номерным знаком «H42W».

> Аннабель - Я видела, как произошло убийство, и я узнал убийцу из своего спортзала, когда тренировался на прошлой неделе 9 января.

### Search in the gym (Поиск в спортзале)

Поскольку машина и сумка могут не принадлежать убийце, то лучший способ сузить круг вопросов — увидеть всех людей, которые пересеклись с Аннабель в спортзале 9 января 2018 года

```SQL
SELECT check_in_time, check_out_time  
FROM get_fit_now_check_in   
WHERE check_in_date = 20180109  AND membership_id = (   
    SELECT id  
    FROM get_fit_now_member    
    WHERE person_id = 16371);
```
| check_in_time|check_out_time | 
|-------------:|:--------------|
|          1600|          1700 |

