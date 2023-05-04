---
title: "Эксперименты над олимпиадной задачей"
description: 'Так получилось, что я попал в магистратуру, и как то гуляя мимо кафедры на глаза попалась олимпиадная задача по 1С. Кратко задача звучит так: "Есть записи продажи за каждый день, необходимо найти наибольший период когда план выполнялся". А потом когда я гулял со спящей дочкой у меня встав вопрос, а сколькими способами это можно сделать на SQL. Решения будут на основе MS SQL.'
date: 2017-05-03
tags : [
    "sql",
    "ms sql server",
    "ms sql",
    "development"
]
---
## Предисловие

Так получилось, что я попал в магистратуру, и как то гуляя мимо кафедры на глаза попалась олимпиадная задача по 1С. Кратко задача звучит так: "Есть записи продажи за каждый день, необходимо найти наибольший период когда план выполнялся". А потом когда я гулял со спящей дочкой у меня встав вопрос, а сколькими способами это можно сделать на SQL. Решения будут на основе MS SQL.

Создадим таблицу и начнем

```sql
CREATE TABLE [tmp].[forFindDate](
	[date] [datetime] NOT NULL,
	[value] [int] NOT NULL,
 CONSTRAINT [PK_forFindDate] PRIMARY KEY CLUSTERED 
(
	[date] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170401', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170402', 20)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170403', 20)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170404', 20)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170405', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170406', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170407', 30)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170408', 36)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170409', 35)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170410', 30)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170411', 30)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170412', 20)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170413', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170414', 40)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170415', 40)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170416', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170417', 50)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170418', 52)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170419', 53)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170420', 53)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170421', 50)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170422', 51)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170423', 52)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170424', 50)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170425', 50)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170426', 50)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170427', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170428', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170429', 10)
INSERT INTO [tmp].[forFindDate] ([date], [value]) VALUES ('20170430', 10)

GO

```


## Первый способ 


SQl запросы через join на самого себя


```sql
declare @planValue int = 15

--самый очевидный left
select 	
	top 1
	'сам на себя',
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from (select 
		s.date BeginDate, 
		ISNULL(min(e.date), '20170501') EndDate
	from [tmp].[forFindDate] s
		left join [tmp].[forFindDate] e
			on
				s.date < e.date
			and
				not e.value > @planValue
	where
		s.value > @planValue
	group by
		s.date) tmp
order by
	DATEDIFF(day, BeginDate, EndDate) desc

-- или как мне нравиться больше через with

;with periodDate1 as(
	select 
		s.date BeginDate, 
		ISNULL(min(e.date), '20170501') EndDate
	from [tmp].[forFindDate] s
		left join [tmp].[forFindDate] e
			on
				s.date < e.date
			and
				not e.value > @planValue
	where
		s.value > @planValue
	group by
		s.date
)
select 
	top 1
	'сам на себя (with left)' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate1
order by
	DATEDIFF(day, BeginDate, EndDate) desc

--меняем на full

;with periodDate2 as(
	select 
		s.date BeginDate, 
		ISNULL(min(e.date), '20170501') EndDate
	from [tmp].[forFindDate] s
		full join [tmp].[forFindDate] e
			on
				s.date < e.date
			and
				not e.value > @planValue
	where
		s.value > @planValue
	group by
		s.date
)
select 
	top 1
	'сам на себя (with full)' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate2
order by
	DATEDIFF(day, BeginDate, EndDate) desc
--меняем на right
;with periodDate3 as(
	select 
		ISNULL(max(s.date), '20170401') BeginDate,
		e.date EndDate
	from [tmp].[forFindDate] s
		right join [tmp].[forFindDate] e
			on
				s.date < e.date
			and
				not s.value > @planValue
	where
		e.value > @planValue
	group by
		e.date
)
select 
	top 1
	'сам на себя (with right)' title,
	DATEADD(Day, 1, BeginDate) BeginDate, 
	EndDate
from periodDate3
order by
	DATEDIFF(day, BeginDate, EndDate) desc
```


Причем при любом join получим абсолютно одинаковый план (через NESTED LOOP (left outer join)).

Планы исполнения запросов через join

{{< figure src="https://habrastorage.org/files/7c5/25f/612/7c525f612b1e47319d27298e7ed1ef80.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/e32/75a/a65/e3275aa650ae475ba8beea4578141da0.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/233/b0c/b5e/233b0cb5e05443c2add0194b709134d0.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/8c3/187/cff/8c3187cff5994ab18c8a6916eb5158d4.png" class="mid" >}}


## Второй способ


SQL запросы через корреляционный запрос


