# Employee_Data_Analysis_SQL

### Employee Attrition Analysis

## Employee Attrition Count 
	This query retrieves and counts the occurrences of each attrition status (Attrition) 
 from the employee_attrition table in the Attrition_dataset dataset of first-project-397223. 
 It groups the data by Attrition, presenting the number of employees for each attrition status (Yes or No). 
 This summary helps analyze and understand attrition trends within the specified dataset for 
 potential insights and decision-making.
 ```

 -- EMPLOYEE ATTRITION COUNT
  SELECT Attrition, COUNT(Attrition) AS Attrition_Count
  FROM `first-project-397223.Attrition_dataset.employee_attrition`
  GROUP BY Attrition;

```

## Employee Attrition Count By Department

This SQL query summarizes employee attrition counts by department from the employee_attrition table within the Attrition_dataset in BigQuery. It categorizes employees based on whether they have experienced attrition (Attrition = true) or not (Attrition = false), presenting these counts for each department. This analysis provides a clear view of attrition distribution across departments, aiding in workforce planning and retention strategies.


```
-- EMPLOYEE ATTRITION COUNT BY DEPARTMENT
SELECT  Department,
       COUNTIF(Attrition = true) AS Attrition_BY_Dept_TRUE,
       COUNTIF(Attrition = false) AS Attrition_BY_Dept_FALSE
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY Department;
```

## Performance Rating Analysis against avg dept performance using CTEs (Common Table Expression)
WITH Department_Average AS (
 		SELECT  Department, AVG(PerformanceRating) AS AVG_PerformanceRatingByDept
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY Department
),
Combining_Tables AS (
 			SELECT master_Table.*, AVG_PerformanceRatingByDept
 			FROM `first-project-397223.Attrition_dataset.employee_attrition` AS master_Table
 			LEFT JOIN Department_Average AS DEPT_AVG_TBL
 				ON master_Table.Department = DEPT_AVG_TBL.Department
),
Compare_Emp_Performance AS (
 			SELECT EmployeeNumber,
         		Department,
         		PerformanceRating, 
         		AVG_PerformanceRatingByDept,
         	IF(PerformanceRating > AVG_PerformanceRatingByDept, "GOOD", "CAN IMPROVE") AS Performance
 			FROM Combining_Tables
),
Check_Dept_PErformance AS (
 			SELECT Department,
         			COUNT(EmployeeNumber) AS Emp_Count,
         			COUNTIF(Performance = "GOOD") AS GOOD_Performance,
         			COUNTIF(Performance = "CAN IMPROVE") AS Improve_Performance
 			FROM Compare_Emp_Performance
 			GROUP BY Department
)


-- SELECT * FROM Department_Average
-- SELECT * FROM Combining_Tables
-- SELECT * FROM Compare_Emp_Performance
SELECT * FROM Check_Dept_PErformance
```
—----------------------------------

###### Repeating above analysis using “WINDOWS” Function

```
WITH Department_Average AS (
SELECT  EmployeeNumber,
         Department,
         PerformanceRating,
         AVG(PerformanceRating) OVER (PARTITION BY Department ORDER BY Department) AS AVG_PerformanceRatingByDept,
FROM `first-project-397223.Attrition_dataset.employee_attrition`
),
Analyse_EMP_Perf_Rating AS (
 SELECT Department_Average.* ,
     IF(PerformanceRating > AVG_PerformanceRatingByDept, "GOOD", "CAN IMPROVE" ) AS EMP_PERFORMANCE
 FROM Department_Average
),
Analyse_Dept_Performance AS (
 SELECT Department,
       COUNT(EmployeeNumber) AS EMP_COUNT,
       COUNTIF(EMP_PERFORMANCE = "GOOD") AS GOOD_COUNT,
       COUNTIF(EMP_PERFORMANCE = "CAN IMPROVE") AS TO_IMPROVE_COUNT
 FROM Analyse_EMP_Perf_Rating
 GROUP BY Department
)
--SELECT * FROM Department_Average
--SELECT * FROM Analyse_EMP_Perf_Rating
SELECT * FROM Analyse_Dept_Performance
```


Analyse Employee Monthly Income Vs Department Avg Monthly Income

WITH Department_Avg_Pay AS (
 SELECT EmployeeNumber,
         MonthlyIncome as Emp_Monthy_Income,
         Department,
       AVG(MonthlyIncome) OVER (PARTITION BY Department) AS Dept_Avg_Monthly_Pay
 FROM `first-project-397223.Attrition_dataset.employee_attrition`
),
Compare_EmpPay_Vs_DeptAvgPay AS (
 SELECT Department_Avg_Pay.*,
       IF(Emp_Monthy_Income > Dept_Avg_Monthly_Pay, "HAPPY" , "UNHAPPY" ) AS Employee_Happiness
 FROM Department_Avg_Pay
),


Analyse_Dept_Attrition_Possibility AS (
 SELECT Department,
       COUNT(EmployeeNumber) AS EMP_COUNT,
       COUNTIF(Employee_Happiness = "HAPPY" ) AS HAPPY_Employees,
       COUNTIF(Employee_Happiness = "UNHAPPY" ) AS UNHAPPY_Employees,
       ROUND((COUNTIF(Employee_Happiness = "HAPPY" ) / COUNT(EmployeeNumber)) * 100 ) AS HAPPY_Employees_Percentage ,
       ROUND((COUNTIF(Employee_Happiness = "UNHAPPY") / COUNT(EmployeeNumber)) * 100 ) AS UNHAPPY_Employees_Percentage
 FROM Compare_EmpPay_Vs_DeptAvgPay
 GROUP BY Department
)


-- SELECT * FROM Department_Avg_Pay
-- SELECT * FROM Compare_EmpPay_Vs_DeptAvgPay
-- SELECT * FROM Compare_EmpPay_Vs_DeptAvgPay
SELECT * FROM Analyse_Dept_Attrition_Possibility


Analyse OVETIME vs attrition



SELECT Overtime, count(EmployeeNumber)
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY OverTime;


-- USING CASE
------------------------------------------------------------
SELECT OverTime,
 			COUNT(
   			CASE 
   				WHEN Attrition = true THEN EmployeeNumber
 			END) AS Attrition_True,
 			COUNT(
   			CASE 
  				WHEN Attrition = false THEN EmployeeNumber
 			END) AS Attrition_False
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY OverTime;


-----------------------------------------------------------
-- USING COUNTIF


SELECT Overtime,
COUNTIF(Attrition = true) AS Attr_TRUE,
COUNTIF(Attrition = false) AS Attr_False
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY OverTime;








