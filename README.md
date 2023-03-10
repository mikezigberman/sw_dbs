# Job title: Service for parsing vacancies for IT specialists 

## 📃 Content

- Description of semester work ✅
- Conceptual schema ✅
- Loop discussion ✅
- Relational schema (:construction: in process)
- Create script (:construction: in process)
- Insert script (:construction: in process)
- Queries (:construction: in process)
- Categories covered by queries (:construction: in process)
    - Category	
    - Description	
    - Covered by
- Conclusion (:construction: in process)
- References (:construction: in process)

# Description

In the modern world, where technological progress and the development of the IT industry are at their peak, finding a job for IT specialists is becoming increasingly important. However, finding the right job can take a lot of time and effort, especially when there are many options on the job market.

Job scraping service for IT professionals is an innovative solution that allows you to automate the search for suitable vacancies and significantly reduce the time spent searching for a job. This service uses machine learning and data analysis technologies to collect and process information about vacancies published on various sites.

Users can customize search parameters such as keywords, city, experience level, salary, and other criteria, and get relevant results in real time. The service also provides the ability to save and track the vacancies of interest, create notifications about new vacancies, and compare different offers by various parameters.

In addition, a service for parsing vacancies for IT specialists can be useful not only for finding a job, but also for analyzing the labor market as a whole. By collecting and analyzing data on vacancies and candidate requirements, the service can help IT professionals identify the most in-demand skills and technologies, as well as predict industry trends.

As a result, the service for parsing vacancies for IT specialists is an important tool for modern workers and employers in the IT industry, which can significantly speed up the job search process and improve the efficiency of labor market analysis.

## Key fields on the basis of which the service database will be built:

1. Job title
2. Vacancy description
3. City
4. Region
5. Salary
6. Experience level
7. Category (for example, software development, testing, design, administration)
8. Type of employment (full-time, part-time, remote work)
9. Requirements for the candidate (skills, level of education, work experience)
10. Company (name, description, location, reviews)
11. Job posting date
12. Source (website where the job was posted)
13. Job status (open/closed)
14. Additional information (contact person, links to additional information about the vacancy).

Each of these key fields will be used to search for and filter vacancies, as well as to generate reports and analyze job market data. For example, the user can select the desired position and city to find a suitable vacancy. Alternatively, the user can analyze candidate requirements data and highlight the skills and technologies that are most in demand.

# Conceptual schema

<img width="977" alt="Screenshot 2023-03-05 at 20 01 51" src="https://user-images.githubusercontent.com/30218257/222980471-4bf9e8e2-72e5-434d-9ad3-ace35863ae11.png">

# Loop discussion

In our conceptual scheme, **3** loop discussions are created:

1. **Vacancy** -> **Team** -> **Company** -> **Address** -> **Person** -> **Candidate** -> **Vacancy**;

2. **Team** -> **Company** -> **Address** -> **Person** -> **Employee** -> **Team**;

Both the first and second loop create the same problem and **person** and **company** can have the same **address**, but we solved this problem through the XOR operator.

3. **Person** -> **Employee** -> **Team** -> **Vacancy** -> **Candidate** -> **Person**.

This loop does not pose a problem, because we assume that the **employee** can work at the moment but also view other **vacancies** in the market.
