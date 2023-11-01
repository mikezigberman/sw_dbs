# Job title: Service for parsing vacancies for IT specialists 

## 📃 Content

- Description of semester work ✅
- Conceptual schema ✅
- Loop discussion ✅
- Relational schema ✅
- Create script ✅
- Insert script ✅
- Queries ✅
- Categories covered by queries ✅
    - Category ✅	
    - Description ✅
    - Covered by ✅
- References ✅
- Iteration Score ✅

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

# Relational schema 

<img width="1095" alt="original" src="https://user-images.githubusercontent.com/30218257/234678715-0244f5d1-265e-4744-a88d-7ee42f11fa2b.png">

# Create script

```
-- odeberu pokud existuje funkce na oodebrání tabulek a sekvencí
DROP FUNCTION IF EXISTS remove_all();

-- vytvořím funkci která odebere tabulky a sekvence
-- chcete také umět psát PLSQL? Zapište si předmět BI-SQL ;-)
CREATE or replace FUNCTION remove_all() RETURNS void AS $$
DECLARE
    rec RECORD;
    cmd text;
BEGIN
    cmd := '';

    FOR rec IN SELECT
            'DROP SEQUENCE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace
        WHERE
            relkind = 'S' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    FOR rec IN SELECT
            'DROP TABLE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace WHERE relkind = 'r' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    EXECUTE cmd;
    RETURN;
END;
$$ LANGUAGE plpgsql;
-- zavolám funkci co odebere tabulky a sekvence - Mohl bych dropnout celé 
-- schéma a znovu jej vytvořit, použíjeme však PLSQL
select remove_all();
-- odeberu pokud existuje funkce na oodebrání tabulek a sekvencí
DROP FUNCTION IF EXISTS remove_all();

-- vytvořím funkci která odebere tabulky a sekvence
-- chcete také umět psát PLSQL? Zapište si předmět BI-SQL ;-)
CREATE or replace FUNCTION remove_all() RETURNS void AS $$
DECLARE
    rec RECORD;
    cmd text;
BEGIN
    cmd := '';

    FOR rec IN SELECT
            'DROP SEQUENCE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace
        WHERE
            relkind = 'S' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    FOR rec IN SELECT
            'DROP TABLE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace WHERE relkind = 'r' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    EXECUTE cmd;
    RETURN;
END;
$$ LANGUAGE plpgsql;
-- zavolám funkci co odebere tabulky a sekvence - Mohl bych dropnout celé 
-- schéma a znovu jej vytvořit, použíjeme však PLSQL
select remove_all();

CREATE TABLE address (
    address_id SERIAL NOT NULL,
    id_city INTEGER NOT NULL,
    street VARCHAR(80) NOT NULL,
    house DECIMAL(10, 0) NOT NULL,
    index DECIMAL(10, 0) NOT NULL
);
ALTER TABLE address ADD CONSTRAINT pk_address PRIMARY KEY (address_id);

CREATE TABLE candidate (
    person_id INTEGER NOT NULL,
    job_title VARCHAR(256) NOT NULL,
    years_of_experience INTEGER NOT NULL,
    category VARCHAR(256) NOT NULL,
    description TEXT NOT NULL
);
ALTER TABLE candidate ADD CONSTRAINT pk_candidate PRIMARY KEY (person_id);

CREATE TABLE certification (
    certification_id SERIAL NOT NULL,
    certification_name VARCHAR(256) NOT NULL,
    certifying_organization VARCHAR(256) NOT NULL,
    certification_date VARCHAR(256) NOT NULL
);
ALTER TABLE certification ADD CONSTRAINT pk_certification PRIMARY KEY 
(certification_id);

CREATE TABLE city (
    id_city SERIAL NOT NULL,
    id_country INTEGER NOT NULL,
    city_name VARCHAR(256) NOT NULL
);
ALTER TABLE city ADD CONSTRAINT pk_city PRIMARY KEY (id_city);

CREATE TABLE company (
    company_id SERIAL NOT NULL,
    address_id INTEGER NOT NULL,
    company_name VARCHAR(70) NOT NULL,
    company_description TEXT
);
ALTER TABLE company ADD CONSTRAINT pk_company PRIMARY KEY (company_id);
ALTER TABLE company ADD CONSTRAINT u_fk_company_address UNIQUE 
(address_id);

CREATE TABLE country (
    id_country SERIAL NOT NULL,
    country_name VARCHAR(256) NOT NULL
);
ALTER TABLE country ADD CONSTRAINT pk_country PRIMARY KEY (id_country);

CREATE TABLE employee (
    person_id INTEGER NOT NULL,
    job_title CHAR(70) NOT NULL,
    work_tel TEXT,
    work_mail VARCHAR(256)
);
ALTER TABLE employee ADD CONSTRAINT pk_employee PRIMARY KEY (person_id);

CREATE TABLE person (
    person_id SERIAL NOT NULL,
    address_id INTEGER NOT NULL,
    name VARCHAR(256) NOT NULL,
    surname VARCHAR(256) NOT NULL,
    tel TEXT NOT NULL,
    mail VARCHAR(256) NOT NULL
);
ALTER TABLE person ADD CONSTRAINT pk_person PRIMARY KEY (person_id);
ALTER TABLE person ADD CONSTRAINT u_fk_person_address UNIQUE (address_id);

CREATE TABLE team (
    team_id SERIAL NOT NULL,
    person_id INTEGER NOT NULL,
    company_id INTEGER NOT NULL,
    team_name VARCHAR(70) NOT NULL,
    command_description TEXT NOT NULL
);
ALTER TABLE team ADD CONSTRAINT pk_team PRIMARY KEY (team_id);

CREATE TABLE vacancy (
    vacancy_id SERIAL NOT NULL,
    team_id INTEGER NOT NULL,
    job_title CHAR(128) NOT NULL,
    vacancy_description TEXT,
    salary money,
    experience_level VARCHAR(256),
    type_of_employment VARCHAR(256) NOT NULL,
    job_posting_date DATE NOT NULL,
    job_status VARCHAR(10) NOT NULL
);
ALTER TABLE vacancy ADD CONSTRAINT pk_vacancy PRIMARY KEY (vacancy_id);

CREATE TABLE certification_candidate (
    certification_id INTEGER NOT NULL,
    person_id INTEGER NOT NULL
);
ALTER TABLE certification_candidate ADD CONSTRAINT 
pk_certification_candidate PRIMARY KEY (certification_id, person_id);

CREATE TABLE vacancy_candidate (
    vacancy_id INTEGER NOT NULL,
    person_id INTEGER NOT NULL
);
ALTER TABLE vacancy_candidate ADD CONSTRAINT pk_vacancy_candidate PRIMARY 
KEY (vacancy_id, person_id);

ALTER TABLE address ADD CONSTRAINT fk_address_city FOREIGN KEY (id_city) 
REFERENCES city (id_city) ON DELETE CASCADE;

ALTER TABLE candidate ADD CONSTRAINT fk_candidate_person FOREIGN KEY 
(person_id) REFERENCES person (person_id) ON DELETE CASCADE;

ALTER TABLE city ADD CONSTRAINT fk_city_country FOREIGN KEY (id_country) 
REFERENCES country (id_country) ON DELETE CASCADE;

ALTER TABLE company ADD CONSTRAINT fk_company_address FOREIGN KEY 
(address_id) REFERENCES address (address_id) ON DELETE CASCADE;

ALTER TABLE employee ADD CONSTRAINT fk_employee_person FOREIGN KEY 
(person_id) REFERENCES person (person_id) ON DELETE CASCADE;

ALTER TABLE person ADD CONSTRAINT fk_person_address FOREIGN KEY 
(address_id) REFERENCES address (address_id) ON DELETE CASCADE;

ALTER TABLE team ADD CONSTRAINT fk_team_employee FOREIGN KEY (person_id) 
REFERENCES employee (person_id) ON DELETE CASCADE;
ALTER TABLE team ADD CONSTRAINT fk_team_company FOREIGN KEY (company_id) 
REFERENCES company (company_id) ON DELETE CASCADE;

ALTER TABLE vacancy ADD CONSTRAINT fk_vacancy_team FOREIGN KEY (team_id) 
REFERENCES team (team_id) ON DELETE CASCADE;

ALTER TABLE certification_candidate ADD CONSTRAINT 
fk_certification_candidate_cert FOREIGN KEY (certification_id) REFERENCES 
certification (certification_id) ON DELETE CASCADE;
ALTER TABLE certification_candidate ADD CONSTRAINT 
fk_certification_candidate_cand FOREIGN KEY (person_id) REFERENCES 
candidate (person_id) ON DELETE CASCADE;

ALTER TABLE vacancy_candidate ADD CONSTRAINT fk_vacancy_candidate_vacancy 
FOREIGN KEY (vacancy_id) REFERENCES vacancy (vacancy_id) ON DELETE 
CASCADE;
ALTER TABLE vacancy_candidate ADD CONSTRAINT 
fk_vacancy_candidate_candidate FOREIGN KEY (person_id) REFERENCES 
candidate (person_id) ON DELETE CASCADE;
```

