# Job title: Service for parsing vacancies for IT specialists 

## 📃 Content

- Description of semester work ✅
- Conceptual schema ✅
- Loop discussion ✅
- Relational schema (:construction: in process)
- Create script (:construction: in process)
- Insert script (:construction: in process)
- Queries ✅
- Categories covered by queries ✅
    - Category	
    - Description	
    - Covered by
- Conclusion (:construction: in process)
- References (:construction: in process)
- Iteration Score

# Description

In the modern world, where technological progress and the development of the IT industry are at their peak, finding a job for IT specialists is becoming increasingly important. However, finding the right job can take a lot of time and effort, especially when there are many options on the job market.

Job scraping service for IT professionals is an innovative solution that allows you to automate the search for suitable vacancies and significantly reduce the time spent searching for a job. This service uses machine learning and data analysis technologies to collect and process information about vacancies published on various sites.

Users can customize search parameters such as keywords, city, experience level, salary, and other criteria, and get relevant results in real time. The service also provides the ability to save and track the vacancies of interest, create notifications about new vacancies, and compare different offers by various parameters.

In addition, a service for parsing vacancies for IT specialists can be useful not only for finding a job, but also for analyzing the labor market as a whole. By collecting and analyzing data on vacancies and candidate requirements, the service can help IT professionals identify the most in-demand skills and technologies, as well as predict industry trends.

As a result, the service for parsing vacancies for IT specialists is an important tool for modern workers and employers in the IT industry, which can significantly speed up the job search process and improve the efficiency of labor market analysis.

Each of these key fields will be used to search for and filter vacancies, as well as to generate reports and analyze job market data. For example, the user can select the desired position and city to find a suitable vacancy. Alternatively, the user can analyze candidate requirements data and highlight the skills and technologies that are most in demand.

## Entities on which the database will be based:

1. Vacancy - represent open positions in the company with fields:
* vacancy_id (unique job ID) serial  
* job_title (job title) string
* vacancy_description (vacancy description) text
* salary (salary) int
* experience_level (experience level) string
* type_of_employment (type of employment) string
* job_posting_date (job posting date) date
* job_status (job status: open/closed) boolean

The essence of the vacancy will be associated with candidates, many candidates can apply for one vacancy, just as one candidate can apply for several vacancies.
If there is a vacancy, then it should be tied to a specific team, but also the team may not need an employee/employees.

2. Candidate - potential job seekers who apply for vacancies, with fields:
* job_title (job title)  string
* years_of_experience (experience in years) int
* category (position category) string
* description (candidate description) text

To simplify the database, we will create a special **person** table, to which a specific role will be attached, one person - one role (XOR for Person)
If a candidate exists, it will be in the **person** table anyway.

3. Person - represent the people associated with the Candidate and Employee entities, with the fields:
* person_id (unique identifier of a person) serial
* name (name) string
* surname (surname) string
* tel (telephone) varchar(50)
* mail (e-mail) varchar(70)

The **person** table has relationships with the **candidate**, the **employee**, and the **address** where the person lives.
And if there is a person/candidate/employee, there will definitely be data on residence for him.

4. Employee - people who work for the team, with fields:
* job_title (job title) string
* work_tel (work phone) varchar(50)
* work_mail (work email) varchar(70)

The person whose data is in the **person** table works for a specific **team**, for which they can look for a **candidate**.

5. Team - groups of employees working on specific projects, with fields:
* team_id (unique team ID) serial 
* team_name (team name) string
* team_description (team description) text

We assume that in our **team** table there may be a team that does not need **candidates** (**employees**), the team works for a specific **company**.
If there is a team, then there is a company and if there is a company, then there is a team.
Certain **employees** work/work for the **team**, and if there is a  **team**, then there must be an **employee** of this **team**, and vice versa.

6. Company - organizations that use the system, with fields:
* company_id (unique company ID) serial
* company_name (company name) string
* company_description (company description) text

