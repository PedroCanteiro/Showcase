
- TrendSurfer: My Trading Algorithm automated into an EA for the Metatrader5 Platform. Based on C++/MQL5.
- Portfolio: My personal investment portfolio focused on the energy sectors and commodities.
- SQL Project: A Basic SQL analysis made during my learning.
- PoweBI Project(To be released): A basic project made during my learning.

# SQL Project Analysis
## Introduction
Welcome to my SQL Portfolio Project, where I delve into the data job market with a focus on data analyst roles. This project is a personal exploration into identifying the top-paying jobs, in-demand skills, and the intersection of high demand with high salary in the field of data analytics.
## Background
The motivation behind this project stemmed from my desire to understand the data analyst job market better, while practicing my SQL skills I learned during a basic course. I aimed to discover which skills are paid the most and in demand, making my job search more targeted and effective. 

The data for this analysis is from Luke Barousse’s SQL Course. This data includes details on job titles, salaries, locations, and required skills. 

The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn for a data analyst looking to maximize job market value?
## Tools I used
In this project, I utilized a variety of tools to conduct my analysis:

- **SQL** (Structured Query Language): Enabled me to interact with the database, extract insights, and answer my key questions through queries.
- **PostgreSQL**: As the database management system, PostgreSQL allowed me to store, query, and manipulate the job posting data.
- **Visual Studio Code:** This open-source administration and development platform helped me manage the database and execute SQL queries.
## The analysis
### 1.Top Paying Jobs
To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs or jobs in my country. This query highlights the high paying opportunities in the field.

Note than initially, I wanted to analyse only within roles related to finance, hence the commented line with the **job_title** associated with keywords such as invest or portfolio.
However, it seems that it is standard practiced by companies that work with financial roles such as banks, to not include the salary in their job postings, because the query only returned job postings with "null" values for the salaries. 
Therefore I had to broaden the spectrum and include all job titles in the query:
```sql
SELECT 
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name as company_name

FROM 
job_postings_fact
LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id

WHERE
    (job_location = 'Anywhere' or  job_location = 'Portugal' or job_location = 'Lisbon' or job_location = 'Lisboa')
    AND salary_year_avg is NOT NULL
   -- removed because all salaries are null or (job_title like '%financ%' or job_title like '%invest%' or job_title like '%portfolio%' or job_title like '%risk%')

ORDER BY salary_year_avg desc

limit 100;
```
<img width="1345" height="900" alt="image" src="https://github.com/user-attachments/assets/f36c68ba-b256-44ae-9379-44e228444483" />

### 2. Skills for Top Paying Jobs
To understand what skills are required for the top-paying jobs, I joined the job postings with the skills data, providing insights into what employers value for high-compensation roles.
```sql
WITH Top_paying_Jobs AS(

    SELECT 
        job_id,
        job_title,
        job_schedule_type,
        salary_year_avg,
        name as company_name

    FROM 
    job_postings_fact
    LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id

    WHERE
        (job_location = 'Anywhere' or job_location = 'Lisbon'
        or job_location = 'Lisboa' or job_location = 'Portugal')
        AND salary_year_avg is NOT NULL

    ORDER BY salary_year_avg desc

    limit 10
            )


SELECT 
    Top_paying_Jobs.*,
    skills

from Top_Paying_Jobs
INNER JOIN skills_job_dim ON Top_paying_Jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

ORDER BY
    salary_year_avg DESC;
```
<img width="1318" height="892" alt="image" src="https://github.com/user-attachments/assets/380485b9-8cbd-40ba-85a0-f295a615c457" />

### 3. In-Demand Skills for Data Analysts
This query helped identify the skills most frequently requested in job postings, directing focus to areas with high demand.
```sql
SELECT 
    skills,
    count(skills_job_dim.job_id) as demand_count

from job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

GROUP BY
    skills

ORDER BY
    demand_count DESC

LIMIT 10;
```

<img width="370" height="356" alt="image" src="https://github.com/user-attachments/assets/678d2b68-0b7c-44f7-985b-8d1091abef0b" />

### 4. Skills Based on Salary
Exploring the average salaries associated with different skills revealed which skills are the highest paying.
```sql
SELECT 
    skills,
    round (avg(salary_year_avg),0) as AverageSalary

from job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

WHERE 
    salary_year_avg IS NOT NULL

GROUP BY
    skills

ORDER BY
    AverageSalary DESC

LIMIT 25;
```
<img width="375" height="802" alt="image" src="https://github.com/user-attachments/assets/37198209-9b5e-4419-bcc6-ab2d420c1371" />

### 5. Most Optimal Skills to Learn
Combining insights from demand and salary data, this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skill development.
```sql
WITH skills_demand AS (
  SELECT
    skills_dim.skill_id,
		skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	  INNER JOIN
	    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
  WHERE
		job_postings_fact.salary_year_avg IS NOT NULL
  GROUP BY
    skills_dim.skill_id
),

average_salary AS (
  SELECT
    skills_job_dim.skill_id,
    AVG(job_postings_fact.salary_year_avg) AS avg_salary
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
  WHERE
		job_postings_fact.salary_year_avg IS NOT NULL
  GROUP BY
    skills_job_dim.skill_id
)

SELECT
  skills_demand.skills,
  skills_demand.demand_count,
  ROUND(average_salary.avg_salary, 0) AS avg_salary
FROM
  skills_demand
	INNER JOIN
	  average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE 
    demand_count > 10
ORDER BY
  avg_salary DESC,
  demand_count DESC 
;
```
<img width="565" height="891" alt="image" src="https://github.com/user-attachments/assets/d92f8d0f-048c-4345-a5fc-6b265c4b36ad" />

Each query not only served to answer a specific question but also to improve my understanding of SQL and database analysis. Through this project, I learned to leverage SQL's powerful data manipulation capabilities to derive meaningful insights from complex datasets.
## What I learned
Throughout this project, I honed several key SQL techniques and skills:

- **Complex Query Construction**: Learning to build advanced SQL queries that combine multiple tables and employ functions like **`WITH`** clauses for temporary tables.
- **Data Aggregation**: Utilizing **`GROUP BY`** and aggregate functions like **`COUNT()`** and **`AVG()`** to summarize data effectively.
- **Analytical Thinking**: Developing the ability to translate real-world questions into actionable SQL queries that got insightful answers.
  ### **Insights**

From the analysis, several general insights emerged:

1. **Top-Paying Data Analyst Jobs**: The highest-paying jobs for data analysts that allow remote work offer a wide range of salaries, the highest at $650,000!
2. **Skills for Top-Paying Jobs**: High-paying data analyst jobs require advanced proficiency in SQL, suggesting it’s a critical skill for earning a top salary.
3. **Most In-Demand Skills**: SQL is also the most demanded skill in the data analyst job market, thus making it essential for job seekers.
4. **Skills with Higher Salaries**: Specialized skills, such as SVN and Solidity, are associated with the highest average salaries, indicating a premium on niche expertise.
5. **Optimal Skills for Job Market Value**: SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value.
## Conclusions
This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.
