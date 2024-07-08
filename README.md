# Employee_Data_Analysis_SQL


**Dataset Summary
**The dataset contains information about employee attrition at a company. Each row represents an individual employee and includes various attributes related to their demographic information, job role, and company-related details.

**Columns and Descriptions**

**Age**: The age of the employee.

**Attrition**: Whether the employee has left the company ("Yes" or "No").

**BusinessTravel**: The frequency of business travel (e.g., "Travel_Rarely", "Travel_Frequently").

**DailyRate**: The daily rate of the employee.

**Department**: The department in which the employee works (e.g., "Sales", "Research & Development").

**DistanceFromHome**: The distance of the employee's home from the workplace.

**Education**: The education level of the employee.

**EducationField**: The field of education of the employee (e.g., "Life Sciences", "Medical").

**EmployeeCount**: The count of employees (constant in this dataset).

**EmployeeNumber**: A unique identifier for each employee.

**EnvironmentSatisfaction**: The satisfaction level of the employee with the work environment.

**Gender**: The gender of the employee.

**HourlyRate**: The hourly rate of the employee.

**JobInvolvement**: The level of involvement of the employee in their job.

**JobLevel**: The job level of the employee.

**JobRole**: The role of the employee within the company (e.g., "Sales Executive", "Research Scientist").

**JobSatisfaction**: The job satisfaction level of the employee.

**MaritalStatus**: The marital status of the employee (e.g., "Single", "Married").

**MonthlyIncome**: The monthly income of the employee.

**MonthlyRate**: The monthly rate of the employee.

**NumCompaniesWorked**: The number of companies the employee has worked at.

**Over18**: Indicates if the employee is over 18 years old.

**OverTime**: Indicates if the employee works overtime ("Yes" or "No").

**PercentSalaryHike**: The percentage increase in salary.

**PerformanceRating**: The performance rating of the employee.

**RelationshipSatisfaction**: The satisfaction level of the employee with their relationships at work.

**StandardHours**: The standard working hours (constant in this dataset).

**StockOptionLevel**: The stock option level of the employee.

**TotalWorkingYears**: The total number of years the employee has worked.

**TrainingTimesLastYear**: The number of training times attended by the employee last year.

**WorkLifeBalance**: The work-life balance level of the employee.

**YearsAtCompany**: The number of years the employee has been with the company.

**YearsInCurrentRole**: The number of years the employee has been in their current role.

**YearsSinceLastPromotion**: The number of years since the employee's last promotion.

**YearsWithCurrManager**: The number of years the employee has been with their current manager.


### Employee Attrition Count 
	
 This query retrieves and counts the occurrences of each attrition status (Attrition) from the employee_attrition table in the Attrition_dataset dataset. 
 It groups the data by Attrition, presenting the number of employees for each attrition status (Yes or No). 
 This summary helps analyze and understand attrition trends within the specified dataset for 
 potential insights and decision-making.
 ```

 -- EMPLOYEE ATTRITION COUNT
  SELECT Attrition, COUNT(Attrition) AS Attrition_Count
  FROM `first-project-397223.Attrition_dataset.employee_attrition`
  GROUP BY Attrition;

```
<img width="267" alt="Screenshot 2024-07-05 at 6 41 37 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/c5246944-e64e-4d34-9f95-530ce5d3f552">

-------------------------------------------------------------------------------------------------------------------------

### Employee Attrition Count By Department

This SQL query summarizes employee attrition counts by department from the employee_attrition table within the Attrition_dataset in BigQuery. It categorizes employees based on whether they have experienced attrition (Attrition = true) or not (Attrition = false), presenting these counts for each department. This analysis provides a clear view of attrition distribution across departments, aiding in workforce planning and retention strategies.


```
-- EMPLOYEE ATTRITION COUNT BY DEPARTMENT
SELECT  Department,
       COUNTIF(Attrition = true) AS Attrition_BY_Dept_TRUE,
       COUNTIF(Attrition = false) AS Attrition_BY_Dept_FALSE
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY Department;
```
<img width="599" alt="Screenshot 2024-07-05 at 6 44 11 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/f3e78e04-52a6-486b-929d-a06867da8538">

-------------------------------------------------------------------------------------------------------------------------

### Performance Rating Analysis against avg dept performance using CTEs (Common Table Expression)