# Insert script 

```
-- smazání všech záznamů z tabulek

CREATE or replace FUNCTION clean_tables() RETURNS void AS $$
declare
  l_stmt text;
begin
  select 'truncate ' || string_agg(format('%I.%I', schemaname, tablename) , ',')
    into l_stmt
  from pg_tables
  where schemaname in ('public');

  execute l_stmt || ' cascade';
end;
$$ LANGUAGE plpgsql;
select clean_tables();

-- reset sekvenci

CREATE or replace FUNCTION restart_sequences() RETURNS void AS $$
DECLARE
i TEXT;
BEGIN
 FOR i IN (SELECT column_default FROM information_schema.columns WHERE column_default SIMILAR TO 'nextval%')
  LOOP
         EXECUTE 'ALTER SEQUENCE'||' ' || substring(substring(i from '''[a-z_]*')from '[a-z_]+') || ' '||' RESTART 1;';
  END LOOP;
END $$ LANGUAGE plpgsql;
select restart_sequences();
-- konec resetu

-- konec mazání
-- mohli bchom použít i jednotlivé příkazy truncate na každo tabulku

-- table country

insert into country (id_country, country_name) values (1, 'Sweden');
insert into country (id_country, country_name) values (2, 'Canada');
insert into country (id_country, country_name) values (3, 'Romania');
...
insert into country (id_country, country_name) values (197, 'Russia');

-- end of table country

-- city tables

insert into city (id_city, id_country, city_name) values (1, 44, 'Orléans');
insert into city (id_city, id_country, city_name) values (2, 8, 'Gaizhou');
insert into city (id_city, id_country, city_name) values (3, 157, 'Gualeguay');
...
insert into city (id_city, id_country, city_name) values (500, 194, 'Ishimbay');

-- end of table cities

-- address table

insert into address (address_id, id_city, street, house, index) values (1, 1, 'Oriole', '4', '400830');
insert into address (address_id, id_city, street, house, index) values (2, 2, 'Buhler', '86859', '090038');
insert into address (address_id, id_city, street, house, index) values (3, 3, 'Crowley', '4167', '246313');
...
insert into address (address_id, id_city, street, house, index) values (500, 500, 'Anthes', '111', '328029');

-- end of table address

-- table person

insert into person (person_id, address_id, name, surname, tel, mail) values (1, 1, 'Ilsa', 'Longina', '+52 (160) 102-0126', 'ilongina0@gmpg.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (2, 2, 'Chalmers', 'Brokenshaw', '+58 (432) 882-0277', 'cbrokenshaw1@sakura.ne.jp');
insert into person (person_id, address_id, name, surname, tel, mail) values (3, 3, 'Aidan', 'Brassington', '+86 (119) 643-0783', 'abrassington2@shutterfly.com');
...
insert into person (person_id, address_id, name, surname, tel, mail) values (500, 500, 'Levin', 'Leyre', '+34 (205) 408-5660', 'lleyredv@yelp.com');

-- end of table persons

-- employee table

insert into employee (person_id, job_title, work_tel, work_mail) values (1, 'Financial Advisor', '+55 (330) 423-5956', 'orowcliffe0@mtv.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (2, 'Occupational Therapist', '+355 (664) 170-8373', 'eoselton1@epa.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (3, 'Nuclear Power Engineer', '+62 (537) 789-4872', 'npetrelli2@spotify.com');
...
insert into employee (person_id, job_title, work_tel, work_mail) values (500, 'Chief Design Engineer', '+212 (753) 408-4515', 'askyramdv@youtu.be');

-- end of table employees

-- candidate table

insert into candidate (person_id, job_title, years_of_experience, category, description) values (1, 'Engineer III', 3, 'Maecenas rhoncus aliquam lacus.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (2, 'Accountant IV', 10, 'Donec posuere metus vitae ipsum.', 'Morbi porttitor lorem id ligula.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (3, 'Structural Analysis Engineer', 1, 'Nulla facilisi.', 'Aliquam erat volutpat. In congue.');
...
insert into candidate (person_id, job_title, years_of_experience, category, description) values (500, 'Paralegal', 18, 'Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.', 'Nullam molestie nibh in lectus.');

-- end of candidate table

-- table certification

insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (1, 'ESLT', 'Skinder', '01/07/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (2, 'BEP', 'Ntags', '27/01/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (3, 'DD^B', 'Vidoo', '05/02/2022');
...
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (500, 'NTRS', 'Yakitri', '28/10/2022');

-- end of table certificates

-- company table

insert into company (company_id, address_id, company_name, company_description) values (1, 1, 'Jabbersphere', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.');
insert into company (company_id, address_id, company_name, company_description) values (2, 2, 'Skiptube', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Duis faucibus accumsan odio. Curabitur convallis.');
insert into company (company_id, address_id, company_name, company_description) values (3, 3, 'Realmix', 'Quisque ut erat. Curabitur gravida nisi at nibh.');
...
insert into company (company_id, address_id, company_name, company_description) values (500, 500, 'Vimbo', 'Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat.');

-- end of company table

-- team table

insert into team (team_id, person_id, company_id, team_name, command_description) values (1, 1, 1, 'Product Management', 'Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (2, 2, 2, 'Engineering', 'Nulla suscipit ligula in lacus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (3, 3, 3, 'Accounting', 'Integer a nibh.');
...
insert into team (team_id, person_id, company_id, team_name, command_description) values (500, 500, 500, 'Product Management', 'Fusce posuere felis sed lacus. Morbi sem mauris, laoreet ut, rhoncus aliquet, pulvinar sed, nisl.');

-- command table end

-- vacancy table

insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (1, 1, 'Actuary', 'Quisque arcu libero, rutrum ac, lobortis vel, dapibus at, diam.', '$272899.56', 3, 'Contract', '2022/02/12', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (2, 2, 'Chief Design Engineer', 'Suspendisse accumsan tortor quis turpis. Sed ante. Vivamus tortor. Duis mattis egestas metus.', '$437508.54', 18, 'Part-time', '2020/04/29', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (3, 3, 'Social Worker', 'Nunc rhoncus dui vel sem. Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus.', '$775156.73', 8, 'Full-time', '2020/10/27', 'Close');
...
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (500, 500, 'Chemical Engineer', 'Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia. Aenean sit amet justo.', '$99831.73', 16, 'Part-time', '2021/10/11', 'Close');

-- end of vacancy table

-- vacancy_candidate table

insert into vacancy_candidate (vacancy_id, person_id) values (1, 1);
insert into vacancy_candidate (vacancy_id, person_id) values (2, 2);
insert into vacancy_candidate (vacancy_id, person_id) values (3, 3);
...
insert into vacancy_candidate (vacancy_id, person_id) values (500, 500);

-- end of vacancy_candidate table

-- certification_candidate table

insert into certification_candidate (certification_id, person_id) values (1, 1);
insert into certification_candidate (certification_id, person_id) values (2, 2);
insert into certification_candidate (certification_id, person_id) values (3, 3);
...
insert into certification_candidate (certification_id, person_id) values (500, 500);

-- end of certification_candidate table

```

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