```sql
declare @planValue int = 15

;with periodDate4 as(
	select 
		s.date BeginDate, 
		ISNULL((select DATEADD(DAY, -1, isNull(min(d.date), '20170501')) from [tmp].[forFindDate] d where d.date > s.date and not d.value > @planValue), '20170501') EndDate
	from 
		[tmp].[forFindDate] s
	where
		s.value > @planValue			
)
select top 1 
	'корреляционный запрос', p.BeginDate, p.EndDate
from 
	periodDate4 p 
order by 
	DATEDIFF(DAY,  p.BeginDate, p.EndDate) desc
```


В данном случай мы получаем NESTED LOOP (inner join)).

{{< figure src="https://habrastorage.org/files/e93/01b/45f/e9301b45ffd44e718c59cee4e85cb24b.png" class="mid" caption="Планы исполнения корреляционного запроса">}}


## Третий способ


пока все не интересно, теперь возьмем функцию APPLY (появилась в MS SQL 2005). Фактически на  каждую строку будем делать под запрос.


```sql
declare @planValue int = 15

;with periodDate5 as(
	select 
		s.date BeginDate, 
		ISNULL(min(e.date), '20170501') EndDate
	from [tmp].[forFindDate] s
		outer apply
		(
		--особенностью храненим данных (сортировка по дате)
		select
			top 1
				ee.date
		from
			[tmp].[forFindDate] ee
		where
				s.date < ee.date
			and
				not ee.value > @planValue
		) e
	where
		s.value > @planValue
	group by
		s.date
)

select 
	top 1
	'sql 2005 apply' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate5
order by
	DATEDIFF(day, BeginDate, EndDate) desc
```


В данном случай опять получаем  NESTED LOOP (left outer join), но этот способ самый оптимальный на данный момент (по крайне мере так считает MS SQL).

{{< figure src="https://habrastorage.org/files/9c7/939/882/9c7939882a40495fa984e17a4e3527cd.png" class="mid" caption="Планы исполнения apply">}}


Пока все было скучно и обыденно.

## Четвертый способ


Поиграем с рекурсией: к текущей строке будем клеить данные если дата на один день старше и выполняется план продаж. Таким образом будем расширять интервал. 

Т.к. данных не много, то длина последовательности небольшая, как и глубина стека вызовов (CTE recursive возможно начиная с 2005).

```sql
declare @planValue int = 15

;with periodDate6 as (
	select
		f.date BeginDate,
		DATEADD(day, 1, f.date) EndDate
	from
		[tmp].[forFindDate] f
	where
		f.date = '20170417'
	and
		f.value > @planValue
	union all
	select
		r.BeginDate,
		DATEADD(day, 1, f.date) EndDate
	from
		[tmp].[forFindDate] f
			inner join periodDate6 r
				on
					r.EndDate = f.Date
				and
					value > @planValue
)
select 
	top 1
	'recursive' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate6
order by
	DATEDIFF(day, BeginDate, EndDate) desc
```

{{< figure src="https://habrastorage.org/files/cf2/05b/93b/cf205b93b3384ccf966bf5801b7a6aeb.png" class="mid" caption="Планы cte">}}


## Пятый способ

Перейдем к возможностям MS SQL 2012 к аналитической функций LEAD (оконные функций). Функция LEAD возвращает следующие значения со сдвигом в пределах секций по сортировке. Секций в данных у нас две: выполнения и не выполнения плана, сортировка нужна по датам, что бы найти максимальный период поступим хитро: будем искать пропуски в не выполнении плана, т.е. потом когда мы сдвинем даты начала вперед, а конец назад, то получим отрезки выполнения плана. В данном случай нам интересна только секция не выполнения плана её и возьмем, но в целом деление можно сделать так ```sql OVER ( PARTITION BY iif(f.value < @planValue, 1, 0) order by f.date)```


```sql
declare @planValue int = 15

with tmp as (	
	select '20170331' [date]
	union all
	select
		date 
	from [tmp].[forFindDate] f
	where
		f.value < @planValue
	union all
	select '20170501'	

), periodDate4 as (
	select
		date beforDate,
		lead(f.date, 1, '20170501')
		OVER (order by f.date) afterDate
	from tmp f
)
select 
	top 1
	'func 2012 t' title,
	DATEADD(Day, 1, beforDate) BeginDate,
	DATEADD(Day, -1, afterDate) EndDate
from periodDate4
order by
	DATEDIFF(day, beforDate, afterDate) desc
```


На плане исполнения видно, что не происходит соединения таблиц, за исключение добавления дат вне отрезка для корректной работы с концами отрезков.

{{< figure src="https://habrastorage.org/web/236/032/9f9/2360329f9d174b8e8b4ed94de28848ac" class="mid" caption="Планы оконной функций">}}

## Шестой способ