Suppose a company cannot operate without a team(s) (read above) and also the company must be located somewhere, for this data there is a table of **address** to which the company table and the **person** table refer.

7. Address - the location of the company's offices and the residence of people associated with the system, with the fields:
* address_id (unique address identifier) serial
* address_type (address type: work/home) string
* country (country) string
* city (city) string
* street (street) string
* house (house) int
* index (zip code) int

**Address** table, which contains data about the addresses of **companies** and addresses of **employees/candidates**.

8. Certification - represents any professional certifications held by a candidate, with fields:
* certification_id (unique certification identifier) serial
* certification_name (name of the certification) string
* certifying_organization (organization that issued the certification) string
* certification_date (date the certification was obtained) date

The **Certification** table is associated with the **Candidate** table and represents a list of professional certifications held by a candidate. **Candidates** can hold multiple certifications, and each certification can be held by multiple candidates.

9. City - represents a city with fields:
* id_city (unique city ID) serial
* city_name (city name) string

The **City** table represents a list of cities that are referenced by the **Address** table.

10. Country - represents a country with fields:
* id_country (unique country ID) serial
* country_name (country name) string

The **Country** table represents a list of countries that are referenced by the **Address** table.

# Conceptual schema

<img width="1019" alt="Screenshot 2023-04-12 at 18 34 09" src="https://user-images.githubusercontent.com/30218257/231523447-54de2145-b6bb-45ad-8cff-d9f5f4ee9f60.png">

# Loop discussion

In our conceptual scheme, **3** loop discussions are created:

1. **Vacancy** -> **Team** -> **Company** -> **Address** -> **Person** -> **Candidate** -> **Vacancy**;

2. **Team** -> **Company** -> **Address** -> **Person** -> **Employee** -> **Team**;

Both the first and second loop create the same problem and **person** and **company** can have the same **address**, but we solved this problem through the XOR operator.

3. **Person** -> **Employee** -> **Team** -> **Vacancy** -> **Candidate** -> **Person**.

This loop does not pose a problem, because we assume that the **employee** can work at the moment but also view other **vacancies** in the market.

# Relational schema (:construction: in process)

# Create script (:construction: in process)

# Insert script (:construction: in process)

# Queries

#### Select all employees working in the artificial intelligence team;

#### Select employees working in the company starting with the letter "A";

#### Select from the table with candidates those whose experience is more than 2 years;

#### Select to me the vacancies for which a person with id 213 applied and for which no one else applied; ![](https://img.shields.io/badge/-D4-blue)

#### Select me all the vacancies for which all candidates applied; ![](https://img.shields.io/badge/-D5-blue)

#### Select company_name and company_description from the Company table where company_id is 5;

#### Select job_title and experience_level from the Vacancy table where salary is greater than 50000;

#### Select job_title and salary from the Vacancy table where job_status is open;

#### Select all fields from the Vacancy and Candidate tables where the vacancy is related to a specific candidate with experience greater than 3 years and salary greater than 50000;

#### Select job_title and work_mail from the Employee table where job_title is 'Manager';

## Categories covered by queries

|  Category  | Description | Covered by |
|---|---|---|
| CN | C (NATURAL) - Select only those related to... | ![](https://img.shields.io/badge/-D4-blue) |
| D1N | D1 (NATURAL) - Select all related to - universal quantification query | ![](https://img.shields.io/badge/-D5-blue) |

# Conclusion (:construction: in process)

# References (:construction: in process)

# Iteration Score:

|  The first iteration  | The second iteration | The third iteration |
|---|---|---|
| <img width="375" alt="Screenshot 2023-04-12 at 18 35 35" src="https://user-images.githubusercontent.com/30218257/231523760-77783389-cb77-4cf0-947b-d08c69468051.png"> | <img width="373" alt="Screenshot 2023-04-12 at 18 36 26" src="https://user-images.githubusercontent.com/30218257/231523959-e47c1ebf-a43f-439b-a5ea-c555b3c7523f.png"> | (:construction: in process) |




