# sql_practice
practicing postgresql in Dbeaver

## Вопросы и ответы: 
1. Куда, в какие дни недели и за какое время можно долететь из Волгограда
2. Найдем ближайший рейс, вылетающий из Екатеринбурга в Москву
3. Найдите 10 бронирований с самой высокой стоимостью
4. В какие города летали пассажиры с самой высокой стоимостью бронирования?
5. Найдите самые популярные направления перелетов из Москвы?
6. В какие города наиболее дешевые перелеты?
7. Выведете список топ-10 городов по убыванию средней стоимости билета
8. Основные направления перелетов по месяцам
9. Назовите самое популярное место бронирования у пассажиров бизнес-класса

--------------

1.
```
select arrival_city , to_char(actual_departure, 'Day') as day, (actual_arrival - actual_departure) as time
from flights_v fv 
where departure_city  = 'Волгоград' and fv is not null;
```

2.
```
select actual_departure 
from flights_v fv 
where departure_city  = 'Екатеринбург' and arrival_city = 'Москва' and actual_departure is not null 
order  by actual_departure desc
limit;
```

3.
```
select book_ref, amount 
from ticket_flights tf 
join tickets t 
on tf.ticket_no = t.ticket_no 
order by amount desc 
limit 10;
```

4.
```
select city ->> lang()
from bookings b 
join tickets t 
on b.book_ref = t.book_ref 
join ticket_flights tf 
on t.ticket_no = tf.ticket_no 
join flights f 
on f.flight_id = tf.flight_id 
join airports_data ad 
on ad.airport_code = f.departure_airport 
order by total_amount desc;
```

5.
```
select arrival_city , count(arrival_city) as count 
from flights_v fv 
where departure_city = 'Москва'
group by arrival_city 
order by count(arrival_city) desc
limit 10;
```

6.
```
select city ->> lang() as city, amount
from ticket_flights tf 
join tickets t 
on tf.ticket_no = t.ticket_no 
join flights f 
on tf.flight_id = f.flight_id 
join airports_data ad
on ad.airport_code = f.arrival_airport 
group by city, amount 
order by amount, city asc;
```

7.
```
select city	->> lang() as city, AVG(amount) as avgc
from ticket_flights tf 
join flights f 
on f.flight_id = tf.flight_id 
join airports_data ad 
on ad.airport_code = f.arrival_airport 
group by city
order by avgc desc 
limit 10;
```

8.
```
select arrival_city , month, count
from(
		(SELECT rank() OVER (PARTITION BY month ORDER BY count DESC) AS therank, * 
			FROM (select arrival_city , to_char(actual_departure, 'Month') as month, count(to_char(actual_departure, 'Month')) as count
			from flights_v fv 
			where fv is not null
			group by  month, arrival_city)as sub
		)
	) t
WHERE therank = 1;
```

9.
```
select seat_no, count(seat_no)
from ticket_flights tf 
join boarding_passes bp 
on tf.ticket_no = bp.ticket_no 
where fare_conditions = 'Business'
group by seat_no
order by count desc
limit 1;
```
