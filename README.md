### SQLInterviewQuestions

## [Question #1](#case-study-questions)
> Find companies who have atleast 2 users who speaks English and German both the language.
```sql
SELECT 
	A.company_id,
	COUNT(1) AS Total_Users
FROM 
(
	SELECT 
		C.company_id,
		C.user_id,
		COUNT(1) AS Languages_Speaks
	FROM DBO.company_users C
	WHERE C.[language] IN ( 'English', 'German')
	GROUP BY C.company_id,C.user_id
	HAVING COUNT(1) = 2
) A
GROUP BY A.company_id
HAVING COUNT(1) >= 2
```
![image](https://user-images.githubusercontent.com/81180156/192112581-580aaf55-9974-4214-a8f1-c45c24fec39d.png)

## [Question #2](#case-study-questions)
> Write a sql to find the total number of people present inside the hospital.

Method 1
```sql
WITH cte as 
(
SELECT 
	emp_id,
	action,
	time,
	MAX(H.[time]) OVER (PARTITION BY EMP_ID ORDER BY EMP_iD) AS Max_time

FROM DBO.hospital H
)
    SELECT * FROM cte 
    WHERE cte.Max_time = cte.[time] 
        AND cte.[action] = 'in'
```
Method 2
```sql
;WITH cte as 
(
SELECT 
	H.emp_id,
	MAX(CASE WHEN H.[action] = 'IN' THEN H.[time] END) AS IN_TIME,
    MAX(CASE WHEN H.[action] = 'OUT' THEN H.[time] END) AS OUT_TIME
FROM DBO.hospital H
GROUP BY H.emp_id
)
    SELECT 
        * 
    FROM cte 
    WHERE CTE.OUT_TIME < CTE.IN_TIME OR cte.OUT_TIME IS NULL 
```

Method 3 
```sql
;WITH cte1 AS
(
SELECT emp_id , MAX(time) in_time FROM dbo.hospital WHERE [action] = 'in' GROUP BY emp_id)
,cte2 AS
(
SELECT emp_id , MAX(time) out_time FROM dbo.hospital WHERE [action] = 'out' GROUP BY emp_id)

    SELECT 
        cte2.emp_id,
        cte1.in_time,
        cte2.out_time 
    FROM cte1 left JOIN cte2
        on cte1.emp_id = cte2.emp_id 
    WHERE cte1.in_time > cte2.out_time
        or cte2.out_time IS NULL
```
![image](https://user-images.githubusercontent.com/81180156/192112750-ba627b92-8835-4f02-9878-de8af4ee5f8a.png)

## [Question #3](#case-study-questions)
> Write a sql query to find the business days between create date and resolved date excluding weekend and holidays.
```sql
SELECT
    *,
    DATEDIFF(DAY, CREATE_DATE, RESOLVED_DATE),
    DATENAME(WEEK,CREATE_DATE),
    DATENAME(WEEK, RESOLVED_DATE),
    DATEDIFF(WEEK, CREATE_DATE, RESOLVED_DATE)
    
FROM DBO.tickets

-- From above we can conclude that 
-- if diff is 0 then 0 weekend i.e. 0 days
-- if diff is 1 then 1 weekend i.e. 2 days
-- if diff is 2 then 2 weekend i.e. 4 days

SELECT 
	ticket_id,
	create_date,
	resolved_date,
	DATEDIFF(DAY, create_date, resolved_date) - 
	DATEDIFF(WEEK, create_date, resolved_date) * 2 -
	numberOF_holidays AS Working_Days_To_Resolve
	
FROM 
(
	SELECT 
		T.ticket_id,
		T.create_date,
		T.resolved_date,
		COUNT(H.holiday_date) as numberOF_holidays
	FROM DBO.tickets T
	LEFT JOIN DBO.holidays H ON H.holiday_date BETWEEN T.create_date AND T.resolved_date
	GROUP BY T.ticket_id, T.create_date, T.resolved_date
) A
```

## [Question #4](#case-study-questions)
A COMPANY WANTS TO HIRE NEW EMPLOYEES. THE BUDGET OF THE COMPANY FOR SALARIES IS 70000.
THE COMPANY'S CRITERIA TO HIRE ARE :
> KEEP HIRING THE SENIOR WITH SMALLEST SALARY UNTIL YOU CANNOT HIRE ANYMORE SENIORS.
> USE REMAINING BUDGET TO HIRE THE JUNIOR WITH SMALLEST SALARY.
> KEEP HIRING THE JUNIOUR WITH SMALLEST SALARY UNTIL YOU CANNOT HIRE ANYMORE JUNIORS.

> WRITE SQL QUERY TO FIND THE SENIORS AND JUNIORS HIRED UNDER THE MENTIONED CRITERIA.

```sql
WITH SENIOR AS
(
SELECT 
	*
	,SUM(SALARY) 
		OVER(
			PARTITION BY EXPERIENCE ORDER BY EMP_ID ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) 
			AS RUNNING_SALARY
FROM DBO.CANDIDATES
)
	SELECT * 
	FROM SENIOR 
	WHERE EXPERIENCE = 'SENIOR' AND RUNNING_SALARY <= 70000

	UNION ALL

	SELECT * 
	FROM SENIOR
	WHERE EXPERIENCE = 'JUNIOR' AND 
	RUNNING_SALARY <= 
		(SELECT SUM(SALARY)
		FROM SENIOR 
		WHERE EXPERIENCE = 'SENIOR' AND RUNNING_SALARY <= 70000)
```
## [Question #5](#case-study-questions)	   
> WRITE A SQL TO FIND OUT CALLERS WHOSE FIRST AND LAST CALL WAS TO THE SAME PERSON ON THE GIVEN DAY.
```sql
WITH CTE AS 
(
SELECT 
	P.Callerid,
	CAST(p.Datecalled as DATE) AS CalledDate,
	MIN(P.Datecalled) AS First_Call,
	MAX(P.Datecalled) AS Last_Call
FROM DBO.phonelog P
GROUP BY P.Callerid, CAST(p.Datecalled as DATE)
)
	SELECT 
		CTE.*,
		P1.Recipientid AS FIRST_REC,
		P2.Recipientid AS LAST_REC
	FROM CTE 
	INNER JOIN phonelog P1 
		ON CTE.Callerid = P1.Callerid AND CTE.First_Call = P1.Datecalled
	INNER JOIN phonelog P2
		ON CTE.Callerid = P2.Callerid AND CTE.Last_Call = P2.Datecalled
	WHERE P1.Recipientid = P2.Recipientid
```	
	
-------Question-6--------------------------------------------------------
https://youtu.be/6CH7IU4yB5I
scripts:
create table students
(
student_id int,
student_name varchar(20)
);
insert into students values
(1,'Daniel'),(2,'Jade'),(3,'Stella'),(4,'Jonathan'),(5,'Will');

create table exams
(
exam_id int,
student_id int,
score int);

insert into exams values
(10,1,70),(10,2,80),(10,3,90),(20,1,80),(30,1,70),(30,3,80),(30,4,90),(40,1,60)
,(40,2,70),(40,4,80);

-- Write the sql query to report the students (student_id, student_name) being "quite" in all exams.
-- A "quite" student is one who took atleast on exam and did not score the high score nor the low score in any of the exam.
-- Do not return the student who have never taken any exam.
-- Return the result table ordered by student_id


;with max_min_table as 
(select exam_id, min(score) min_score, max(score) max_score from exams group by exam_id)
,number_of_exams as 
(select student_id, count(distinct exam_id) as exam_numbers from exams group by student_id)

select 
	*
from exams
left join max_min_table on exams.exam_id = max_min_table.exam_id

left join number_of_exams on exams.student_id = number_of_exams.student_id
where exams.score > max_min_table.min_score 
	or exams.score < max_min_table.max_score

-------- Question 7 --------------------------------------------------------------------
https://youtu.be/dOLBRfwzYcU

script:
CREATE TABLE events (
ID int,
event varchar(255),
YEAR INt,
GOLD varchar(255),
SILVER varchar(255),
BRONZE varchar(255)
);

delete from events;

INSERT INTO events VALUES (1,'100m',2016, 'Amthhew Mcgarray','donald','barbara');
INSERT INTO events VALUES (2,'200m',2016, 'Nichole','Alvaro Eaton','janet Smith');
INSERT INTO events VALUES (3,'500m',2016, 'Charles','Nichole','Susana');
INSERT INTO events VALUES (4,'100m',2016, 'Ronald','maria','paula');
INSERT INTO events VALUES (5,'200m',2016, 'Alfred','carol','Steven');
INSERT INTO events VALUES (6,'500m',2016, 'Nichole','Alfred','Brandon');
INSERT INTO events VALUES (7,'100m',2016, 'Charles','Dennis','Susana');
INSERT INTO events VALUES (8,'200m',2016, 'Thomas','Dawn','catherine');
INSERT INTO events VALUES (9,'500m',2016, 'Thomas','Dennis','paula');
INSERT INTO events VALUES (10,'100m',2016, 'Charles','Dennis','Susana');
INSERT INTO events VALUES (11,'200m',2016, 'jessica','Donald','Stefeney');
INSERT INTO events VALUES (12,'500m',2016,'Thomas','Steven','Catherine');

-- WRITE A QUERY TO FIND THE GOLD MEDAL PER SWIMMER WHO WON ONLY GOLD MEDALS
-- METHOD - 1 
SELECT 
	GOLD,
	COUNT(GOLD) AS TOTAL_GOLD
FROM EVENTS
WHERE GOLD NOT IN (SELECT BRONZE FROM EVENTS UNION SELECT SILVER FROM events)
GROUP BY GOLD

---- METHOD - 1 
SELECT GOLD,COUNT(GOLD) AS Total_Gold
FROM events
WHERE GOLD IN 
(
SELECT GOLD FROM events 
EXCEPT
SELECT SILVER FROM events
EXCEPT 
SELECT BRONZE FROM events
)
GROUP BY GOLD

-----------QUESTION - 8 ----------------------------------
https://youtu.be/Xh0EevUOWF0
script:
create table brands 
(
category varchar(20),
brand_name varchar(20)
);
insert into brands values
('chocolates','5-star')
,(null,'dairy milk')
,(null,'perk')
,(null,'eclair')
,('Biscuits','britannia')
,(null,'good day')
,(null,'boost');

-- WRITE A SQL TO POPULATE CATEGORY VALUES TO LAST NOT NULL VALUE
-- LIKE DOWN FILL IN EXCEL AND POWER QUERY

WITH CTE AS 
(
SELECT 
	*,
	ROW_NUMBER() OVER ( ORDER BY (SELECT NULL)) AS RN
FROM BRANDS
),CTE_2 AS 
	(
		SELECT * ,
			LEAD(RN,1,100) OVER ( ORDER BY RN ) AS NEXT_RN
		FROM CTE 
		WHERE CATEGORY IS NOT NULL
	)
		
		SELECT 
			* 
		FROM CTE
		INNER JOIN CTE_2 
			ON CTE.RN >= CTE_2.RN 
				AND CTE.RN <= CTE_2.NEXT_RN-1
		

---------------------------- QUESTION - 9 ----------------------------------------

https://youtu.be/7LufPVm01NQ

DATASET IN DESCRIPTION

--Find sachin's milestone innings/matches

WITH CTE AS 
(
SELECT 
	*,
	SUM(RUNS) OVER (ORDER BY [MATCH] ASC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS TOTAL_RUNS
FROM SACHIN_SCORES
),CTE_MILESTONES AS 
(
	SELECT 
	*,
		CASE 
			WHEN CTE.TOTAL_RUNS >= 1000 AND CTE.TOTAL_RUNS < 5000 THEN 1
			WHEN CTE.TOTAL_RUNS >= 5000  AND CTE.TOTAL_RUNS < 10000 THEN 2
			WHEN CTE.TOTAL_RUNS > 10000  THEN 3 
		END AS MILESTONES
	FROM CTE
)
	SELECT 
		CTE_MILESTONES.MILESTONES,
		SUM(CASE WHEN CTE_MILESTONES.INNINGS IS NOT NULL THEN 1 ELSE 0 END ) AS TOTAL_INNINGS,
		COUNT(CTE_MILESTONES.MATCH) AS TOTAL_MATCHES

	FROM CTE_MILESTONES
	GROUP BY CTE_MILESTONES.MILESTONES
	
--------------------QUESTION 10 ----------------------------------------------------------------------------
https://youtu.be/pk8BKFysjP8

create table call_details  (
call_type varchar(10),
call_number varchar(12),
call_duration int
);

insert into call_details
values ('OUT','181868',13),('OUT','2159010',8)
,('OUT','2159010',178),('SMS','4153810',1),('OUT','2159010',152),('OUT','9140152',18),('SMS','4162672',1)
,('SMS','9168204',1),('OUT','9168204',576),('INC','2159010',5),('INC','2159010',4),('SMS','2159010',1)
,('SMS','4535614',1),('OUT','181868',20),('INC','181868',54),('INC','218748',20),('INC','2159010',9)
,('INC','197432',66),('SMS','2159010',1),('SMS','4535614',1);

/*Write a sql to determine phone numbers that satisfies the below condition:
1-the numbers have both incoming and outgoing calls
2-the sum of duration of outgoing calls should be greater than sum of duration of incoming calls
*/

with cte_in as 
(
	select * from call_details where call_type = 'in'
), cte_out as 
	(select * from call_details where call_type = 'OUT')

	select 

------- Question - 11 ------------------------------------------
https://youtu.be/e4IILSHtKl4

script:
create table movie(
seat varchar(50),occupancy int
);
insert into movie values('a1',1),('a2',1),('a3',0),('a4',0),('a5',0),('a6',0),('a7',1),('a8',1),('a9',0),('a10',0),
('b1',0),('b2',0),('b3',0),('b4',1),('b5',1),('b6',1),('b7',1),('b8',0),('b9',0),('b10',0),
('c1',0),('c2',1),('c3',0),('c4',1),('c5',1),('c6',0),('c7',1),('c8',0),('c9',0),('c10',1);

-- there are 3 rows in a movie hall each with 10 seats in each row
-- write a sql query to find the four consecutive empty space.

;with cte as 
(
select 
	left(seat,1) as seat_type,
	cast(SUBSTRING(seat,2,2) as int) as seat_id,
	seat,
	occupancy
from movie
), Cte2 as 
(
	select 
		*,
		max(cte.occupancy) over ( partition by cte.seat_type order by cte.seat_id asc rows between current row and 3 following) as is_4_Zeros,
		Count(cte.occupancy) over ( partition by cte.seat_type order by cte.seat_id asc rows between current row and 3 following) as is_4_rows
	from cte
),cte3 as 
(
	select * from cte2
	where cte2.is_4_rows = 4 and cte2.is_4_Zeros = 0
)	
	select * from cte
	inner join cte3
	on cte.seat_type = cte3.seat_type
		and cte.seat_id between cte3.seat_id and cte3.seat_id + 3