# References

**Database Design Books**:
1. [PostgreSQL MADE EASY: A Beginner's Handbook to easily Learn PostgreSQL](https://www.amazon.com/PostgreSQL-MADE-EASY-Beginners-Programming-ebook/dp/B095V4L3WK/ref=sr_1_25?keywords=postgresql&qid=1680971337&sprefix=Postgre%2Caps%2C214&sr=8-25) by MAGIGE ROBI (Author) 
2. [Beginning Databases with PostgreSQL: From Novice to Professional (Beginning From Novice to Professional) ](https://www.amazon.com/Beginning-Databases-PostgreSQL-Novice-Professional/dp/1590594789/ref=sr_1_9?keywords=postgresql&qid=1680971197&sprefix=Postgre%2Caps%2C214&sr=8-9) by Richard Stones (Author), Neil Matthew (Author)
3. [Learn PostgreSQL: Build and manage high-performance database solutions using PostgreSQL 12 and 13](https://www.amazon.com/Learn-PostgreSQL-high-performance-database-solutions/dp/183898528X/ref=sr_1_5?keywords=postgresql&qid=1680971197&sprefix=Postgre%2Caps%2C214&sr=8-5) by Luca Ferrari (Author), Enrico Pirozzi (Author) 
4. [The Manga Guide to Databases](https://www.amazon.com/Manga-Guide-Databases-Mana-Takahashi/dp/1593271905/ref=sr_1_1?crid=1E7VI7OODCWF8&keywords=Manga+databases&qid=1680970579&sprefix=manga+databases%2Caps%2C235&sr=8-1) by Mana Takahashi (Author), Shoko Azuma (Author), Co Ltd Trend (Author)

**Online courses and video tutorials**:
1. "Introduction to Relational Databases" on [Coursera](https://www.coursera.org/learn/introduction-to-relational-databases)
2. "Fundamentals of Database Engineering" at [Udemy](https://www.udemy.com/course/database-engines-crash-course/)
3. "Databases: Relational Databases and SQL" on [edX](https://www.edx.org/course/databases-5-sql?index=product&queryID=1731b8f95475bcb368ec331588d3cb12&position=4&search_index=product&results_level=first-level-results&campaign=Databases%3A+Relational+Databases+and+SQL&source=edX&product_category=course&placement_url=https%3A%2F%2Fwww.edx.org%2Fsearch)


**Websites and blogs**:
1. 20 Database Design Best Practices at [codebalance.blogspot.com](http://codebalance.blogspot.com/2011/07/20-database-design-best-practices.html)
2. 11 important database designing rules which I follow at [codeproject.com](https://www.codeproject.com/Articles/359654/11-important-database-designing-rules-which-I-fo-2)
3. Guide To Design Database For Blog Management In MySQL at [mysql.tutorials24x7.com](https://mysql.tutorials24x7.com/blog/guide-to-design-a-database-for-blog-management-in-mysql)

# Iteration Score:

|  The first iteration  | The second iteration | The third iteration |
|---|---|---|
| <img width="490" alt="Screenshot 2023-11-01 at 3 59 54" src="https://github.com/mikezigberman/sw_dbs/assets/30218257/dfec47fd-a80d-40db-9b18-c52007f462ba"> | <img width="488" alt="Screenshot 2023-11-01 at 4 00 23" src="https://github.com/mikezigberman/sw_dbs/assets/30218257/b09f9659-dea4-4e19-b14b-4a35c5b72fa9"> | <img width="489" alt="Screenshot 2023-11-01 at 4 00 50" src="https://github.com/mikezigberman/sw_dbs/assets/30218257/9d8c9021-fdf4-4711-8c3a-ca6a96ae527f"> |
