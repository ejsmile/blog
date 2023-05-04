---
title: "Experiments on an Olympiad problem"
description: 'It so happened that I got into graduate school, and while walking past the department, I stumbled upon an Olympiad problem on 1C. In short, the problem sounds like this: "There are sales records for each day, you need to find the longest period when the plan was executed." And then, while walking with my sleeping daughter, I had a question, how many ways can this be done in SQL? Solutions will be based on MS SQL.'
date: 2017-05-03
tags : [
    "sql",
    "ms sql server",
    "ms sql",
    "development"
]
---
## Introduction

It so happened that I got into graduate school, and while walking past the department, I stumbled upon an Olympiad problem on 1C. In short, the problem sounds like this: 'There are sales records for each day, you need to find the longest period when the plan was executed.' And then, while walking with my sleeping daughter, I had a question, how many ways can this be done in SQL? Solutions will be based on MS SQL."

Let's create a table and begin.

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

## First method

SQL queries using self-join.

```sql
declare @planValue int = 15

--самый очевидный left
select 	
	top 1
	'to itself',
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

-- or as I prefer, through CTE 

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
	'to itself (with left)' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate1
order by
	DATEDIFF(day, BeginDate, EndDate) desc

--change to full join

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
	'to itself (with full)' title,
	BeginDate,
	DATEADD(Day, -1, EndDate) EndDate
from periodDate2
order by
	DATEDIFF(day, BeginDate, EndDate) desc
--change to right join
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
	'to itself (with right)' title,
	DATEADD(Day, 1, BeginDate) BeginDate, 
	EndDate
from periodDate3
order by
	DATEDIFF(day, BeginDate, EndDate) desc
```


Moreover, for any join we will get exactly the same execution plan (via NESTED LOOP (left outer join)).

Execution plans for queries using join.

{{< figure src="https://habrastorage.org/files/7c5/25f/612/7c525f612b1e47319d27298e7ed1ef80.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/e32/75a/a65/e3275aa650ae475ba8beea4578141da0.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/233/b0c/b5e/233b0cb5e05443c2add0194b709134d0.png" class="mid" >}}
{{< figure src="https://habrastorage.org/files/8c3/187/cff/8c3187cff5994ab18c8a6916eb5158d4.png" class="mid" >}}


## Second method


SQL queries using a correlated subquery.


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
	'correlated subquery"', p.BeginDate, p.EndDate
from 
	periodDate4 p 
order by 
	DATEDIFF(DAY,  p.BeginDate, p.EndDate) desc
```

In this case, we get NESTED LOOP (inner join).

{{< figure src="https://habrastorage.org/files/e93/01b/45f/e9301b45ffd44e718c59cee4e85cb24b.png" class="mid" caption="Execution plans for a correlated subquery">}}


## Third method


Now let's take the APPLY function (which appeared in MS SQL 2005). Essentially, we will be applying a subquery to each row.


```sql
declare @planValue int = 15

;with periodDate5 as(
	select 
		s.date BeginDate, 
		ISNULL(min(e.date), '20170501') EndDate
	from [tmp].[forFindDate] s
		outer apply
		(
		--we will use the data storage feature (sorting by date).
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


In this case, we again get NESTED LOOP (left outer join), but this method is the most optimal at the moment (at least according to MS SQL).

{{< figure src="https://habrastorage.org/files/9c7/939/882/9c7939882a40495fa984e17a4e3527cd.png" class="mid" caption="Execution plans for apply">}}


So far, everything has been boring and mundane.

## Fourth method.

Let's play with recursion: for the current row, we will append data if the date is one day later and the sales plan is being executed. Thus, we will expand the interval.

Since there are not many data, the length of the sequence is small, as is the depth of the call stack.

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

{{< figure src="https://habrastorage.org/files/cf2/05b/93b/cf205b93b3384ccf966bf5801b7a6aeb.png" class="mid" caption="Recursion cte">}}


## Fifth method

Let's move on to the capabilities of MS SQL 2012 with the analytical function LEAD (window functions). The LEAD function returns the next values with a shift within the sections according to the sorting. We have two sections in our data: the execution and non-execution of the plan, sorting is needed by dates to find the maximum period. We will search for gaps in the non-execution of the plan, i.e. then when we shift the start dates forward and the end dates back, we will get periods of plan execution. In this case, we are interested only in the section of non-execution of the plan, but in general, the division can be made as follows:```sql OVER ( PARTITION BY iif(f.value < @planValue, 1, 0) order by f.date)```


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


I see. It is evident in the execution plan that there are no table joins, except for adding dates outside the segment to handle segment boundaries correctly.

{{< figure src="https://habrastorage.org/web/236/032/9f9/2360329f9d174b8e8b4ed94de28848ac" class="mid" caption="Plan windows function">}}

## Sixth way

Now let's have some fun and multiply the table by itself (Cartesian product): we will find all possible date combinations (30*30 = 900), select those where the start date is less than the end date and there is no event of not executing the plan during this interval.

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
The sixth method is the slowest and heaviest. But we are not chasing performance, we are chasing the number of solutions to the problem.

{{< figure src="https://habrastorage.org/files/e3b/484/d1f/e3b484d1fce343db8e05f25655ed93e2.png" class="mid" caption="Plan cartesian product">}}

## Seventh method:

Using a simple cursor. We iterate over the data where the plan is executed, and search for the longest consecutive sequence.

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

## Not my ways of solving the problem

This approach involves using window functions to find the transitions between plan execution and non-execution states, and then selecting the appropriate intervals. There may be issues if an interval starts with plan execution or ends with plan execution, as there will be no transition and the data may be selected incorrectly.


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


This method involves defining intervals based on date shifts: consecutive dates have the same day increment, just like the regular increment across rows. Based on this property, we can calculate a certain offset, and if the offsets of several dates coincide with the increment growth, then these dates are sequential.


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