1. Department Average Performance:
	Computes the average PerformanceRating for each Department in the employee_attrition dataset (AVG_PerformanceRatingByDept).

2. Employee Performance Comparison:
	Joins each employee's PerformanceRating with their department's average rating and categorizes it as "GOOD" or "CAN IMPROVE" based on 	comparison.

3. Departmental Performance Summary:
	Counts employees (Emp_Count) in each department and categorizes them by performance ("GOOD" and "CAN IMPROVE"), providing insights 	into departmental performance trends.

This structured approach enables analysis of how departmental performance ratings relate to employee ratings, facilitating strategic insights for organizational improvement and performance management.

```
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

<img width="689" alt="Screenshot 2024-07-05 at 6 48 14 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/3153a610-740c-41fe-874c-283a671e0e18">


—-------------------------------------------------------------------------------------------------------------------

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
<img width="657" alt="Screenshot 2024-07-05 at 6 49 27 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/f74e3a4d-7530-43c5-bb1d-3e1be6c1503d">

-------------------------------------------------------------------------------------------------------------------------

### Analyse Employee Monthly Income Vs Department Avg Monthly Income

This SQL query evaluates employee happiness based on departmental average pay.

1. Departmental Average Pay Calculation:
	Computes each employee's monthly income (Emp_Monthly_Income) and the average monthly income (Dept_Avg_Monthly_Pay) for their 		department.

2. Comparison of Employee Happiness:
	Categorizes employees as "HAPPY" or "UNHAPPY" based on whether their monthly income is above or below the departmental average.

3. Analysis of Departmental Attrition Likelihood:
	Aggregates employee counts and percentages within each department, categorizing employees as "HAPPY" or "UNHAPPY", and calculates the 	percentage of each category.

This structured approach helps assess the relationship between employee satisfaction (based on income relative to departmental average) and potential attrition risks, providing insights for retention strategies and organizational management.

```
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

```
<img width="981" alt="Screenshot 2024-07-05 at 6 52 57 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/6a904a8c-a66b-4dc7-9937-40ec08c07e0d">

-------------------------------------------------------------------------------------------------------------------------

### Analyse OVERTIME vs attrition

This SQL query summarizes the count of employees based on their overtime status (Overtime) from the employee_attrition table in the Attrition_dataset:

1. Overtime Count by Category:
	Counts the number of employees (EmployeeNumber) grouped by their overtime status (Overtime).

2. Grouping by Overtime Status:
	Groups the data to distinguish between employees who work overtime (Overtime = true) and those who do not (Overtime = false).

3. Insight into Workload Distribution:
	Provides insights into how overtime is distributed among employees, facilitating analysis of workload management and potential 		implications for employee retention and productivity strategies.

```
SELECT Overtime, count(EmployeeNumber) AS Emp_Count
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY OverTime;
```

<img width="267" alt="Screenshot 2024-07-05 at 6 55 18 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/384f793a-b667-4707-8749-9f97d9094bb1">

-------------------------------------------------------------------------------------------------------------------------

###### USING CASE STATEMENT

This SQL query categorizes employees based on their overtime status (OverTime) and counts the number of employees who have experienced attrition (Attrition = true) or not (Attrition = false):

1. Overtime Grouping and Counting:
	Groups employees by their overtime status (OverTime) and counts the number of employees in each group.

2. Attrition Analysis by Overtime:
	Calculates the count of employees who have experienced attrition (Attrition_True) and those who have not (Attrition_False) within each 	overtime category (OverTime).

3. Insights into Attrition and Workload:
	Provides insights into how attrition rates vary based on overtime practices, helping to understand the relationship between workload 	(overtime) and employee turnover within the dataset.

```
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

```
<img width="391" alt="Screenshot 2024-07-05 at 7 01 49 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/99404105-f8c0-401f-92ad-5425362defe1">

-------------------------------------------------------------------------------------------------------------------------
###### USING COUNTIF

```

SELECT Overtime,
COUNTIF(Attrition = true) AS Attr_TRUE,
COUNTIF(Attrition = false) AS Attr_False
FROM `first-project-397223.Attrition_dataset.employee_attrition`
GROUP BY OverTime;

```
<img width="394" alt="Screenshot 2024-07-05 at 7 02 17 PM" src="https://github.com/jeevareha/Employee_Data_Analysis_SQL/assets/32441508/ab3c06c8-7877-4cac-89d9-5968b05cb629">