А теперь побалуемся перемножим таблицу саму на себя (декартого произведение): найдем все возможные сочетания дат (30*30 = 900), отберем те у которых дата начало меньше даты конца и в этом интервале нет события не исполнения плана.

```sql
declare @planValue int = 15

;with periodDate8 as(
	select 
		ISNULL(s.date, '20170401') BeginDate, 
		ISNULL(e.date, '20170501') EndDate
	from [tmp].[forFindDate] s, [tmp].[forFindDate] e
)
select top 1 
	'for Fun', p.BeginDate, p.EndDate
from periodDate8 p 
	left join [tmp].[forFindDate] b
		on
			b.date between p.BeginDate and p.EndDate
		and
			not b.value > @planValue
where
	p.BeginDate < p.EndDate
and
	b.date is null
order by
	DATEDIFF(day, BeginDate, EndDate) desc
```
Самый медленный и тяжелый способ. Но мы гонимся не за производительностью, а за количество решений задачи.

{{< figure src="https://habrastorage.org/files/e3b/484/d1f/e3b484d1fce343db8e05f25655ed93e2.png" class="mid" caption="Планы декартового произведение">}}

## Седьмой способ


Обычный курсор. По перебираем данные у которых план выполнен, и ищем максимальный последовательный участок.

```sql
declare @planValue int = 15
declare @endDate datetime = null, 
		@Intervat int = 0,
		@maxIntervat int = 0, 
		@old_date datetime = '20170401', 
		@cur_date datetime

DECLARE date_cursor CURSOR  
    FOR SELECT date FROM [tmp].[forFindDate] where value > @planValue
	OPEN date_cursor  
		FETCH NEXT FROM date_cursor 
		INTO @cur_date  
		WHILE @@FETCH_STATUS = 0  
			BEGIN

			IF (@cur_date = DATEADD(DAY, 1, @old_date)) BEGIN
				SET @Intervat = @Intervat + 1
				IF(@Intervat > @maxIntervat) BEGIN
					SET @maxIntervat = @Intervat
					SET @endDate = @cur_date
				END
			END else 
			Begin
				set @Intervat = 0
			end
			SET @old_date = @cur_date
			FETCH NEXT FROM date_cursor 
			INTO @cur_date
		END   
	CLOSE date_cursor;  
DEALLOCATE date_cursor;  

select 'cursor' title, DATEADD(DAY, - @maxIntervat, @endDate) BeginDate, @endDate EndDate
```

## Не мой варианты

Через оконный функций найдем переходы, между состояниями выполнено/не выполнение плана. И потом подбор интервалов. Могут быть проблемы из-за того, что интервал начался с выполнения плана или закончился выполнение плана, то не произойдет переход и данные подберутся не верно.


```sql
declare @planValue int = 15

;WITH Step01 AS
    (SELECT
            *
    FROM
            tmp.forFindDate
    WHERE     
            value > @planValue
	)
, Step02 AS
	(SELECT
		date
		, CASE
			WHEN date - LAG([date], 1, '20170401') OVER(ORDER BY date)  > 1
			THEN 1
			ELSE 0
		END AS isStart
		, CASE
			WHEN LEAD([date], 1, '20170430') OVER(ORDER BY date) - date > 1
			THEN 1
			ELSE 0
		END AS isEnd
FROM
		Step01)
, Step03 AS (
	SELECT
		
		date AS rangestart
		, CASE
			WHEN isEnd = 1
			THEN date
			ELSE LEAD(date, 1) OVER(ORDER BY date)
		END AS  rangeend
		, isstart
FROM
		Step02
WHERE     
	isstart = 1
		OR isend = 1)
SELECT TOP 1
	rangestart
	, rangeend
	, DATEDIFF(day, rangestart, rangeend) AS 'дни'
FROM
	Step03
WHERE     
	isstart = 1
ORDER BY 'дни' DESC
```


С определение интервалов по сдвигу дат: последовательно идущие даты имеют одинаковое приращения по дням, как обычный инкремент по строкам. На это свойстве можно получить какое то смещения и если смещения у нескольких дат совпадает при росте инкремента, то эти даты идут последовательно.


```sql
declare @planValue int = 15

;with internals AS (
    SELECT
		dateadd(day, -ROW_NUMBER() OVER (ORDER BY date), [date]) fake_start,
		[date]
    FROM tmp.forFindDate 
	where 
		value > 15
  )
SELECT TOP 1
  'new method' as title,
  MIN([date]) AS BeginDate,
  MAX([date]) AS EndDate
FROM 
	internals
GROUP BY 
	fake_start
ORDER BY 
	COUNT(*) DESC
```

[habrahabr.ru](https://habrahabr.ru/post/327862/)