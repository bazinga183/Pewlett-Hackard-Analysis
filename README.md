# Pewlett-Hackard-Analysis

## Overview of the analysis 
### Purpose
The purpose of this project is to assist Pewlett-Hackard in finding which of their employees are retiring and which employees can step up as potential mentors to help build up the soon-to-be absent labor force.

### Retiring Employees Analysis
I began this analysis by using QuickDBD to familiarize myself with how the different files from PH were connected.

![EmployeeDB](https://user-images.githubusercontent.com/46951897/128730935-90b8d1c9-0205-478d-a855-19a86298831a.png)

With the relationships between these files understood, I switched to using PostgreSQL 12.7 and pgAdmin 4.

I created a ['schema'](https://github.com/bazinga183/Pewlett-Hackard-Analysis/blob/main/Pewett-Hackard-Analysis-Folder/schema.sql) to map to be able to import the different csv files that I would work with. Next, I mapped out the databases that would be needed so that I can import the files:
```
CREATE TABLE departments (
	dept_no VARCHAR(4) NOT NULL,
	dept_name VARCHAR(40) NOT NULL,
	PRIMARY KEY (dept_no),
	UNIQUE (dept_name)
);

CREATE TABLE employees (
	emp_no INT NOT NULL,
	birth_date DATE NOT NULL,
	first_name VARCHAR NOT NULL,
	last_name VARCHAR NOT NULL,
	gender VARCHAR NOT NULL,
	hire_date DATE NOT NULL,
	PRIMARY KEY (emp_no)
);

CREATE TABLE dept_manager (
	dept_no VARCHAR(4) NOT NULL,
	emp_no INT NOT NULL,
	from_date DATE NOT NULL,
	to_date DATE NOT NULL,
FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
	PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE salaries(
	emp_no INT NOT NULL,
	salary INT NOT NULL,
	from_date DATE NOT NULL,
	to_date DATE NOT NULL,
	FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	PRIMARY KEY (emp_no)
);

CREATE TABLE dept_emp(
	emp_no INT NOT NULL,
	dept_no VARCHAR NOT NULL,
	from_date DATE NOT NULL,
	to_date DATE NOT NULL,
	FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
	PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE titles (
    emp_no INT NOT NULL,
    title VARCHAR NOT NULL,
    from_date DATE NOT NULL,
    to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
    PRIMARY KEY (emp_no, title, from_date)
);
```
With the headers and characteristics of the columns mapped, I imported the csv files into PostgreSQL so that I could play around with the data. I was required to then look for employees that would potentially retire in the near future according to their birth date and put this data into a table called retirement_titles:

```
SELECT e.emp_no,
       e.first_name,
       e.last_name,
       t.title,
       t.from_date,
       t.to_date
INTO retirement_titles
FROM employees as e
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
order by e.emp_no;
```
Then, in order to avoid duplicate employees from showing up twice in the data I used the ```DISTINCT ON``` statement and organized this into the unique_title table. Last, I got the total count of these employees to get a rough estimate of who would porentially retire:

```
-- Use Dictinct with Orderby to remove duplicate rows
SELECT DISTINCT ON (emp_no) emp_no,
first_name,
last_name,
title
INTO unique_titles
FROM retirement_titles
ORDER BY emp_no, to_date DESC;

-- Retrieve employees by job title who are about to retire.
SELECT COUNT(ut.emp_no),
ut.title
INTO retiring_titles
FROM unique_titles as ut
GROUP BY title 
ORDER BY COUNT(title) DESC;
```

When I executed these pieces of code, I got the following result:

![retirement_count](https://user-images.githubusercontent.com/46951897/128736130-2be35544-3982-4b3d-bb77-d86cb26e0e6f.PNG)

The results show a concerable amount of employees are projected to retire soon. Thus, my next assignment was to find out potential mentors within PH so that hiring new employees would be easier.

### Mentor Analysis
To begin, I needed to collect the columns of data from different files and join them all to the employees.csv file using the code:

```
-- Deliverable 2
SELECT DISTINCT ON(e.emp_no) e.emp_no, 
    e.first_name, 
    e.last_name, 
    e.birth_date,
    de.from_date,
    de.to_date,
    t.title
INTO mentorship_eligibilty
FROM employees as e
Left outer Join dept_emp as de
ON (e.emp_no = de.emp_no)
Left outer Join titles as t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
AND de.to_date = ('9999-01-01')
ORDER BY e.emp_no;
```

In this code, I joined the necessary data with the conditions that these workers were born during 1965 and currently employed at PH so that the data didn't show any non-existent employees. Luckily for PH, they have a potential 1549 workers that could be a part of this mentorship initiative.

![mentor_count](https://user-images.githubusercontent.com/46951897/128737719-d53a5177-0a79-463a-b17a-0ee202493569.PNG)

## Results
The analysis yielded these conclusions:
  - 57,668 Senior Engineers and Senior Staff members are soon to retire.
  - 26,465 Engineer and Staff members are soon to retire.
  - In total, 90,398 employees are projected to retire from PH. 
  - There are 1,549 potential mentors among the currently employed workers at PH.
  - PH needs to consider expanding the total number of mentors because the current figures show that each mentor would need to take on at least 58 new hires in order to make up the potential gap.
  - 
## Summary
In short, to make up for the upcoming "silver tsunami" that PH is going to experience, they need to fill 90,398 roles if the projections are correct.
As it currently stands, there are not enough mentors to help fill this gap because each mentor would need to mentor at least 58 students.
Potential questions that PH will need to consider in the future is:
  - Are there any ways they can incentivize these current projected retirees to stay at the company by increasing pay, benefits, or potential pensions so that the lack of labor does not impact the companies workflow in the future?
  - And how can they expand the total number of mentors within PH so that the current projected mentors are not ovewhelmed?
