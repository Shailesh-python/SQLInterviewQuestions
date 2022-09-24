# SQL Interview Questions

<img src='https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white)'/>

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

Check solution link [Ankit Bansal Solution](https://youtu.be/oGYinDMDfnA)

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
![image](https://user-images.githubusercontent.com/81180156/192115140-8c8e836c-6f40-4505-8cab-a48d250b15a5.png)

Check solution link [Ankit Bansal Solution](https://youtu.be/KLqRHJ-Eg2s)

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
![image](https://user-images.githubusercontent.com/81180156/192113828-9b7f935b-1f50-47e2-93a0-0a3f82c53f67.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/KLqRHJ-Eg2s)

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
![image](https://user-images.githubusercontent.com/81180156/192113875-7f279c73-0331-4571-89a2-2542e0c9f52d.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/3qEfsSC27_4)

## [Question #6](#case-study-questions)

> Write the sql query to report the students (student_id, student_name) being "quite" in all exams.

A "quite" student is one who took atleast on exam and did not score the high score nor the low score in any of the exam.
Do not return the student who have never taken any exam.
Return the result table ordered by student_id
```sql
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
```
![image](https://user-images.githubusercontent.com/81180156/192113986-475900fa-9d73-48be-9f7b-da7569039f2d.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/6CH7IU4yB5I)

## [Question #7](#case-study-questions)	 
> WRITE A QUERY TO FIND THE GOLD MEDAL PER SWIMMER WHO WON ONLY GOLD MEDALS

METHOD - 1 
```sql
SELECT 
	GOLD,
	COUNT(GOLD) AS TOTAL_GOLD
FROM EVENTS
WHERE GOLD NOT IN (SELECT BRONZE FROM EVENTS UNION SELECT SILVER FROM events)
GROUP BY GOLD
```
METHOD - 2
```sql
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
```
![image](https://user-images.githubusercontent.com/81180156/192114177-249f71c4-2454-4b44-a3eb-3fec366f381f.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/dOLBRfwzYcU)

## [Question #8](#case-study-questions)	 

> WRITE A SQL TO POPULATE CATEGORY VALUES TO LAST NOT NULL VALUE

LIKE DOWN FILL IN EXCEL AND POWER QUERY
```sql
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
```		
![image](https://user-images.githubusercontent.com/81180156/192114282-e694e0f4-b95c-4af2-ba08-5594a251c874.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/Xh0EevUOWF0)

## [Question #9](#case-study-questions)	 
> Find sachin's milestone innings/matches.
```sql
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
```
![image](https://user-images.githubusercontent.com/81180156/192114648-95352e93-7033-4720-8e1b-9196e795a143.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/7LufPVm01NQ)

## [Question #10](#case-study-questions)	

> Write a sql to determine phone numbers that satisfies the below condition:
1-the numbers have both incoming and outgoing calls
2-the sum of duration of outgoing calls should be greater than sum of duration of incoming calls

```sql
;with cte_in as 
(
	select * from call_details where call_type = 'in'
), cte_out as 
	(select * from call_details where call_type = 'OUT')
```
Check solution link [Ankit Bansal Solution](https://youtu.be/pk8BKFysjP8)

## [Question #11](#case-study-questions)
> There are 3 rows in a movie hall each with 10 seats in each row
> write a sql query to find the four consecutive empty space.
```sql
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
```
![image](https://user-images.githubusercontent.com/81180156/192114800-e5fb9dd5-cbe6-41d4-bbcf-3f6bd3635018.png)
Check solution link [Ankit Bansal Solution](https://youtu.be/e4IILSHtKl4)
