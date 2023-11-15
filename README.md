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
    address_id SERIAL,
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
    certification_id SERIAL,
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
    company_id SERIAL,
    address_id INTEGER NOT NULL,
    company_name VARCHAR(70) NOT NULL,
    company_description TEXT
);
ALTER TABLE company ADD CONSTRAINT pk_company PRIMARY KEY (company_id);

CREATE TABLE country (
    id_country SERIAL,
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
    person_id SERIAL,
    address_id INTEGER NOT NULL,
    name VARCHAR(256) NOT NULL,
    surname VARCHAR(256) NOT NULL,
    tel TEXT NOT NULL,
    mail VARCHAR(256) NOT NULL
);
ALTER TABLE person ADD CONSTRAINT pk_person PRIMARY KEY (person_id);

CREATE TABLE team (
    team_id SERIAL,
    person_id INTEGER NOT NULL,
    company_id INTEGER,
    team_name VARCHAR(70) NOT NULL,
    command_description TEXT NOT NULL
);
ALTER TABLE team ADD CONSTRAINT pk_team PRIMARY KEY (team_id);

CREATE TABLE vacancy (
    vacancy_id SERIAL,
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
    vacancy_id INTEGER,
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
insert into country (id_country, country_name) values (4, 'Palestinian Territory');
insert into country (id_country, country_name) values (5, 'Brazil');
insert into country (id_country, country_name) values (6, 'Mongolia');
insert into country (id_country, country_name) values (7, 'Russia');
insert into country (id_country, country_name) values (8, 'Indonesia');
insert into country (id_country, country_name) values (9, 'France');
insert into country (id_country, country_name) values (10, 'China');
insert into country (id_country, country_name) values (11, 'China');
insert into country (id_country, country_name) values (12, 'Lithuania');
insert into country (id_country, country_name) values (13, 'Poland');
insert into country (id_country, country_name) values (14, 'Bulgaria');
insert into country (id_country, country_name) values (15, 'Indonesia');
insert into country (id_country, country_name) values (16, 'United States');
insert into country (id_country, country_name) values (17, 'France');
insert into country (id_country, country_name) values (18, 'China');
insert into country (id_country, country_name) values (19, 'Uzbekistan');
insert into country (id_country, country_name) values (20, 'Indonesia');
insert into country (id_country, country_name) values (21, 'Portugal');
insert into country (id_country, country_name) values (22, 'China');
insert into country (id_country, country_name) values (23, 'South Africa');
insert into country (id_country, country_name) values (24, 'Peru');
insert into country (id_country, country_name) values (25, 'Slovenia');
insert into country (id_country, country_name) values (26, 'China');
insert into country (id_country, country_name) values (27, 'Mexico');
insert into country (id_country, country_name) values (28, 'Poland');
insert into country (id_country, country_name) values (29, 'Japan');
insert into country (id_country, country_name) values (30, 'Greece');
insert into country (id_country, country_name) values (31, 'Russia');
insert into country (id_country, country_name) values (32, 'United States');
insert into country (id_country, country_name) values (33, 'Honduras');
insert into country (id_country, country_name) values (34, 'Luxembourg');
insert into country (id_country, country_name) values (35, 'Argentina');
insert into country (id_country, country_name) values (36, 'Russia');
insert into country (id_country, country_name) values (37, 'France');
insert into country (id_country, country_name) values (38, 'Bulgaria');
insert into country (id_country, country_name) values (39, 'South Africa');
insert into country (id_country, country_name) values (40, 'China');
insert into country (id_country, country_name) values (41, 'China');
insert into country (id_country, country_name) values (42, 'Myanmar');
insert into country (id_country, country_name) values (43, 'Democratic Republic of the Congo');
insert into country (id_country, country_name) values (44, 'South Africa');
insert into country (id_country, country_name) values (45, 'Russia');
insert into country (id_country, country_name) values (46, 'Malta');
insert into country (id_country, country_name) values (47, 'Philippines');
insert into country (id_country, country_name) values (48, 'China');
insert into country (id_country, country_name) values (49, 'Greece');
insert into country (id_country, country_name) values (50, 'China');
insert into country (id_country, country_name) values (51, 'Russia');
insert into country (id_country, country_name) values (52, 'Indonesia');
insert into country (id_country, country_name) values (53, 'Thailand');
insert into country (id_country, country_name) values (54, 'Poland');
insert into country (id_country, country_name) values (55, 'China');
insert into country (id_country, country_name) values (56, 'Portugal');
insert into country (id_country, country_name) values (57, 'Canada');
insert into country (id_country, country_name) values (58, 'Vietnam');
insert into country (id_country, country_name) values (59, 'United States');
insert into country (id_country, country_name) values (60, 'Japan');
insert into country (id_country, country_name) values (61, 'China');
insert into country (id_country, country_name) values (62, 'Argentina');
insert into country (id_country, country_name) values (63, 'Russia');
insert into country (id_country, country_name) values (64, 'Vietnam');
insert into country (id_country, country_name) values (65, 'China');
insert into country (id_country, country_name) values (66, 'Finland');
insert into country (id_country, country_name) values (67, 'Bulgaria');
insert into country (id_country, country_name) values (68, 'Poland');
insert into country (id_country, country_name) values (69, 'France');
insert into country (id_country, country_name) values (70, 'Madagascar');
insert into country (id_country, country_name) values (71, 'Colombia');
insert into country (id_country, country_name) values (72, 'China');
insert into country (id_country, country_name) values (73, 'Sweden');
insert into country (id_country, country_name) values (74, 'Argentina');
insert into country (id_country, country_name) values (75, 'Poland');
insert into country (id_country, country_name) values (76, 'France');
insert into country (id_country, country_name) values (77, 'China');
insert into country (id_country, country_name) values (78, 'Brazil');
insert into country (id_country, country_name) values (79, 'France');
insert into country (id_country, country_name) values (80, 'Indonesia');
insert into country (id_country, country_name) values (81, 'Greece');
insert into country (id_country, country_name) values (82, 'Sweden');
insert into country (id_country, country_name) values (83, 'China');
insert into country (id_country, country_name) values (84, 'China');
insert into country (id_country, country_name) values (85, 'China');
insert into country (id_country, country_name) values (86, 'Netherlands');
insert into country (id_country, country_name) values (87, 'Guatemala');
insert into country (id_country, country_name) values (88, 'Thailand');
insert into country (id_country, country_name) values (89, 'Portugal');
insert into country (id_country, country_name) values (90, 'Brazil');
insert into country (id_country, country_name) values (91, 'China');
insert into country (id_country, country_name) values (92, 'Sweden');
insert into country (id_country, country_name) values (93, 'Indonesia');
insert into country (id_country, country_name) values (94, 'China');
insert into country (id_country, country_name) values (95, 'Portugal');
insert into country (id_country, country_name) values (96, 'Slovenia');
insert into country (id_country, country_name) values (97, 'China');
insert into country (id_country, country_name) values (98, 'Russia');
insert into country (id_country, country_name) values (99, 'China');
insert into country (id_country, country_name) values (100, 'France');
insert into country (id_country, country_name) values (101, 'China');
insert into country (id_country, country_name) values (102, 'Japan');
insert into country (id_country, country_name) values (103, 'Israel');
insert into country (id_country, country_name) values (104, 'New Zealand');
insert into country (id_country, country_name) values (105, 'Armenia');
insert into country (id_country, country_name) values (106, 'Ukraine');
insert into country (id_country, country_name) values (107, 'Indonesia');
insert into country (id_country, country_name) values (108, 'China');
insert into country (id_country, country_name) values (109, 'Spain');
insert into country (id_country, country_name) values (110, 'Ecuador');
insert into country (id_country, country_name) values (111, 'Croatia');
insert into country (id_country, country_name) values (112, 'Russia');
insert into country (id_country, country_name) values (113, 'France');
insert into country (id_country, country_name) values (114, 'Uganda');
insert into country (id_country, country_name) values (115, 'Czech Republic');
insert into country (id_country, country_name) values (116, 'Honduras');
insert into country (id_country, country_name) values (117, 'Brazil');
insert into country (id_country, country_name) values (118, 'Brazil');
insert into country (id_country, country_name) values (119, 'Greece');
insert into country (id_country, country_name) values (120, 'China');
insert into country (id_country, country_name) values (121, 'Indonesia');
insert into country (id_country, country_name) values (122, 'Philippines');
insert into country (id_country, country_name) values (123, 'South Africa');
insert into country (id_country, country_name) values (124, 'Benin');
insert into country (id_country, country_name) values (125, 'United States');
insert into country (id_country, country_name) values (126, 'Indonesia');
insert into country (id_country, country_name) values (127, 'Philippines');
insert into country (id_country, country_name) values (128, 'China');
insert into country (id_country, country_name) values (129, 'Nigeria');
insert into country (id_country, country_name) values (130, 'South Africa');
insert into country (id_country, country_name) values (131, 'China');
insert into country (id_country, country_name) values (132, 'Indonesia');
insert into country (id_country, country_name) values (133, 'Sri Lanka');
insert into country (id_country, country_name) values (134, 'Kazakhstan');
insert into country (id_country, country_name) values (135, 'Indonesia');
insert into country (id_country, country_name) values (136, 'Bosnia and Herzegovina');
insert into country (id_country, country_name) values (137, 'Greece');
insert into country (id_country, country_name) values (138, 'Poland');
insert into country (id_country, country_name) values (139, 'Afghanistan');
insert into country (id_country, country_name) values (140, 'China');
insert into country (id_country, country_name) values (141, 'Greece');
insert into country (id_country, country_name) values (142, 'China');
insert into country (id_country, country_name) values (143, 'Cameroon');
insert into country (id_country, country_name) values (144, 'Sri Lanka');
insert into country (id_country, country_name) values (145, 'Portugal');
insert into country (id_country, country_name) values (146, 'Philippines');
insert into country (id_country, country_name) values (147, 'China');
insert into country (id_country, country_name) values (148, 'China');
insert into country (id_country, country_name) values (149, 'Poland');
insert into country (id_country, country_name) values (150, 'France');
insert into country (id_country, country_name) values (151, 'Russia');
insert into country (id_country, country_name) values (152, 'China');
insert into country (id_country, country_name) values (153, 'Portugal');
insert into country (id_country, country_name) values (154, 'China');
insert into country (id_country, country_name) values (155, 'Georgia');
insert into country (id_country, country_name) values (156, 'Pakistan');
insert into country (id_country, country_name) values (157, 'Netherlands');
insert into country (id_country, country_name) values (158, 'Nigeria');
insert into country (id_country, country_name) values (159, 'Bosnia and Herzegovina');
insert into country (id_country, country_name) values (160, 'Nigeria');
insert into country (id_country, country_name) values (161, 'Norway');
insert into country (id_country, country_name) values (162, 'Haiti');
insert into country (id_country, country_name) values (163, 'China');
insert into country (id_country, country_name) values (164, 'Argentina');
insert into country (id_country, country_name) values (165, 'Poland');
insert into country (id_country, country_name) values (166, 'Zimbabwe');
insert into country (id_country, country_name) values (167, 'Indonesia');
insert into country (id_country, country_name) values (168, 'Sweden');
insert into country (id_country, country_name) values (169, 'Portugal');
insert into country (id_country, country_name) values (170, 'Canada');
insert into country (id_country, country_name) values (171, 'China');
insert into country (id_country, country_name) values (172, 'Iran');
insert into country (id_country, country_name) values (173, 'Zimbabwe');
insert into country (id_country, country_name) values (174, 'China');
insert into country (id_country, country_name) values (175, 'Ireland');
insert into country (id_country, country_name) values (176, 'Ukraine');
insert into country (id_country, country_name) values (177, 'China');
insert into country (id_country, country_name) values (178, 'China');
insert into country (id_country, country_name) values (179, 'Indonesia');
insert into country (id_country, country_name) values (180, 'Indonesia');
insert into country (id_country, country_name) values (181, 'Sweden');
insert into country (id_country, country_name) values (182, 'Philippines');
insert into country (id_country, country_name) values (183, 'Norway');
insert into country (id_country, country_name) values (184, 'Indonesia');
insert into country (id_country, country_name) values (185, 'China');
insert into country (id_country, country_name) values (186, 'Canada');
insert into country (id_country, country_name) values (187, 'China');
insert into country (id_country, country_name) values (188, 'Indonesia');
insert into country (id_country, country_name) values (189, 'France');
insert into country (id_country, country_name) values (190, 'Iran');
insert into country (id_country, country_name) values (191, 'Honduras');
insert into country (id_country, country_name) values (192, 'Mexico');
insert into country (id_country, country_name) values (193, 'Brazil');
insert into country (id_country, country_name) values (194, 'Colombia');
insert into country (id_country, country_name) values (195, 'Mexico');
insert into country (id_country, country_name) values (196, 'China');
insert into country (id_country, country_name) values (197, 'Russia');

-- end of table country

-- city tables

insert into city (id_city, id_country, city_name) values (1, 44, 'Orléans');
insert into city (id_city, id_country, city_name) values (2, 8, 'Gaizhou');
insert into city (id_city, id_country, city_name) values (3, 157, 'Gualeguay');
insert into city (id_city, id_country, city_name) values (4, 112, 'Mas‘adah');
insert into city (id_city, id_country, city_name) values (5, 74, 'Wirral');
insert into city (id_city, id_country, city_name) values (6, 141, 'Pak Phanang');
insert into city (id_city, id_country, city_name) values (7, 158, 'Wanmu');
insert into city (id_city, id_country, city_name) values (8, 32, 'Ingå');
insert into city (id_city, id_country, city_name) values (9, 79, 'Unity');
insert into city (id_city, id_country, city_name) values (10, 91, 'Stockholm');
insert into city (id_city, id_country, city_name) values (11, 15, 'Kleszczewo');
insert into city (id_city, id_country, city_name) values (12, 187, 'Eastern Suburbs Mc');
insert into city (id_city, id_country, city_name) values (13, 169, 'General Luna');
insert into city (id_city, id_country, city_name) values (14, 48, 'Chenda');
insert into city (id_city, id_country, city_name) values (15, 153, 'Guanzhuang');
insert into city (id_city, id_country, city_name) values (16, 45, 'Sukomulyo');
insert into city (id_city, id_country, city_name) values (17, 110, 'Itaocara');
insert into city (id_city, id_country, city_name) values (18, 126, 'Caramuca');
insert into city (id_city, id_country, city_name) values (19, 167, 'Unden');
insert into city (id_city, id_country, city_name) values (20, 30, 'Lebak');
insert into city (id_city, id_country, city_name) values (21, 111, 'Bolotnoye');
insert into city (id_city, id_country, city_name) values (22, 195, 'Rovira');
insert into city (id_city, id_country, city_name) values (23, 56, 'Miguel Hidalgo');
insert into city (id_city, id_country, city_name) values (24, 103, 'Przystajń');
insert into city (id_city, id_country, city_name) values (25, 149, 'Dumlan');
insert into city (id_city, id_country, city_name) values (26, 55, 'Huaping');
insert into city (id_city, id_country, city_name) values (27, 2, 'Suwaru');
insert into city (id_city, id_country, city_name) values (28, 197, 'Shenavan');
insert into city (id_city, id_country, city_name) values (29, 29, 'Berlin');
insert into city (id_city, id_country, city_name) values (30, 75, 'Tongli');
insert into city (id_city, id_country, city_name) values (31, 174, 'Tuapse');
insert into city (id_city, id_country, city_name) values (32, 79, 'Oskarshamn');
insert into city (id_city, id_country, city_name) values (33, 181, 'Jambeyan');
insert into city (id_city, id_country, city_name) values (34, 73, 'Cool űrhajó');
insert into city (id_city, id_country, city_name) values (35, 34, 'Fort-de-France');
insert into city (id_city, id_country, city_name) values (36, 14, 'København');
insert into city (id_city, id_country, city_name) values (37, 51, 'Al Maḩwīt');
insert into city (id_city, id_country, city_name) values (38, 13, 'Huangsha');
insert into city (id_city, id_country, city_name) values (39, 127, 'Berezovo');
insert into city (id_city, id_country, city_name) values (40, 12, 'Jalanbaru');
insert into city (id_city, id_country, city_name) values (41, 44, 'Winong');
insert into city (id_city, id_country, city_name) values (42, 147, 'Novokuz’minki');
insert into city (id_city, id_country, city_name) values (43, 80, 'Calimita');
insert into city (id_city, id_country, city_name) values (44, 82, 'Santa Marta');
insert into city (id_city, id_country, city_name) values (45, 94, 'Al Ma‘baţlī');
insert into city (id_city, id_country, city_name) values (46, 5, 'Barrancas');
insert into city (id_city, id_country, city_name) values (47, 90, 'Nîmes');
insert into city (id_city, id_country, city_name) values (48, 16, 'Várzea');
insert into city (id_city, id_country, city_name) values (49, 97, 'Chivor');
insert into city (id_city, id_country, city_name) values (50, 12, 'Nikkō');
insert into city (id_city, id_country, city_name) values (51, 103, 'Wādī as Salqā');
insert into city (id_city, id_country, city_name) values (52, 146, 'Wenquan');
insert into city (id_city, id_country, city_name) values (53, 121, 'Järfälla');
insert into city (id_city, id_country, city_name) values (54, 107, 'Donghai');
insert into city (id_city, id_country, city_name) values (55, 24, 'Ningyang');
insert into city (id_city, id_country, city_name) values (56, 63, 'Planken');
insert into city (id_city, id_country, city_name) values (57, 191, 'Zhukovka');
insert into city (id_city, id_country, city_name) values (58, 193, 'Sokolov');
insert into city (id_city, id_country, city_name) values (59, 189, 'Koysinceq');
insert into city (id_city, id_country, city_name) values (60, 7, 'Fonte Boa da Brincosa');
insert into city (id_city, id_country, city_name) values (61, 50, 'Sumberbatas');
insert into city (id_city, id_country, city_name) values (62, 197, 'Bamban');
insert into city (id_city, id_country, city_name) values (63, 16, 'Diang');
insert into city (id_city, id_country, city_name) values (64, 165, 'Krasnoye');
insert into city (id_city, id_country, city_name) values (65, 153, 'Yakovlevo');
insert into city (id_city, id_country, city_name) values (66, 84, 'Västra Frölunda');
insert into city (id_city, id_country, city_name) values (67, 98, 'Tongxing');
insert into city (id_city, id_country, city_name) values (68, 150, 'Presidente Epitácio');
insert into city (id_city, id_country, city_name) values (69, 164, 'Samho-rodongjagu');
insert into city (id_city, id_country, city_name) values (70, 38, 'Chacarita');
insert into city (id_city, id_country, city_name) values (71, 170, 'Jaunpils');
insert into city (id_city, id_country, city_name) values (72, 6, 'Guangyuan');
insert into city (id_city, id_country, city_name) values (73, 51, 'Dorp Antriol');
insert into city (id_city, id_country, city_name) values (74, 77, 'Isidro Fabela');
insert into city (id_city, id_country, city_name) values (75, 21, 'Gradil');
insert into city (id_city, id_country, city_name) values (76, 37, 'New York City');
insert into city (id_city, id_country, city_name) values (77, 75, 'Cotuí');
insert into city (id_city, id_country, city_name) values (78, 166, 'Ballivor');
insert into city (id_city, id_country, city_name) values (79, 58, 'Uyskoye');
insert into city (id_city, id_country, city_name) values (80, 19, 'Sudimoro');
insert into city (id_city, id_country, city_name) values (81, 146, 'Jankowice');
insert into city (id_city, id_country, city_name) values (82, 78, 'Bakaran Kulon');
insert into city (id_city, id_country, city_name) values (83, 59, 'Kerema');
insert into city (id_city, id_country, city_name) values (84, 181, 'Tatuí');
insert into city (id_city, id_country, city_name) values (85, 92, 'Wuyanquan');
insert into city (id_city, id_country, city_name) values (86, 95, 'Noyemberyan');
insert into city (id_city, id_country, city_name) values (87, 5, 'Watuagung');
insert into city (id_city, id_country, city_name) values (88, 81, 'Boa Viagem');
insert into city (id_city, id_country, city_name) values (89, 68, 'Horgo');
insert into city (id_city, id_country, city_name) values (90, 34, 'Da’an');
insert into city (id_city, id_country, city_name) values (91, 134, 'Lyon');
insert into city (id_city, id_country, city_name) values (92, 143, 'Ambatolampy');
insert into city (id_city, id_country, city_name) values (93, 27, 'Bombardopolis');
insert into city (id_city, id_country, city_name) values (94, 99, 'Strängnäs');
insert into city (id_city, id_country, city_name) values (95, 148, 'Shanghuang');
insert into city (id_city, id_country, city_name) values (96, 140, 'Tejen');
insert into city (id_city, id_country, city_name) values (97, 7, 'Wichit');
insert into city (id_city, id_country, city_name) values (98, 111, 'Balad');
insert into city (id_city, id_country, city_name) values (99, 61, 'Néos Mylótopos');
insert into city (id_city, id_country, city_name) values (100, 12, 'La Jicaral');

insert into city (id_city, id_country, city_name) values (101, 91, 'Aqtobe');
insert into city (id_city, id_country, city_name) values (102, 102, 'Aqtau');
insert into city (id_city, id_country, city_name) values (103, 194, 'Brno');

-- end of table cities

-- address table

insert into address (address_id, id_city, street, house, index) values (1, 1, 'Oriole', '4', '400830');
insert into address (address_id, id_city, street, house, index) values (2, 2, 'Buhler', '86859', '090038');
insert into address (address_id, id_city, street, house, index) values (3, 3, 'Crowley', '4167', '246313');
insert into address (address_id, id_city, street, house, index) values (4, 4, 'Thierer', '4241', '329217');
insert into address (address_id, id_city, street, house, index) values (5, 5, 'Green', '74', '639994');
insert into address (address_id, id_city, street, house, index) values (6, 6, 'Kennedy', '850', '808382');
insert into address (address_id, id_city, street, house, index) values (7, 7, 'Nancy', '80112', '299989');
insert into address (address_id, id_city, street, house, index) values (8, 8, 'Hansons', '60409', '981637');
insert into address (address_id, id_city, street, house, index) values (9, 9, 'Amoth', '70498', '652116');
insert into address (address_id, id_city, street, house, index) values (10, 10, 'Lake View', '06', '574409');
insert into address (address_id, id_city, street, house, index) values (11, 11, 'Marcy', '1606', '813791');
insert into address (address_id, id_city, street, house, index) values (12, 12, 'Bay', '7', '283638');
insert into address (address_id, id_city, street, house, index) values (13, 13, 'Cordelia', '4919', '835738');
insert into address (address_id, id_city, street, house, index) values (14, 14, 'Jenna', '5382', '000920');
insert into address (address_id, id_city, street, house, index) values (15, 15, 'Stoughton', '484', '757424');
insert into address (address_id, id_city, street, house, index) values (16, 16, 'Hoffman', '987', '624275');
insert into address (address_id, id_city, street, house, index) values (17, 17, 'Magdeline', '34113', '904247');
insert into address (address_id, id_city, street, house, index) values (18, 18, 'Mayfield', '48984', '915717');
insert into address (address_id, id_city, street, house, index) values (19, 19, 'Caliangt', '1', '096969');
insert into address (address_id, id_city, street, house, index) values (20, 20, 'Acker', '98046', '000727');
insert into address (address_id, id_city, street, house, index) values (21, 21, 'Cottonwood', '5', '499489');
insert into address (address_id, id_city, street, house, index) values (22, 22, 'Morrow', '23332', '971830');
insert into address (address_id, id_city, street, house, index) values (23, 23, 'Garrison', '3227', '120432');
insert into address (address_id, id_city, street, house, index) values (24, 24, 'Prairie Rose', '046', '859291');
insert into address (address_id, id_city, street, house, index) values (25, 25, 'Division', '5', '917971');
insert into address (address_id, id_city, street, house, index) values (26, 26, 'Packers', '92372', '195779');
insert into address (address_id, id_city, street, house, index) values (27, 27, 'High Crossing', '1076', '530600');
insert into address (address_id, id_city, street, house, index) values (28, 28, 'Heffernan', '447', '655461');
insert into address (address_id, id_city, street, house, index) values (29, 29, 'Briar Crest', '044', '719828');
insert into address (address_id, id_city, street, house, index) values (30, 30, 'Bartelt', '1163', '541121');
insert into address (address_id, id_city, street, house, index) values (31, 31, 'Moulton', '7708', '407084');
insert into address (address_id, id_city, street, house, index) values (32, 32, 'Parkside', '505', '386548');
insert into address (address_id, id_city, street, house, index) values (33, 33, 'Ilene', '2311', '427891');
insert into address (address_id, id_city, street, house, index) values (34, 34, 'Myrtle', '933', '647123');
insert into address (address_id, id_city, street, house, index) values (35, 35, 'Scott', '481', '889476');
insert into address (address_id, id_city, street, house, index) values (36, 36, 'Gale', '245', '508180');
insert into address (address_id, id_city, street, house, index) values (37, 37, 'Caliangt', '30', '642217');
insert into address (address_id, id_city, street, house, index) values (38, 38, 'Rockefeller', '2045', '754215');
insert into address (address_id, id_city, street, house, index) values (39, 39, 'Boyd', '8', '381671');
insert into address (address_id, id_city, street, house, index) values (40, 40, 'Kinsman', '53', '222012');
insert into address (address_id, id_city, street, house, index) values (41, 41, 'Valley Edge', '47', '878880');
insert into address (address_id, id_city, street, house, index) values (42, 42, 'Dakota', '36', '555092');
insert into address (address_id, id_city, street, house, index) values (43, 43, 'East', '36929', '508091');
insert into address (address_id, id_city, street, house, index) values (44, 44, 'Kennedy', '99363', '966473');
insert into address (address_id, id_city, street, house, index) values (45, 45, 'Karstens', '21', '178917');
insert into address (address_id, id_city, street, house, index) values (46, 46, 'Talisman', '39862', '189175');
insert into address (address_id, id_city, street, house, index) values (47, 47, 'Saint Paul', '5', '059154');
insert into address (address_id, id_city, street, house, index) values (48, 48, 'Hoard', '320', '949888');
insert into address (address_id, id_city, street, house, index) values (49, 49, '8th', '7', '406772');
insert into address (address_id, id_city, street, house, index) values (50, 50, 'Union', '9', '488449');
insert into address (address_id, id_city, street, house, index) values (51, 51, 'Butterfield', '92291', '292265');
insert into address (address_id, id_city, street, house, index) values (52, 52, 'Northridge', '12590', '041115');
insert into address (address_id, id_city, street, house, index) values (53, 53, 'Schmedeman', '49', '400233');
insert into address (address_id, id_city, street, house, index) values (54, 54, 'Donald', '55668', '693185');
insert into address (address_id, id_city, street, house, index) values (55, 55, 'Graceland', '6', '809213');
insert into address (address_id, id_city, street, house, index) values (56, 56, 'Prairieview', '1', '617997');
insert into address (address_id, id_city, street, house, index) values (57, 57, 'Cordelia', '05057', '633685');
insert into address (address_id, id_city, street, house, index) values (58, 58, 'Loftsgordon', '34', '754483');
insert into address (address_id, id_city, street, house, index) values (59, 59, 'Larry', '3', '234205');
insert into address (address_id, id_city, street, house, index) values (60, 60, 'Delladonna', '5328', '453622');
insert into address (address_id, id_city, street, house, index) values (61, 61, 'Laurel', '73472', '217980');
insert into address (address_id, id_city, street, house, index) values (62, 62, 'Hollow Ridge', '9', '510277');
insert into address (address_id, id_city, street, house, index) values (63, 63, 'Onsgard', '896', '142422');
insert into address (address_id, id_city, street, house, index) values (64, 64, 'Clemons', '9', '647849');
insert into address (address_id, id_city, street, house, index) values (65, 65, 'Nancy', '15138', '232736');
insert into address (address_id, id_city, street, house, index) values (66, 66, 'Messerschmidt', '7082', '237202');
insert into address (address_id, id_city, street, house, index) values (67, 67, 'Coolidge', '3696', '352185');
insert into address (address_id, id_city, street, house, index) values (68, 68, 'Cherokee', '5', '237433');
insert into address (address_id, id_city, street, house, index) values (69, 69, 'Fremont', '6543', '910000');
insert into address (address_id, id_city, street, house, index) values (70, 70, 'Westend', '02', '208728');
insert into address (address_id, id_city, street, house, index) values (71, 71, 'Anzinger', '6', '731475');
insert into address (address_id, id_city, street, house, index) values (72, 72, 'Pond', '60007', '189061');
insert into address (address_id, id_city, street, house, index) values (73, 73, 'Sauthoff', '0810', '956532');
insert into address (address_id, id_city, street, house, index) values (74, 74, 'Melody', '0663', '154834');
insert into address (address_id, id_city, street, house, index) values (75, 75, 'Bayside', '262', '261545');
insert into address (address_id, id_city, street, house, index) values (76, 76, 'Red Cloud', '889', '538358');
insert into address (address_id, id_city, street, house, index) values (77, 77, 'Fordem', '5275', '717616');
insert into address (address_id, id_city, street, house, index) values (78, 78, 'Hallows', '06', '076928');
insert into address (address_id, id_city, street, house, index) values (79, 79, 'Warner', '11337', '532190');
insert into address (address_id, id_city, street, house, index) values (80, 80, 'Spaight', '49225', '465059');
insert into address (address_id, id_city, street, house, index) values (81, 81, 'Barnett', '103', '708979');
insert into address (address_id, id_city, street, house, index) values (82, 82, 'Jenna', '5969', '764517');
insert into address (address_id, id_city, street, house, index) values (83, 83, 'Forster', '068', '630733');
insert into address (address_id, id_city, street, house, index) values (84, 84, 'Darwin', '973', '414627');
insert into address (address_id, id_city, street, house, index) values (85, 85, 'Forster', '58659', '427325');
insert into address (address_id, id_city, street, house, index) values (86, 86, 'Sugar', '42', '496014');
insert into address (address_id, id_city, street, house, index) values (87, 87, 'Norway Maple', '59', '799889');
insert into address (address_id, id_city, street, house, index) values (88, 88, 'Service', '0832', '480301');
insert into address (address_id, id_city, street, house, index) values (89, 89, 'Knutson', '98294', '859936');
insert into address (address_id, id_city, street, house, index) values (90, 90, 'Pleasure', '5533', '541276');
insert into address (address_id, id_city, street, house, index) values (91, 91, 'Farwell', '17791', '158398');
insert into address (address_id, id_city, street, house, index) values (92, 92, 'Bayside', '1884', '296337');
insert into address (address_id, id_city, street, house, index) values (93, 93, 'Aberg', '120', '682708');
insert into address (address_id, id_city, street, house, index) values (94, 94, 'Center', '37422', '149155');
insert into address (address_id, id_city, street, house, index) values (95, 95, 'Orin', '58', '826604');
insert into address (address_id, id_city, street, house, index) values (96, 96, 'Blue Bill Park', '33790', '550381');
insert into address (address_id, id_city, street, house, index) values (97, 97, 'Sugar', '20549', '453381');
insert into address (address_id, id_city, street, house, index) values (98, 98, 'Debra', '4', '631522');
insert into address (address_id, id_city, street, house, index) values (99, 99, 'Sunnyside', '8637', '880215');
insert into address (address_id, id_city, street, house, index) values (100, 100, 'Scofield', '72', '880312');

insert into address (address_id, id_city, street, house, index) values (101, 101, 'Shaikenova', '43', '432013');
insert into address (address_id, id_city, street, house, index) values (102, 102, 'Praha', '67', '132800');
insert into address (address_id, id_city, street, house, index) values (103, 103, 'Jourey', '98', '123456');

-- end of table address

-- table person

insert into person (person_id, address_id, name, surname, tel, mail) values (1, 1, 'Ilsa', 'Longina', '+52 (160) 102-0126', 'ilongina0@gmpg.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (2, 2, 'Chalmers', 'Brokenshaw', '+58 (432) 882-0277', 'cbrokenshaw1@sakura.ne.jp');
insert into person (person_id, address_id, name, surname, tel, mail) values (3, 3, 'Aidan', 'Brassington', '+86 (119) 643-0783', 'abrassington2@shutterfly.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (4, 4, 'Analiese', 'Arney', '+7 (656) 208-0684', 'aarney3@apple.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (5, 5, 'Mikey', 'Honnan', '+62 (289) 422-4213', 'mhonnan4@ovh.net');
insert into person (person_id, address_id, name, surname, tel, mail) values (6, 6, 'Krisha', 'Alden', '+86 (421) 206-5845', 'kalden5@stanford.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (7, 7, 'Kattie', 'Coggeshall', '+33 (613) 890-7749', 'kcoggeshall6@dedecms.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (8, 8, 'Jillane', 'Koles', '+381 (103) 952-0001', 'jkoles7@techcrunch.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (9, 9, 'Lesya', 'Blizard', '+51 (649) 210-6156', 'lblizard8@liveinternet.ru');
insert into person (person_id, address_id, name, surname, tel, mail) values (10, 10, 'Hollie', 'Brandes', '+64 (795) 633-9196', 'hbrandes9@mozilla.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (11, 11, 'Alta', 'Tolle', '+30 (342) 644-2219', 'atollea@redcross.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (12, 12, 'Hartwell', 'Liddall', '+62 (316) 182-3091', 'hliddallb@msu.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (13, 13, 'Garwin', 'Pheazey', '+62 (484) 944-7484', 'gpheazeyc@businesswire.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (14, 14, 'Kimberlee', 'Fick', '+86 (367) 730-9900', 'kfickd@opensource.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (15, 15, 'Dulcy', 'Boyce', '+86 (570) 714-6659', 'dboycee@topsy.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (16, 16, 'Nanni', 'Elcock', '+86 (829) 161-9294', 'nelcockf@archive.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (17, 17, 'Coraline', 'Stilling', '+7 (836) 118-7002', 'cstillingg@dailymotion.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (18, 18, 'Terrell', 'Hatherell', '+62 (469) 870-3702', 'thatherellh@squarespace.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (19, 19, 'Garreth', 'Shiels', '+55 (144) 775-9148', 'gshielsi@mysql.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (20, 20, 'Vladimir', 'Seman', '+373 (875) 831-1814', 'vsemanj@telegraph.co.uk');
insert into person (person_id, address_id, name, surname, tel, mail) values (21, 21, 'Barbaraanne', 'Napleton', '+994 (214) 479-5847', 'bnapletonk@dropbox.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (22, 22, 'Emery', 'Hundy', '+44 (571) 760-8220', 'ehundyl@sitemeter.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (23, 23, 'Harley', 'Busfield', '+64 (946) 985-7557', 'hbusfieldm@canalblog.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (24, 24, 'Trixy', 'Prestner', '+358 (216) 433-4848', 'tprestnern@jalbum.net');
insert into person (person_id, address_id, name, surname, tel, mail) values (25, 25, 'Theo', 'Anwell', '+86 (710) 161-0227', 'tanwello@ucsd.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (26, 26, 'Aguie', 'Jahner', '+46 (148) 121-5328', 'ajahnerp@smugmug.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (27, 27, 'Arlana', 'Bunhill', '+994 (992) 162-3450', 'abunhillq@weather.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (28, 28, 'Kermit', 'Rasher', '+66 (761) 275-4768', 'krasherr@reference.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (29, 29, 'Micky', 'Padson', '+63 (881) 796-3284', 'mpadsons@thetimes.co.uk');
insert into person (person_id, address_id, name, surname, tel, mail) values (30, 30, 'Katti', 'Runsey', '+62 (670) 165-7546', 'krunseyt@bloglines.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (31, 31, 'Joceline', 'Wallington', '+86 (574) 223-3331', 'jwallingtonu@wufoo.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (32, 32, 'Jeremias', 'Beazer', '+46 (809) 405-5635', 'jbeazerv@tripadvisor.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (33, 33, 'Loydie', 'Hiom', '+1 (915) 771-5375', 'lhiomw@cloudflare.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (34, 34, 'Alanson', 'Swaddle', '+46 (655) 770-6019', 'aswaddlex@huffingtonpost.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (35, 35, 'Gui', 'Pedersen', '+62 (267) 997-8173', 'gpederseny@admin.ch');
insert into person (person_id, address_id, name, surname, tel, mail) values (36, 36, 'Sula', 'Corona', '+380 (468) 516-0688', 'scoronaz@seattletimes.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (37, 37, 'Zsa zsa', 'Seeley', '+63 (285) 838-3743', 'zseeley10@parallels.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (38, 38, 'Lillian', 'Goncaves', '+62 (670) 317-9830', 'lgoncaves11@bloomberg.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (39, 39, 'Kristofer', 'Muff', '+1 (413) 343-4905', 'kmuff12@reverbnation.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (40, 40, 'Gannon', 'Pamment', '+242 (233) 643-9733', 'gpamment13@newyorker.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (41, 41, 'Patsy', 'Hastings', '+49 (431) 218-0866', 'phastings14@oakley.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (42, 42, 'Aube', 'Lopez', '+94 (237) 948-4105', 'alopez15@sogou.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (43, 43, 'Oliviero', 'Shiers', '+86 (299) 787-2813', 'oshiers16@blinklist.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (44, 44, 'Lyda', 'Poley', '+994 (373) 439-3829', 'lpoley17@privacy.gov.au');
insert into person (person_id, address_id, name, surname, tel, mail) values (45, 45, 'Gaby', 'Coltman', '+48 (518) 686-4439', 'gcoltman18@upenn.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (46, 46, 'Lyn', 'Morcomb', '+33 (913) 647-8277', 'lmorcomb19@hud.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (47, 47, 'Blancha', 'Giovani', '+420 (204) 840-1393', 'bgiovani1a@yolasite.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (48, 48, 'Shaughn', 'Janusz', '+7 (707) 280-2345', 'sjanusz1b@indiegogo.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (49, 49, 'Inez', 'Horley', '+86 (424) 528-4550', 'ihorley1c@ustream.tv');
insert into person (person_id, address_id, name, surname, tel, mail) values (50, 50, 'Sabrina', 'Newband', '+86 (584) 909-2656', 'snewband1d@icio.us');
insert into person (person_id, address_id, name, surname, tel, mail) values (51, 51, 'Wait', 'Bendin', '+261 (185) 373-4744', 'wbendin1e@privacy.gov.au');
insert into person (person_id, address_id, name, surname, tel, mail) values (52, 52, 'Dianemarie', 'Brannan', '+86 (323) 471-8406', 'dbrannan1f@hatena.ne.jp');
insert into person (person_id, address_id, name, surname, tel, mail) values (53, 53, 'Stevie', 'Benfield', '+33 (770) 267-9551', 'sbenfield1g@biblegateway.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (54, 54, 'Nathanial', 'Balaisot', '+7 (778) 117-4023', 'nbalaisot1h@ezinearticles.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (55, 55, 'Geri', 'McKirton', '+963 (550) 930-3878', 'gmckirton1i@mysql.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (56, 56, 'Murvyn', 'Spoor', '+420 (348) 241-3883', 'mspoor1j@cmu.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (57, 57, 'Alain', 'Beatson', '+86 (701) 274-9956', 'abeatson1k@state.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (58, 58, 'Corrie', 'MacCulloch', '+380 (785) 773-7613', 'cmacculloch1l@tumblr.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (59, 59, 'Winnah', 'd'' Elboux', '+86 (323) 677-7044', 'wdelboux1m@mail.ru');
insert into person (person_id, address_id, name, surname, tel, mail) values (60, 60, 'Cammi', 'Cristofanini', '+66 (676) 664-2919', 'ccristofanini1n@boston.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (61, 61, 'Ezekiel', 'Lobbe', '+258 (962) 892-8063', 'elobbe1o@archive.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (62, 62, 'Neilla', 'Trill', '+33 (606) 501-2491', 'ntrill1p@usgs.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (63, 63, 'Luella', 'Leathwood', '+234 (137) 989-8574', 'lleathwood1q@umich.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (64, 64, 'Annaliese', 'Draisey', '+86 (765) 163-7934', 'adraisey1r@ezinearticles.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (65, 65, 'Donnie', 'Conman', '+351 (269) 757-8454', 'dconman1s@ibm.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (66, 66, 'Grover', 'Tourner', '+351 (607) 479-0182', 'gtourner1t@whitehouse.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (67, 67, 'Buck', 'Winward', '+62 (491) 856-7801', 'bwinward1u@elegantthemes.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (68, 68, 'Royal', 'Thornthwaite', '+385 (186) 549-2444', 'rthornthwaite1v@gravatar.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (69, 69, 'Herc', 'Bessent', '+7 (155) 442-1061', 'hbessent1w@imdb.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (70, 70, 'Jany', 'Gammet', '+351 (335) 243-7679', 'jgammet1x@nasa.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (71, 71, 'Aurilia', 'Tempest', '+970 (478) 598-1562', 'atempest1y@trellian.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (72, 72, 'Koo', 'Bodman', '+7 (199) 289-7269', 'kbodman1z@dropbox.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (73, 73, 'Marika', 'Meiningen', '+46 (423) 795-2300', 'mmeiningen20@accuweather.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (74, 74, 'Muffin', 'Woollhead', '+86 (704) 404-9456', 'mwoollhead21@hibu.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (75, 75, 'Zachery', 'Cowl', '+63 (679) 138-0742', 'zcowl22@hc360.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (76, 76, 'Lyda', 'O''Crevan', '+53 (635) 870-5002', 'locrevan23@auda.org.au');
insert into person (person_id, address_id, name, surname, tel, mail) values (77, 77, 'Lissi', 'Meeson', '+86 (494) 813-3120', 'lmeeson24@t-online.de');
insert into person (person_id, address_id, name, surname, tel, mail) values (78, 78, 'Loren', 'Smartman', '+63 (402) 551-9885', 'lsmartman25@blogtalkradio.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (79, 79, 'Lester', 'Fitzsimons', '+225 (122) 594-8971', 'lfitzsimons26@nps.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (80, 80, 'Tedmund', 'Gillum', '+7 (607) 757-7731', 'tgillum27@archive.org');
insert into person (person_id, address_id, name, surname, tel, mail) values (81, 81, 'Juline', 'Donner', '+48 (969) 540-1137', 'jdonner28@nymag.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (82, 82, 'Adel', 'Soppeth', '+86 (391) 297-9045', 'asoppeth29@sun.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (83, 83, 'Athena', 'Cowpland', '+7 (508) 295-0947', 'acowpland2a@yellowbook.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (84, 84, 'Laurence', 'Bodiam', '+86 (793) 571-3905', 'lbodiam2b@typepad.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (85, 85, 'Perla', 'Keys', '+57 (705) 615-5648', 'pkeys2c@netvibes.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (86, 86, 'Benoite', 'Simione', '+86 (262) 732-6141', 'bsimione2d@house.gov');
insert into person (person_id, address_id, name, surname, tel, mail) values (87, 87, 'Catarina', 'Coveny', '+86 (401) 894-8620', 'ccoveny2e@phpbb.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (88, 88, 'Marcela', 'Kealy', '+380 (979) 105-4745', 'mkealy2f@naver.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (89, 89, 'Taylor', 'Gunner', '+249 (215) 216-4271', 'tgunner2g@admin.ch');
insert into person (person_id, address_id, name, surname, tel, mail) values (90, 90, 'Dugald', 'Tinkham', '+86 (660) 826-2011', 'dtinkham2h@wufoo.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (91, 91, 'Renaud', 'Robarts', '+55 (299) 223-7381', 'rrobarts2i@linkedin.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (92, 92, 'Tera', 'Paradin', '+386 (474) 309-7463', 'tparadin2j@t-online.de');
insert into person (person_id, address_id, name, surname, tel, mail) values (93, 93, 'Phoebe', 'Adamovitch', '+62 (435) 350-0396', 'padamovitch2k@marriott.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (94, 94, 'Buiron', 'Debling', '+62 (139) 147-4181', 'bdebling2l@rakuten.co.jp');
insert into person (person_id, address_id, name, surname, tel, mail) values (95, 95, 'Genevieve', 'Prater', '+62 (878) 526-7852', 'gprater2m@washington.edu');
insert into person (person_id, address_id, name, surname, tel, mail) values (96, 96, 'Bryanty', 'Poytheras', '+30 (173) 957-8511', 'bpoytheras2n@mashable.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (97, 97, 'Jerrilyn', 'Genny', '+86 (334) 512-2781', 'jgenny2o@soup.io');
insert into person (person_id, address_id, name, surname, tel, mail) values (98, 98, 'Bonny', 'Brugger', '+967 (499) 678-3316', 'bbrugger2p@plala.or.jp');
insert into person (person_id, address_id, name, surname, tel, mail) values (99, 99, 'Maynard', 'Townley', '+86 (931) 467-7217', 'mtownley2q@yandex.ru');
insert into person (person_id, address_id, name, surname, tel, mail) values (100, 100, 'Viola', 'Bartke', '+977 (801) 516-9956', 'vbartke2r@ed.gov');

SELECT setval(pg_get_serial_sequence('person', 'person_id'), 100);
insert into person (person_id, address_id, name, surname, tel, mail) values (default, 101, 'Sagadat', 'Seitzhan', '+7 (205) 23-521', 'ereyadsredv@gmail.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (default, 102, 'Yuryi', 'Nauryzbay', '+334 4065-5360', 'aqtobesila@fit.com');
insert into person (person_id, address_id, name, surname, tel, mail) values (default, 103, 'Levin', 'Shpak', '+544 (312) 408-5278', 'cherkasisila@cvut.cz');

-- end of table persons

-- employee table

insert into employee (person_id, job_title, work_tel, work_mail) values (1, 'Financial Advisor', '+55 (330) 423-5956', 'orowcliffe0@mtv.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (2, 'Occupational Therapist', '+355 (664) 170-8373', 'eoselton1@epa.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (3, 'Nuclear Power Engineer', '+62 (537) 789-4872', 'npetrelli2@spotify.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (4, 'Civil Engineer', '+351 (431) 193-4656', 'iwhitta3@about.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (5, 'Legal Assistant', '+86 (584) 618-7765', 'ojoskowitz4@1688.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (6, 'Quality Engineer', '+216 (299) 284-4594', 'agilcrist5@google.co.uk');
insert into employee (person_id, job_title, work_tel, work_mail) values (7, 'GIS Technical Architect', '+7 (366) 813-7080', 'grussell6@japanpost.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (8, 'Business Systems Development Analyst', '+51 (145) 420-7868', 'baspling7@springer.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (9, 'Structural Analysis Engineer', '+81 (456) 770-0141', 'nandreopolos8@cisco.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (10, 'Environmental Specialist', '+86 (239) 926-0376', 'svanhalle9@exblog.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (11, 'Librarian', '+7 (616) 518-2524', 'ndrewsona@cyberchimps.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (12, 'Web Designer III', '+420 (297) 140-1335', 'rmouthb@nsw.gov.au');
insert into employee (person_id, job_title, work_tel, work_mail) values (13, 'Junior Executive', '+55 (244) 180-5174', 'dimortc@wikimedia.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (14, 'Environmental Specialist', '+63 (972) 955-5489', 'msaywelld@adobe.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (15, 'Editor', '+62 (204) 261-8036', 'bcoldwelle@diigo.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (16, 'Project Manager', '+86 (559) 962-2149', 'myieldingf@yellowbook.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (17, 'Analyst Programmer', '+357 (756) 946-2398', 'jmilbyg@behance.net');
insert into employee (person_id, job_title, work_tel, work_mail) values (18, 'Assistant Media Planner', '+62 (971) 906-2135', 'mcoronash@bigcartel.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (19, 'Geologist II', '+55 (949) 600-7142', 'bpeascodi@seattletimes.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (20, 'Business Systems Development Analyst', '+389 (655) 620-1576', 'msawdenj@ezinearticles.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (21, 'Tax Accountant', '+351 (594) 564-0169', 'pjobeyk@cdc.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (22, 'Physical Therapy Assistant', '+55 (713) 563-9156', 'tkennonl@topsy.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (23, 'Social Worker', '+55 (514) 334-4678', 'swaithm@vimeo.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (24, 'Recruiting Manager', '+46 (641) 571-7528', 'mpirnien@usnews.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (25, 'Nuclear Power Engineer', '+389 (683) 924-3082', 'kgreasleyo@nyu.edu');
insert into employee (person_id, job_title, work_tel, work_mail) values (26, 'Cost Accountant', '+62 (105) 324-0941', 'ccancelierp@prweb.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (27, 'Senior Editor', '+55 (715) 261-4225', 'mhassenq@msn.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (28, 'Food Chemist', '+381 (578) 270-3532', 'akilmurrayr@icio.us');
insert into employee (person_id, job_title, work_tel, work_mail) values (29, 'Research Assistant II', '+86 (478) 243-9884', 'vcleeves@bbb.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (30, 'Technical Writer', '+56 (144) 545-3757', 'hberntt@businessinsider.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (31, 'Actuary', '+593 (220) 183-4498', 'mperelliu@latimes.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (32, 'Software Test Engineer IV', '+58 (443) 512-4569', 'naverayv@cdbaby.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (33, 'Director of Sales', '+86 (722) 488-3482', 'sclawsleyw@liveinternet.ru');
insert into employee (person_id, job_title, work_tel, work_mail) values (34, 'Assistant Manager', '+356 (468) 363-5196', 'pwakenshawx@icq.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (35, 'Mechanical Systems Engineer', '+1 (283) 116-2818', 'jfaveyy@washingtonpost.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (36, 'Senior Financial Analyst', '+7 (139) 236-4048', 'ledgesonz@gmpg.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (37, 'Occupational Therapist', '+48 (776) 624-8388', 'gstubbington10@tripod.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (38, 'Analyst Programmer', '+55 (687) 752-6485', 'mveracruysse11@moonfruit.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (39, 'Compensation Analyst', '+63 (968) 487-6215', 'akelloch12@google.nl');
insert into employee (person_id, job_title, work_tel, work_mail) values (40, 'Occupational Therapist', '+86 (543) 775-6648', 'sbrunger13@census.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (41, 'Senior Quality Engineer', '+1 (817) 842-3195', 'hvasyunin14@oaic.gov.au');
insert into employee (person_id, job_title, work_tel, work_mail) values (42, 'Software Consultant', '+62 (816) 474-1258', 'mfortnam15@geocities.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (43, 'Research Nurse', '+504 (453) 916-4379', 'mkeysall16@typepad.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (44, 'VP Quality Control', '+351 (241) 549-7293', 'rtrengrouse17@bravesites.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (45, 'Operator', '+977 (644) 556-5976', 'creade18@nba.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (46, 'Recruiter', '+66 (362) 301-7939', 'tmathiassen19@live.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (47, 'Senior Sales Associate', '+7 (527) 406-9237', 'tviggers1a@nasa.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (48, 'Systems Administrator II', '+62 (195) 188-4759', 'aaisbett1b@symantec.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (49, 'Business Systems Development Analyst', '+7 (216) 703-9174', 'iisacsson1c@opensource.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (50, 'Analyst Programmer', '+46 (597) 189-0939', 'eaugar1d@bbc.co.uk');
insert into employee (person_id, job_title, work_tel, work_mail) values (51, 'Food Chemist', '+86 (757) 142-5162', 'tdongate1e@altervista.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (52, 'Product Engineer', '+33 (812) 138-7027', 'emeagh1f@wikipedia.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (53, 'Database Administrator IV', '+86 (259) 452-7699', 'abenbow1g@globo.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (54, 'Administrative Officer', '+30 (638) 774-6056', 'tpetcher1h@prweb.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (55, 'Graphic Designer', '+55 (727) 441-9640', 'asadlier1i@nba.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (56, 'Administrative Officer', '+249 (165) 557-5073', 'lwellum1j@prlog.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (57, 'Pharmacist', '+84 (595) 245-6956', 'jmatskiv1k@jiathis.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (58, 'Media Manager I', '+86 (743) 587-1843', 'akynder1l@plala.or.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (59, 'Professor', '+33 (750) 432-1834', 'hreinbech1m@npr.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (60, 'Research Assistant III', '+48 (154) 769-6438', 'kroycroft1n@surveymonkey.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (61, 'Help Desk Technician', '+57 (374) 279-4339', 'sosan1o@apache.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (62, 'Pharmacist', '+86 (445) 838-2892', 'lomrod1p@goo.gl');
insert into employee (person_id, job_title, work_tel, work_mail) values (63, 'Staff Accountant IV', '+358 (697) 466-4108', 'lrubinfeld1q@merriam-webster.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (64, 'Research Associate', '+63 (552) 279-2425', 'apenreth1r@people.com.cn');
insert into employee (person_id, job_title, work_tel, work_mail) values (65, 'Senior Financial Analyst', '+265 (689) 383-1627', 'achamberlain1s@amazon.de');
insert into employee (person_id, job_title, work_tel, work_mail) values (66, 'Nuclear Power Engineer', '+57 (482) 967-6952', 'wbassingden1t@ocn.ne.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (67, 'Librarian', '+93 (750) 461-3878', 'ctowll1u@slideshare.net');
insert into employee (person_id, job_title, work_tel, work_mail) values (68, 'Senior Quality Engineer', '+970 (879) 401-6091', 'hmccoy1v@walmart.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (69, 'Associate Professor', '+502 (709) 835-8864', 'ogoard1w@nymag.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (70, 'Nurse Practicioner', '+86 (345) 180-7032', 'csoares1x@craigslist.org');
insert into employee (person_id, job_title, work_tel, work_mail) values (71, 'Help Desk Technician', '+1 (607) 996-2092', 'skochs1y@nbcnews.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (72, 'Financial Analyst', '+36 (223) 977-0003', 'igillbe1z@digg.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (73, 'GIS Technical Architect', '+994 (715) 155-4250', 'gchomley20@bizjournals.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (74, 'Paralegal', '+62 (109) 764-9852', 'cforster21@networksolutions.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (75, 'Civil Engineer', '+86 (915) 256-2930', 'rboadby22@census.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (76, 'Senior Cost Accountant', '+30 (492) 936-9963', 'smarron23@princeton.edu');
insert into employee (person_id, job_title, work_tel, work_mail) values (77, 'Physical Therapy Assistant', '+30 (513) 681-8296', 'adelyth24@linkedin.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (78, 'Environmental Tech', '+86 (707) 989-8661', 'ayellowlees25@seattletimes.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (79, 'Software Engineer II', '+351 (456) 593-7036', 'cupsale26@csmonitor.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (80, 'Clinical Specialist', '+351 (977) 770-2774', 'rmcdougal27@foxnews.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (81, 'Staff Accountant I', '+962 (280) 393-4723', 'cdannehl28@youku.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (82, 'Product Engineer', '+51 (465) 276-0045', 'mgelsthorpe29@guardian.co.uk');
insert into employee (person_id, job_title, work_tel, work_mail) values (83, 'Quality Control Specialist', '+48 (336) 260-7931', 'lcamplejohn2a@cbsnews.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (84, 'Compensation Analyst', '+686 (506) 344-7624', 'hbaniard2b@columbia.edu');
insert into employee (person_id, job_title, work_tel, work_mail) values (85, 'Programmer Analyst II', '+33 (541) 507-5276', 'jjerzyk2c@hostgator.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (86, 'Safety Technician III', '+55 (543) 975-1469', 'keberdt2d@indiatimes.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (87, 'VP Product Management', '+386 (909) 494-3550', 'cfelix2e@goo.ne.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (88, 'Software Consultant', '+86 (571) 686-3186', 'mgreber2f@hugedomains.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (89, 'Director of Sales', '+218 (848) 213-2967', 'mlivzey2g@pagesperso-orange.fr');
insert into employee (person_id, job_title, work_tel, work_mail) values (90, 'Information Systems Manager', '+62 (355) 171-0207', 'aakester2h@topsy.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (91, 'Chemical Engineer', '+86 (565) 686-3783', 'cjaye2i@purevolume.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (92, 'Human Resources Manager', '+51 (426) 202-9361', 'wwickersham2j@booking.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (93, 'Senior Sales Associate', '+353 (416) 100-9508', 'cstubbington2k@hatena.ne.jp');
insert into employee (person_id, job_title, work_tel, work_mail) values (94, 'Statistician II', '+92 (740) 904-2295', 'jluparti2l@howstuffworks.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (95, 'Sales Associate', '+420 (943) 742-2765', 'rgainforth2m@1688.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (96, 'VP Accounting', '+55 (386) 918-1209', 'mbidwell2n@uol.com.br');
insert into employee (person_id, job_title, work_tel, work_mail) values (97, 'Associate Professor', '+62 (281) 594-0436', 'cpersey2o@google.ru');
insert into employee (person_id, job_title, work_tel, work_mail) values (98, 'Data Coordinator', '+976 (641) 655-2011', 'jdrohane2p@epa.gov');
insert into employee (person_id, job_title, work_tel, work_mail) values (99, 'VP Product Management', '+55 (834) 172-8880', 'raxel2q@nymag.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (100, 'Professor', '+53 (152) 921-7924', 'sallbut2r@quantcast.com');
insert into employee (person_id, job_title, work_tel, work_mail) values (101, 'Data Science', '+7 (234) 123-7123', 'klmkp@quantcast.com');

-- end of table employees

-- candidate table

insert into candidate (person_id, job_title, years_of_experience, category, description) values (1, 'Engineer III', 3, 'Maecenas rhoncus aliquam lacus.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (2, 'Accountant IV', 10, 'Donec posuere metus vitae ipsum.', 'Morbi porttitor lorem id ligula.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (3, 'Structural Analysis Engineer', 1, 'Nulla facilisi.', 'Aliquam erat volutpat. In congue.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (4, 'Sales Representative', 10, 'Sed vel enim sit amet nunc viverra dapibus.', 'Etiam pretium iaculis justo. In hac habitasse platea dictumst. Etiam faucibus cursus urna. Ut tellus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (5, 'Legal Assistant', 1, 'Suspendisse potenti.', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (6, 'Geological Engineer', 2, 'Donec posuere metus vitae ipsum.', 'Curabitur at ipsum ac tellus semper interdum. Mauris ullamcorper purus sit amet nulla. Quisque arcu libero, rutrum ac, lobortis vel, dapibus at, diam. Nam tristique tortor eu pede.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (7, 'Office Assistant III', 11, 'Sed ante.', 'Ut tellus. Nulla ut erat id mauris vulputate elementum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (8, 'VP Marketing', 16, 'Ut at dolor quis odio consequat varius.', 'Nam tristique tortor eu pede.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (9, 'Structural Analysis Engineer', 4, 'Aliquam sit amet diam in magna bibendum imperdiet.', 'Vivamus tortor. Duis mattis egestas metus. Aenean fermentum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (10, 'Senior Editor', 9, 'Etiam vel augue.', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue. Vestibulum rutrum rutrum neque.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (11, 'Chemical Engineer', 26, 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.', 'In hac habitasse platea dictumst. Maecenas ut massa quis augue luctus tincidunt. Nulla mollis molestie lorem. Quisque ut erat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (12, 'Accountant I', 30, 'Vestibulum ac est lacinia nisi venenatis tristique.', 'In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat. Nulla nisl.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (13, 'Automation Specialist III', 14, 'Quisque porta volutpat erat.', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (14, 'Staff Accountant III', 12, 'Nullam molestie nibh in lectus.', 'Curabitur at ipsum ac tellus semper interdum. Mauris ullamcorper purus sit amet nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (15, 'Professor', 25, 'Praesent id massa id nisl venenatis lacinia.', 'Proin risus. Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Duis faucibus accumsan odio.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (16, 'VP Marketing', 13, 'Suspendisse potenti.', 'Donec quis orci eget orci vehicula condimentum. Curabitur in libero ut massa volutpat convallis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (17, 'Technical Writer', 4, 'In hac habitasse platea dictumst.', 'In congue. Etiam justo.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (18, 'Electrical Engineer', 22, 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit.', 'Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (19, 'Accountant IV', 13, 'Morbi vel lectus in quam fringilla rhoncus.', 'Morbi a ipsum. Integer a nibh.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (20, 'Director of Sales', 14, 'Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.', 'Curabitur in libero ut massa volutpat convallis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (21, 'Cost Accountant', 24, 'Aenean lectus.', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (22, 'Sales Representative', 12, 'In sagittis dui vel nisl.', 'Nulla ut erat id mauris vulputate elementum. Nullam varius.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (23, 'Budget/Accounting Analyst I', 27, 'Curabitur in libero ut massa volutpat convallis.', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (24, 'Nurse', 21, 'Nullam sit amet turpis elementum ligula vehicula consequat.', 'Morbi a ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (25, 'Senior Quality Engineer', 12, 'Praesent blandit lacinia erat.', 'Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (26, 'Nuclear Power Engineer', 4, 'Morbi vel lectus in quam fringilla rhoncus.', 'In sagittis dui vel nisl.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (27, 'Community Outreach Specialist', 20, 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', 'Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (28, 'Graphic Designer', 20, 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', 'Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla. Nunc purus. Phasellus in felis. Donec semper sapien a libero.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (29, 'Safety Technician IV', 15, 'Morbi a ipsum.', 'Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est. Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (30, 'Biostatistician III', 25, 'Curabitur gravida nisi at nibh.', 'Nunc rhoncus dui vel sem.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (31, 'Developer I', 14, 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (32, 'Chemical Engineer', 11, 'In hac habitasse platea dictumst.', 'Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (33, 'Professor', 25, 'Curabitur in libero ut massa volutpat convallis.', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum. Aliquam non mauris. Morbi non lectus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (34, 'VP Sales', 27, 'Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (35, 'Pharmacist', 27, 'Quisque id justo sit amet sapien dignissim vestibulum.', 'Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (36, 'Environmental Specialist', 19, 'Quisque arcu libero, rutrum ac, lobortis vel, dapibus at, diam.', 'Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo. Pellentesque viverra pede ac diam. Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (37, 'Pharmacist', 28, 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', 'Vivamus vestibulum sagittis sapien.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (38, 'Programmer IV', 23, 'Suspendisse ornare consequat lectus.', 'In quis justo. Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (39, 'Occupational Therapist', 10, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (40, 'Design Engineer', 23, 'Donec ut mauris eget massa tempor convallis.', 'Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi. Nam ultrices, libero non mattis pulvinar, nulla pede ullamcorper augue, a suscipit nulla elit ac nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (41, 'Software Consultant', 24, 'Nulla ac enim.', 'Vestibulum sed magna at nunc commodo placerat. Praesent blandit.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (42, 'Biostatistician II', 6, 'Nulla tellus.', 'Nullam varius.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (43, 'Account Coordinator', 28, 'Nulla ut erat id mauris vulputate elementum.', 'Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat. Praesent blandit.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (44, 'Programmer Analyst II', 2, 'Duis bibendum.', 'Phasellus in felis. Donec semper sapien a libero. Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (45, 'Assistant Professor', 22, 'Maecenas rhoncus aliquam lacus.', 'Duis consequat dui nec nisi volutpat eleifend. Donec ut dolor.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (46, 'Civil Engineer', 27, 'Morbi quis tortor id nulla ultrices aliquet.', 'Phasellus in felis. Donec semper sapien a libero.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (47, 'Senior Financial Analyst', 21, 'Nunc purus.', 'Vivamus vestibulum sagittis sapien. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (48, 'Senior Financial Analyst', 18, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.', 'Fusce posuere felis sed lacus. Morbi sem mauris, laoreet ut, rhoncus aliquet, pulvinar sed, nisl. Nunc rhoncus dui vel sem.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (49, 'Administrative Assistant II', 5, 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin interdum mauris non ligula pellentesque ultrices. Phasellus id sapien in sapien iaculis congue. Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (50, 'Accounting Assistant IV', 19, 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', 'Donec ut mauris eget massa tempor convallis. Nulla neque libero, convallis eget, eleifend luctus, ultricies eu, nibh.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (51, 'Senior Financial Analyst', 21, 'Quisque id justo sit amet sapien dignissim vestibulum.', 'Morbi odio odio, elementum eu, interdum eu, tincidunt in, leo. Maecenas pulvinar lobortis est.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (52, 'Financial Advisor', 24, 'Nulla ac enim.', 'Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (53, 'Computer Systems Analyst II', 1, 'Sed vel enim sit amet nunc viverra dapibus.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (54, 'Desktop Support Technician', 11, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Duis faucibus accumsan odio.', 'Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (55, 'Senior Sales Associate', 16, 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit.', 'Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo. In blandit ultrices enim. Lorem ipsum dolor sit amet, consectetuer adipiscing elit.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (56, 'Research Associate', 16, 'Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.', 'Donec ut dolor.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (57, 'Help Desk Operator', 8, 'Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem.', 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo. In blandit ultrices enim.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (58, 'Paralegal', 30, 'Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla.', 'In sagittis dui vel nisl.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (59, 'Chemical Engineer', 14, 'Duis at velit eu est congue elementum.', 'Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (60, 'VP Quality Control', 24, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est. Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros. Vestibulum ac est lacinia nisi venenatis tristique.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (61, 'Assistant Manager', 20, 'Vivamus tortor.', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (62, 'Information Systems Manager', 10, 'Nulla justo.', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (63, 'Nurse', 1, 'Pellentesque at nulla.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (64, 'Engineer I', 6, 'Fusce consequat.', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin risus. Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (65, 'Chemical Engineer', 6, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.', 'Phasellus sit amet erat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (66, 'Engineer III', 9, 'Nunc rhoncus dui vel sem.', 'Aliquam non mauris. Morbi non lectus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (67, 'Web Designer IV', 17, 'Duis consequat dui nec nisi volutpat eleifend.', 'Aenean fermentum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (68, 'Analyst Programmer', 4, 'Nunc purus.', 'Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (69, 'Dental Hygienist', 22, 'Nulla neque libero, convallis eget, eleifend luctus, ultricies eu, nibh.', 'Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (70, 'Registered Nurse', 22, 'Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (71, 'Sales Representative', 17, 'Duis at velit eu est congue elementum.', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (72, 'Senior Cost Accountant', 15, 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', 'Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat. Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (73, 'Clinical Specialist', 17, 'Nunc rhoncus dui vel sem.', 'Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci. Mauris lacinia sapien quis libero. Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (74, 'Biostatistician II', 27, 'In congue.', 'Phasellus in felis. Donec semper sapien a libero. Nam dui.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (75, 'Information Systems Manager', 16, 'Vivamus vestibulum sagittis sapien.', 'Aliquam non mauris. Morbi non lectus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (76, 'Structural Engineer', 6, 'Nunc purus.', 'Nulla justo. Aliquam quis turpis eget elit sodales scelerisque. Mauris sit amet eros.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (77, 'Research Assistant II', 19, 'Nulla mollis molestie lorem.', 'Suspendisse potenti. Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (78, 'Dental Hygienist', 14, 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.', 'Curabitur gravida nisi at nibh. In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (79, 'Operator', 30, 'In hac habitasse platea dictumst.', 'Duis bibendum. Morbi non quam nec dui luctus rutrum.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (80, 'Compensation Analyst', 16, 'Donec quis orci eget orci vehicula condimentum.', 'Morbi sem mauris, laoreet ut, rhoncus aliquet, pulvinar sed, nisl. Nunc rhoncus dui vel sem. Sed sagittis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (81, 'Biostatistician II', 21, 'Donec dapibus.', 'In hac habitasse platea dictumst.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (82, 'Registered Nurse', 3, 'Quisque id justo sit amet sapien dignissim vestibulum.', 'Suspendisse potenti. Cras in purus eu magna vulputate luctus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (83, 'Senior Editor', 18, 'Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.', 'In quis justo. Maecenas rhoncus aliquam lacus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (84, 'Research Associate', 26, 'Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.', 'Quisque ut erat.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (85, 'Staff Accountant III', 14, 'Suspendisse potenti.', 'Duis at velit eu est congue elementum. In hac habitasse platea dictumst. Morbi vestibulum, velit id pretium iaculis, diam erat fermentum justo, nec condimentum neque sapien placerat ante. Nulla justo.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (86, 'Chief Design Engineer', 22, 'In hac habitasse platea dictumst.', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (87, 'Financial Advisor', 12, 'Morbi non quam nec dui luctus rutrum.', 'Nunc purus. Phasellus in felis. Donec semper sapien a libero.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (88, 'Community Outreach Specialist', 19, 'Duis ac nibh.', 'Pellentesque ultrices mattis odio. Donec vitae nisi. Nam ultrices, libero non mattis pulvinar, nulla pede ullamcorper augue, a suscipit nulla elit ac nulla.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (89, 'Chemical Engineer', 28, 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', 'Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (90, 'Human Resources Assistant III', 7, 'Praesent id massa id nisl venenatis lacinia.', 'Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti. Nullam porttitor lacus at turpis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (91, 'Senior Cost Accountant', 26, 'Nullam sit amet turpis elementum ligula vehicula consequat.', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (92, 'Director of Sales', 18, 'Nunc rhoncus dui vel sem.', 'Vestibulum sed magna at nunc commodo placerat. Praesent blandit.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (93, 'Software Consultant', 12, 'Vestibulum rutrum rutrum neque.', 'Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet. Nullam orci pede, venenatis non, sodales sed, tincidunt eu, felis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (94, 'Editor', 25, 'Integer tincidunt ante vel ipsum.', 'Mauris lacinia sapien quis libero. Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum. Integer a nibh.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (95, 'Health Coach II', 1, 'Curabitur convallis.', 'Phasellus in felis. Donec semper sapien a libero. Nam dui.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (96, 'Office Assistant IV', 12, 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', 'Praesent id massa id nisl venenatis lacinia. Aenean sit amet justo. Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (97, 'Graphic Designer', 26, 'Aliquam quis turpis eget elit sodales scelerisque.', 'Pellentesque at nulla. Suspendisse potenti. Cras in purus eu magna vulputate luctus.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (98, 'Budget/Accounting Analyst IV', 21, 'Etiam pretium iaculis justo.', 'Phasellus in felis.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (99, 'Clinical Specialist', 20, 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', 'Praesent id massa id nisl venenatis lacinia. Aenean sit amet justo. Morbi ut odio.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (100, 'Pharmacist', 26, 'Aliquam non mauris.', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.');
insert into candidate (person_id, job_title, years_of_experience, category, description) values (101, 'Data Scientist', 13, 'DataBases', 'HIM!');

-- end of candidate table

-- table certification

insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (1, 'ESLT', 'Skinder', '01/07/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (2, 'BEP', 'Ntags', '27/01/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (3, 'DD^B', 'Vidoo', '05/02/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (4, 'TJX', 'Yodel', '10/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (5, 'NEE^K', 'Fivebridge', '25/09/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (6, 'HYH', 'Dynava', '14/01/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (7, 'KGJI', 'Zoombeat', '04/09/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (8, 'MUR', 'Roomm', '16/08/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (9, 'BML^I', 'Edgepulse', '21/06/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (10, 'FWP', 'Yoveo', '19/08/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (11, 'PFH', 'Katz', '19/01/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (12, 'MOTAW', 'Skajo', '05/08/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (13, 'LAD', 'Wikibox', '18/08/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (14, 'OAKS^A', 'Meemm', '19/07/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (15, 'GPACW', 'Oodoo', '03/12/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (16, 'WHF', 'Avamba', '21/08/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (17, 'NAKD', 'Skimia', '13/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (18, 'LMRKP', 'Avamba', '26/03/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (19, 'NCZ', 'Kwideo', '08/07/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (20, 'WSCI', 'Agivu', '28/01/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (21, 'PGTI', 'Photobean', '17/07/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (22, 'FENX', 'Bubblebox', '24/02/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (23, 'VEC', 'Youtags', '10/12/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (24, 'COGT', 'Oozz', '01/03/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (25, 'ALKS', 'Skalith', '18/11/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (26, 'BPOPM', 'Wikivu', '02/06/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (27, 'MPA', 'Gabvine', '18/12/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (28, 'GDOT', 'Kimia', '14/11/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (29, 'EWBC', 'Blogtags', '21/05/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (30, 'VOXX', 'Riffpedia', '21/05/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (31, 'KOPN', 'Ozu', '21/09/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (32, 'TWMC', 'Photolist', '14/10/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (33, 'GEMP', 'Jaxbean', '06/04/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (34, 'MRUS', 'Kamba', '09/02/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (35, 'CETV', 'Kazio', '21/04/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (36, 'CHMA', 'Pixope', '26/07/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (37, 'SAN^B', 'Linktype', '04/09/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (38, 'DSM', 'Gabtype', '16/12/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (39, 'VNCE', 'Vipe', '16/08/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (40, 'SJW', 'Zoovu', '12/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (41, 'OFG^D', 'Babbleblab', '31/01/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (42, 'PNF', 'Nlounge', '13/05/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (43, 'CPAH', 'Plambee', '09/04/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (44, 'GDDY', 'Skaboo', '22/11/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (45, 'NI', 'Gigashots', '05/05/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (46, 'PRAH', 'Fanoodle', '24/02/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (47, 'PSDV', 'Plajo', '23/03/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (48, 'LMRK', 'Edgepulse', '07/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (49, 'DAR', 'Realfire', '17/11/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (50, 'CRY', 'Bubblebox', '27/01/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (51, 'CMFN', 'Wordpedia', '26/03/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (52, 'DEI', 'Skilith', '23/01/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (53, 'ADES', 'Kaymbo', '06/12/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (54, 'ANIP', 'Trunyx', '29/08/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (55, 'NMM', 'Topiczoom', '24/09/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (56, 'DVA', 'Jaxworks', '03/04/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (57, 'SGBK', 'Quamba', '10/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (58, 'SNHNL', 'Ainyx', '19/08/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (59, 'TPX', 'Browsezoom', '25/04/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (60, 'LINK', 'Avamba', '25/09/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (61, 'NVMI', 'Fliptune', '26/01/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (62, 'OHGI', 'Fanoodle', '19/03/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (63, 'ANTX', 'Skinix', '14/08/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (64, 'RE', 'Meevee', '18/02/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (65, 'BBN', 'Photospace', '16/02/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (66, 'KWEB', 'Ntags', '15/03/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (67, 'AAWW', 'Yakitri', '12/02/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (68, 'NGHCP', 'Chatterbridge', '14/10/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (69, 'NMT', 'Feedfish', '17/12/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (70, 'VEDL', 'Thoughtstorm', '06/04/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (71, 'EBIO', 'Topicstorm', '13/10/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (72, 'CIO', 'Flashset', '30/03/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (73, 'FRSH', 'Browsezoom', '16/06/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (74, 'GFY', 'Thoughtsphere', '18/04/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (75, 'TTGT', 'JumpXS', '09/08/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (76, 'MTB^', 'Realblab', '03/01/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (77, 'PEI^B', 'Thoughtstorm', '22/03/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (78, 'DBVT', 'Trilia', '13/03/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (79, 'OXY', 'Jatri', '12/03/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (80, 'JOUT', 'Blogspan', '06/05/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (81, 'FANG', 'Edgewire', '11/02/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (82, 'OKE', 'DabZ', '04/01/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (83, 'WFT', 'Voomm', '07/02/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (84, 'JSYN', 'Tagchat', '28/05/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (85, 'USB^O', 'Kwilith', '20/04/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (86, 'ESE', 'Ailane', '06/08/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (87, 'NPTN', 'Oyondu', '05/01/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (88, 'IRM', 'Quire', '27/04/2021');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (89, 'BKSC', 'Youfeed', '04/01/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (90, 'SYT', 'Youbridge', '27/09/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (91, 'TNP^B', 'Thoughtstorm', '25/06/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (92, 'OXM', 'Realmix', '19/11/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (93, 'AGZD', 'Eimbee', '08/04/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (94, 'GNL', 'Flashpoint', '23/07/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (95, 'CRZO', 'Skaboo', '30/12/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (96, 'AEK', 'Dabtype', '06/02/2023');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (97, 'RVSB', 'Twitterlist', '05/01/2020');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (98, 'ISRG', 'Rhyzio', '02/01/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (99, 'THRM', 'Buzzdog', '03/05/2022');
insert into certification (certification_id, certification_name, certifying_organization, certification_date) values (100, 'PPL', 'Blogpad', '01/10/2022');

-- end of table certificates

-- company table

insert into company (company_id, address_id, company_name, company_description) values (1, 1, 'Jabbersphere', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.');
insert into company (company_id, address_id, company_name, company_description) values (2, 2, 'Skiptube', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Duis faucibus accumsan odio. Curabitur convallis.');
insert into company (company_id, address_id, company_name, company_description) values (3, 3, 'Realmix', 'Quisque ut erat. Curabitur gravida nisi at nibh.');
insert into company (company_id, address_id, company_name, company_description) values (4, 4, 'Jatri', 'Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus.');
insert into company (company_id, address_id, company_name, company_description) values (5, 5, 'Cogidoo', 'Curabitur convallis. Duis consequat dui nec nisi volutpat eleifend. Donec ut dolor.');
insert into company (company_id, address_id, company_name, company_description) values (6, 6, 'Meemm', 'Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa. Donec dapibus. Duis at velit eu est congue elementum. In hac habitasse platea dictumst.');
insert into company (company_id, address_id, company_name, company_description) values (7, 7, 'Dynazzy', 'Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti.');
insert into company (company_id, address_id, company_name, company_description) values (8, 8, 'Gabspot', 'Nam ultrices, libero non mattis pulvinar, nulla pede ullamcorper augue, a suscipit nulla elit ac nulla. Sed vel enim sit amet nunc viverra dapibus. Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum.');
insert into company (company_id, address_id, company_name, company_description) values (9, 9, 'Skajo', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat.');
insert into company (company_id, address_id, company_name, company_description) values (10, 10, 'Eadel', 'Morbi non quam nec dui luctus rutrum. Nulla tellus. In sagittis dui vel nisl. Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.');
insert into company (company_id, address_id, company_name, company_description) values (11, 11, 'Dabtype', 'In hac habitasse platea dictumst. Etiam faucibus cursus urna. Ut tellus.');
insert into company (company_id, address_id, company_name, company_description) values (12, 12, 'Roombo', 'Nulla tellus. In sagittis dui vel nisl. Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.');
insert into company (company_id, address_id, company_name, company_description) values (13, 13, 'Jabberbean', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi.');
insert into company (company_id, address_id, company_name, company_description) values (14, 14, 'Plambee', 'Nam tristique tortor eu pede.');
insert into company (company_id, address_id, company_name, company_description) values (15, 15, 'Mybuzz', 'Nulla tellus. In sagittis dui vel nisl. Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.');
insert into company (company_id, address_id, company_name, company_description) values (16, 16, 'Yozio', 'Vivamus tortor. Duis mattis egestas metus.');
insert into company (company_id, address_id, company_name, company_description) values (17, 17, 'Centimia', 'Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue. Etiam justo.');
insert into company (company_id, address_id, company_name, company_description) values (18, 18, 'Wordware', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat.');
insert into company (company_id, address_id, company_name, company_description) values (19, 19, 'Yakidoo', 'Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis. Ut at dolor quis odio consequat varius. Integer ac leo. Pellentesque ultrices mattis odio.');
insert into company (company_id, address_id, company_name, company_description) values (20, 20, 'Brainverse', 'Morbi vestibulum, velit id pretium iaculis, diam erat fermentum justo, nec condimentum neque sapien placerat ante.');
insert into company (company_id, address_id, company_name, company_description) values (21, 21, 'Edgewire', 'Donec ut mauris eget massa tempor convallis.');
insert into company (company_id, address_id, company_name, company_description) values (22, 22, 'Tavu', 'Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit.');
insert into company (company_id, address_id, company_name, company_description) values (23, 23, 'Mynte', 'Quisque ut erat. Curabitur gravida nisi at nibh. In hac habitasse platea dictumst.');
insert into company (company_id, address_id, company_name, company_description) values (24, 24, 'Feedspan', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus. Pellentesque at nulla. Suspendisse potenti.');
insert into company (company_id, address_id, company_name, company_description) values (25, 25, 'Dabshots', 'Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.');
insert into company (company_id, address_id, company_name, company_description) values (26, 26, 'Innotype', 'Aliquam quis turpis eget elit sodales scelerisque. Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante. Vivamus tortor.');
insert into company (company_id, address_id, company_name, company_description) values (27, 27, 'Topdrive', 'Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo. In blandit ultrices enim. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin interdum mauris non ligula pellentesque ultrices.');
insert into company (company_id, address_id, company_name, company_description) values (28, 28, 'Realcube', 'Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci. Mauris lacinia sapien quis libero. Nullam sit amet turpis elementum ligula vehicula consequat.');
insert into company (company_id, address_id, company_name, company_description) values (29, 29, 'Fivespan', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti. Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum. Aliquam non mauris.');
insert into company (company_id, address_id, company_name, company_description) values (30, 30, 'Oba', 'Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum. Mauris ullamcorper purus sit amet nulla.');
insert into company (company_id, address_id, company_name, company_description) values (31, 31, 'Meeveo', 'Nulla tellus.');
insert into company (company_id, address_id, company_name, company_description) values (32, 32, 'Cogibox', 'Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into company (company_id, address_id, company_name, company_description) values (33, 33, 'Flashdog', 'Duis mattis egestas metus.');
insert into company (company_id, address_id, company_name, company_description) values (34, 34, 'Camimbo', 'Phasellus in felis. Donec semper sapien a libero. Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis.');
insert into company (company_id, address_id, company_name, company_description) values (35, 35, 'Realblab', 'Proin interdum mauris non ligula pellentesque ultrices. Phasellus id sapien in sapien iaculis congue. Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.');
insert into company (company_id, address_id, company_name, company_description) values (36, 36, 'Edgeclub', 'Morbi a ipsum. Integer a nibh. In quis justo. Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet.');
insert into company (company_id, address_id, company_name, company_description) values (37, 37, 'Pixoboo', 'In congue. Etiam justo. Etiam pretium iaculis justo. In hac habitasse platea dictumst. Etiam faucibus cursus urna.');
insert into company (company_id, address_id, company_name, company_description) values (38, 38, 'Nlounge', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum. Integer a nibh. In quis justo.');
insert into company (company_id, address_id, company_name, company_description) values (39, 39, 'Cogidoo', 'Etiam justo. Etiam pretium iaculis justo. In hac habitasse platea dictumst. Etiam faucibus cursus urna. Ut tellus.');
insert into company (company_id, address_id, company_name, company_description) values (40, 40, 'Agimba', 'Etiam vel augue. Vestibulum rutrum rutrum neque. Aenean auctor gravida sem.');
insert into company (company_id, address_id, company_name, company_description) values (41, 41, 'Podcat', 'Morbi non quam nec dui luctus rutrum. Nulla tellus. In sagittis dui vel nisl.');
insert into company (company_id, address_id, company_name, company_description) values (42, 42, 'Photojam', 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo. In blandit ultrices enim. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin interdum mauris non ligula pellentesque ultrices.');
insert into company (company_id, address_id, company_name, company_description) values (43, 43, 'Trupe', 'Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo. Pellentesque viverra pede ac diam. Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.');
insert into company (company_id, address_id, company_name, company_description) values (44, 44, 'Thoughtstorm', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat.');
insert into company (company_id, address_id, company_name, company_description) values (45, 45, 'Voomm', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum.');
insert into company (company_id, address_id, company_name, company_description) values (46, 46, 'Gigashots', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.');
insert into company (company_id, address_id, company_name, company_description) values (47, 47, 'Chatterpoint', 'Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus. Pellentesque at nulla. Suspendisse potenti.');
insert into company (company_id, address_id, company_name, company_description) values (48, 48, 'Voolia', 'Etiam faucibus cursus urna. Ut tellus.');
insert into company (company_id, address_id, company_name, company_description) values (49, 49, 'Skibox', 'Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci. Mauris lacinia sapien quis libero. Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum. Integer a nibh.');
insert into company (company_id, address_id, company_name, company_description) values (50, 50, 'Wikido', 'Donec semper sapien a libero. Nam dui.');
insert into company (company_id, address_id, company_name, company_description) values (51, 51, 'Kaymbo', 'Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla.');
insert into company (company_id, address_id, company_name, company_description) values (52, 52, 'Gabspot', 'Vivamus tortor. Duis mattis egestas metus. Aenean fermentum. Donec ut mauris eget massa tempor convallis.');
insert into company (company_id, address_id, company_name, company_description) values (53, 53, 'Feednation', 'In blandit ultrices enim.');
insert into company (company_id, address_id, company_name, company_description) values (54, 54, 'Twimm', 'Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus. Pellentesque at nulla. Suspendisse potenti.');
insert into company (company_id, address_id, company_name, company_description) values (55, 55, 'Thoughtbeat', 'Donec posuere metus vitae ipsum. Aliquam non mauris. Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet.');
insert into company (company_id, address_id, company_name, company_description) values (56, 56, 'Tagfeed', 'Nulla justo.');
insert into company (company_id, address_id, company_name, company_description) values (57, 57, 'Feedfire', 'Nulla tempus. Vivamus in felis eu sapien cursus vestibulum. Proin eu mi. Nulla ac enim.');
insert into company (company_id, address_id, company_name, company_description) values (58, 58, 'Linktype', 'Etiam vel augue. Vestibulum rutrum rutrum neque. Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia.');
insert into company (company_id, address_id, company_name, company_description) values (59, 59, 'Eidel', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat.');
insert into company (company_id, address_id, company_name, company_description) values (60, 60, 'Zoomcast', 'Integer ac neque.');
insert into company (company_id, address_id, company_name, company_description) values (61, 61, 'Rhyzio', 'Ut tellus.');
insert into company (company_id, address_id, company_name, company_description) values (62, 62, 'Meedoo', 'Sed accumsan felis. Ut at dolor quis odio consequat varius. Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi.');
insert into company (company_id, address_id, company_name, company_description) values (63, 63, 'Eire', 'Etiam pretium iaculis justo.');
insert into company (company_id, address_id, company_name, company_description) values (64, 64, 'Flashspan', 'Proin at turpis a pede posuere nonummy.');
insert into company (company_id, address_id, company_name, company_description) values (65, 65, 'DabZ', 'Donec ut mauris eget massa tempor convallis. Nulla neque libero, convallis eget, eleifend luctus, ultricies eu, nibh. Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.');
insert into company (company_id, address_id, company_name, company_description) values (66, 66, 'Zoomdog', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti. Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum.');
insert into company (company_id, address_id, company_name, company_description) values (67, 67, 'Latz', 'Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi. Nam ultrices, libero non mattis pulvinar, nulla pede ullamcorper augue, a suscipit nulla elit ac nulla.');
insert into company (company_id, address_id, company_name, company_description) values (68, 68, 'Flashset', 'Nunc purus. Phasellus in felis.');
insert into company (company_id, address_id, company_name, company_description) values (69, 69, 'Oyope', 'Quisque porta volutpat erat. Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla. Nunc purus.');
insert into company (company_id, address_id, company_name, company_description) values (70, 70, 'Edgeblab', 'Nulla mollis molestie lorem.');
insert into company (company_id, address_id, company_name, company_description) values (71, 71, 'Skinder', 'Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla.');
insert into company (company_id, address_id, company_name, company_description) values (72, 72, 'Dabjam', 'In eleifend quam a odio. In hac habitasse platea dictumst. Maecenas ut massa quis augue luctus tincidunt. Nulla mollis molestie lorem.');
insert into company (company_id, address_id, company_name, company_description) values (73, 73, 'Dabvine', 'Vivamus in felis eu sapien cursus vestibulum. Proin eu mi.');
insert into company (company_id, address_id, company_name, company_description) values (74, 74, 'Browsezoom', 'Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque.');
insert into company (company_id, address_id, company_name, company_description) values (75, 75, 'Meemm', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue.');
insert into company (company_id, address_id, company_name, company_description) values (76, 76, 'Oozz', 'Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis. Ut at dolor quis odio consequat varius.');
insert into company (company_id, address_id, company_name, company_description) values (77, 77, 'Jazzy', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.');
insert into company (company_id, address_id, company_name, company_description) values (78, 78, 'Katz', 'Morbi a ipsum. Integer a nibh. In quis justo. Maecenas rhoncus aliquam lacus.');
insert into company (company_id, address_id, company_name, company_description) values (79, 79, 'Wordpedia', 'Curabitur gravida nisi at nibh.');
insert into company (company_id, address_id, company_name, company_description) values (80, 80, 'Skilith', 'Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo. In blandit ultrices enim. Lorem ipsum dolor sit amet, consectetuer adipiscing elit.');
insert into company (company_id, address_id, company_name, company_description) values (81, 81, 'Vimbo', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat.');
insert into company (company_id, address_id, company_name, company_description) values (82, 82, 'Mita', 'Ut at dolor quis odio consequat varius. Integer ac leo.');
insert into company (company_id, address_id, company_name, company_description) values (83, 83, 'Voomm', 'Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue.');
insert into company (company_id, address_id, company_name, company_description) values (84, 84, 'Livetube', 'Nullam orci pede, venenatis non, sodales sed, tincidunt eu, felis. Fusce posuere felis sed lacus. Morbi sem mauris, laoreet ut, rhoncus aliquet, pulvinar sed, nisl.');
insert into company (company_id, address_id, company_name, company_description) values (85, 85, 'Thoughtblab', 'Donec dapibus. Duis at velit eu est congue elementum.');
insert into company (company_id, address_id, company_name, company_description) values (86, 86, 'Teklist', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus.');
insert into company (company_id, address_id, company_name, company_description) values (87, 87, 'Gabcube', 'Nulla ut erat id mauris vulputate elementum. Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit.');
insert into company (company_id, address_id, company_name, company_description) values (88, 88, 'Mybuzz', 'Donec posuere metus vitae ipsum.');
insert into company (company_id, address_id, company_name, company_description) values (89, 89, 'Skalith', 'Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum. Mauris ullamcorper purus sit amet nulla.');
insert into company (company_id, address_id, company_name, company_description) values (90, 90, 'Gigabox', 'Pellentesque eget nunc. Donec quis orci eget orci vehicula condimentum. Curabitur in libero ut massa volutpat convallis.');
insert into company (company_id, address_id, company_name, company_description) values (91, 91, 'Mynte', 'Aliquam sit amet diam in magna bibendum imperdiet.');
insert into company (company_id, address_id, company_name, company_description) values (92, 92, 'Buzzshare', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.');
insert into company (company_id, address_id, company_name, company_description) values (93, 93, 'Pixonyx', 'Morbi vel lectus in quam fringilla rhoncus.');
insert into company (company_id, address_id, company_name, company_description) values (94, 94, 'Leenti', 'Donec quis orci eget orci vehicula condimentum. Curabitur in libero ut massa volutpat convallis. Morbi odio odio, elementum eu, interdum eu, tincidunt in, leo. Maecenas pulvinar lobortis est.');
insert into company (company_id, address_id, company_name, company_description) values (95, 95, 'Browsetype', 'In eleifend quam a odio.');
insert into company (company_id, address_id, company_name, company_description) values (96, 96, 'Miboo', 'Etiam faucibus cursus urna. Ut tellus. Nulla ut erat id mauris vulputate elementum. Nullam varius. Nulla facilisi.');
insert into company (company_id, address_id, company_name, company_description) values (97, 97, 'Tagchat', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque.');
insert into company (company_id, address_id, company_name, company_description) values (98, 98, 'Kwimbee', 'In quis justo. Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet.');
insert into company (company_id, address_id, company_name, company_description) values (99, 99, 'Yata', 'Nunc nisl. Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa. Donec dapibus. Duis at velit eu est congue elementum.');
insert into company (company_id, address_id, company_name, company_description) values (100, 100, 'Innotype', 'In hac habitasse platea dictumst.');

-- end of company table

-- team table

insert into team (team_id, person_id, company_id, team_name, command_description) values (1, 1, 1, 'Product Management', 'Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (2, 2, 2, 'Engineering', 'Nulla suscipit ligula in lacus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (3, 3, 3, 'Accounting', 'Integer a nibh.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (4, 4, 4, 'Research and Development', 'Vivamus tortor.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (5, 5, 5, 'Research and Development', 'Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (6, 6, 6, 'Services', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (7, 7, 7, 'Engineering', 'Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (8, 8, 8, 'Accounting', 'Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (9, 9, 9, 'Human Resources', 'Aenean fermentum. Donec ut mauris eget massa tempor convallis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (10, 10, 10, 'Services', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (11, 11, 11, 'Research and Development', 'Praesent blandit lacinia erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (12, 12, 12, 'Accounting', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (13, 13, 13, 'Accounting', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue. Vestibulum rutrum rutrum neque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (14, 14, 14, 'Accounting', 'Sed sagittis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (15, 15, 15, 'Marketing', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (16, 16, 16, 'Sales', 'Fusce consequat. Nulla nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (17, 17, 17, 'Accounting', 'Etiam vel augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (18, 18, 18, 'Engineering', 'Nullam sit amet turpis elementum ligula vehicula consequat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (19, 19, 19, 'Product Management', 'Sed ante. Vivamus tortor.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (20, 20, 20, 'Research and Development', 'In hac habitasse platea dictumst.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (21, 21, 21, 'Training', 'Sed accumsan felis. Ut at dolor quis odio consequat varius.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (22, 22, 22, 'Business Development', 'Proin eu mi. Nulla ac enim. In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (23, 23, 23, 'Human Resources', 'Proin at turpis a pede posuere nonummy. Integer non velit. Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (24, 24, 24, 'Business Development', 'Nullam varius.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (25, 25, 25, 'Product Management', 'Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (26, 26, 26, 'Legal', 'Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (27, 27, 27, 'Business Development', 'Vestibulum rutrum rutrum neque. Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (28, 28, 28, 'Business Development', 'Integer a nibh. In quis justo. Maecenas rhoncus aliquam lacus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (29, 29, 29, 'Product Management', 'In congue. Etiam justo. Etiam pretium iaculis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (30, 30, 30, 'Marketing', 'Nulla justo. Aliquam quis turpis eget elit sodales scelerisque. Mauris sit amet eros.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (31, 31, 31, 'Accounting', 'In hac habitasse platea dictumst.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (32, 32, 32, 'Marketing', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (33, 33, 33, 'Services', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti. Nullam porttitor lacus at turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (34, 34, 34, 'Product Management', 'Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (35, 35, 35, 'Engineering', 'Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (36, 36, 36, 'Training', 'Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (37, 37, 37, 'Business Development', 'Morbi a ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (38, 38, 38, 'Sales', 'Morbi quis tortor id nulla ultrices aliquet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (39, 39, 39, 'Engineering', 'Etiam faucibus cursus urna.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (40, 40, 40, 'Human Resources', 'Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (41, 41, 41, 'Services', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (42, 42, 42, 'Training', 'Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (43, 43, 43, 'Marketing', 'Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (44, 44, 44, 'Services', 'Duis ac nibh.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (45, 45, 45, 'Accounting', 'Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (46, 46, 46, 'Accounting', 'Maecenas ut massa quis augue luctus tincidunt.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (47, 47, 47, 'Product Management', 'Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (48, 48, 48, 'Support', 'Nulla tempus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (49, 49, 49, 'Engineering', 'Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (50, 50, 50, 'Training', 'Sed vel enim sit amet nunc viverra dapibus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (51, 51, 51, 'Legal', 'Nunc purus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (52, 52, 52, 'Engineering', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (53, 53, 53, 'Engineering', 'Integer ac leo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (54, 54, 54, 'Research and Development', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (55, 55, 55, 'Research and Development', 'Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (56, 56, 56, 'Training', 'Morbi non quam nec dui luctus rutrum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (57, 57, 57, 'Product Management', 'Aenean lectus. Pellentesque eget nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (58, 58, 58, 'Human Resources', 'Vivamus tortor. Duis mattis egestas metus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (59, 59, 59, 'Human Resources', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (60, 60, 60, 'Engineering', 'Aliquam non mauris. Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (61, 61, 61, 'Accounting', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (62, 62, 62, 'Research and Development', 'Integer a nibh. In quis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (63, 63, 63, 'Research and Development', 'Pellentesque ultrices mattis odio.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (64, 64, 64, 'Support', 'Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (65, 65, 65, 'Support', 'Sed vel enim sit amet nunc viverra dapibus. Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (66, 66, 66, 'Product Management', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (67, 67, 67, 'Human Resources', 'Morbi a ipsum. Integer a nibh. In quis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (68, 68, 68, 'Engineering', 'Suspendisse potenti.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (69, 69, 69, 'Services', 'Pellentesque at nulla.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (70, 70, 70, 'Support', 'Aliquam non mauris. Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (71, 71, 71, 'Engineering', 'Phasellus in felis. Donec semper sapien a libero. Nam dui.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (72, 72, 72, 'Services', 'Praesent id massa id nisl venenatis lacinia. Aenean sit amet justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (73, 73, 73, 'Support', 'Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (74, 74, 74, 'Business Development', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (75, 75, 75, 'Research and Development', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (76, 76, 76, 'Human Resources', 'Morbi porttitor lorem id ligula.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (77, 77, 77, 'Engineering', 'Aenean lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (78, 78, 78, 'Human Resources', 'Aliquam quis turpis eget elit sodales scelerisque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (79, 79, 79, 'Business Development', 'Pellentesque ultrices mattis odio.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (80, 80, 80, 'Human Resources', 'Aliquam erat volutpat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (81, 81, 81, 'Sales', 'Vivamus in felis eu sapien cursus vestibulum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (82, 82, 82, 'Accounting', 'Phasellus id sapien in sapien iaculis congue. Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (83, 83, 83, 'Engineering', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum. Aliquam non mauris.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (84, 84, 84, 'Product Management', 'Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (85, 85, 85, 'Services', 'Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (86, 86, 86, 'Services', 'Aenean lectus. Pellentesque eget nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (87, 87, 87, 'Support', 'Nunc nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (88, 88, 88, 'Accounting', 'In est risus, auctor sed, tristique in, tempus sit amet, sem.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (89, 89, 89, 'Engineering', 'Maecenas pulvinar lobortis est. Phasellus sit amet erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (90, 90, 90, 'Product Management', 'Nunc nisl. Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (91, 91, 91, 'Research and Development', 'Duis consequat dui nec nisi volutpat eleifend. Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (92, 92, 92, 'Human Resources', 'Suspendisse potenti. Cras in purus eu magna vulputate luctus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (93, 93, 93, 'Business Development', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (94, 94, 94, 'Marketing', 'Etiam vel augue. Vestibulum rutrum rutrum neque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (95, 95, 95, 'Support', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (96, 96, 96, 'Engineering', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin risus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (97, 97, 97, 'Marketing', 'Cras in purus eu magna vulputate luctus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (98, 98, 98, 'Support', 'Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (99, 99, 99, 'Accounting', 'Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros. Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (100, 100, 100, 'Training', 'Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.');


insert into team (team_id, person_id, company_id, team_name, command_description) values (101, 101, 1, 'Product Management', 'Praesent lectus. Vestibulum quam sapien, varius ut, blandit non, interdum in, ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (102, 101, 2, 'Engineering', 'Nulla suscipit ligula in lacus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (103, 101, 3, 'Accounting', 'Integer a nibh.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (104, 101, 4, 'Research and Development', 'Vivamus tortor.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (105, 101, 5, 'Research and Development', 'Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (106, 101, 6, 'Services', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (107, 101, 7, 'Engineering', 'Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (108, 101, 8, 'Accounting', 'Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (109, 101, 9, 'Human Resources', 'Aenean fermentum. Donec ut mauris eget massa tempor convallis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (110, 101, 10, 'Services', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (111, 101, 11, 'Research and Development', 'Praesent blandit lacinia erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (112, 101, 12, 'Accounting', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (113, 101, 13, 'Accounting', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue. Vestibulum rutrum rutrum neque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (114, 101, 14, 'Accounting', 'Sed sagittis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (115, 101, 15, 'Marketing', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (116, 101, 16, 'Sales', 'Fusce consequat. Nulla nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (117, 101, 17, 'Accounting', 'Etiam vel augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (118, 101, 18, 'Engineering', 'Nullam sit amet turpis elementum ligula vehicula consequat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (119, 101, 19, 'Product Management', 'Sed ante. Vivamus tortor.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (120, 101, 20, 'Research and Development', 'In hac habitasse platea dictumst.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (121, 101, 21, 'Training', 'Sed accumsan felis. Ut at dolor quis odio consequat varius.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (122, 101, 22, 'Business Development', 'Proin eu mi. Nulla ac enim. In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (123, 101, 23, 'Human Resources', 'Proin at turpis a pede posuere nonummy. Integer non velit. Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (124, 101, 24, 'Business Development', 'Nullam varius.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (125, 101, 25, 'Product Management', 'Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (126, 101, 26, 'Legal', 'Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (127, 101, 27, 'Business Development', 'Vestibulum rutrum rutrum neque. Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (128, 101, 28, 'Business Development', 'Integer a nibh. In quis justo. Maecenas rhoncus aliquam lacus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (129, 101, 29, 'Product Management', 'In congue. Etiam justo. Etiam pretium iaculis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (130, 101, 30, 'Marketing', 'Nulla justo. Aliquam quis turpis eget elit sodales scelerisque. Mauris sit amet eros.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (131, 101, 31, 'Accounting', 'In hac habitasse platea dictumst.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (132, 101, 32, 'Marketing', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (133, 101, 33, 'Services', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam. Suspendisse potenti. Nullam porttitor lacus at turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (134, 101, 34, 'Product Management', 'Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (135, 101, 35, 'Engineering', 'Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (136, 101, 36, 'Training', 'Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (137, 101, 37, 'Business Development', 'Morbi a ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (138, 101, 38, 'Sales', 'Morbi quis tortor id nulla ultrices aliquet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (139, 101, 39, 'Engineering', 'Etiam faucibus cursus urna.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (140, 101, 40, 'Human Resources', 'Nullam varius. Nulla facilisi. Cras non velit nec nisi vulputate nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (141, 101, 41, 'Services', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (142, 101, 42, 'Training', 'Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (143, 101, 43, 'Marketing', 'Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (144, 101, 44, 'Services', 'Duis ac nibh.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (145, 101, 45, 'Accounting', 'Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (146, 101, 46, 'Accounting', 'Maecenas ut massa quis augue luctus tincidunt.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (147, 101, 47, 'Product Management', 'Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (148, 101, 48, 'Support', 'Nulla tempus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (149, 101, 49, 'Engineering', 'Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (150, 101, 50, 'Training', 'Sed vel enim sit amet nunc viverra dapibus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (151, 101, 51, 'Legal', 'Nunc purus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (152, 101, 52, 'Engineering', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (153, 101, 53, 'Engineering', 'Integer ac leo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (154, 101, 54, 'Research and Development', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (155, 101, 55, 'Research and Development', 'Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (156, 101, 56, 'Training', 'Morbi non quam nec dui luctus rutrum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (157, 101, 57, 'Product Management', 'Aenean lectus. Pellentesque eget nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (158, 101, 58, 'Human Resources', 'Vivamus tortor. Duis mattis egestas metus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (159, 101, 59, 'Human Resources', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (160, 101, 60, 'Engineering', 'Aliquam non mauris. Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (161, 101, 61, 'Accounting', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (162, 101, 62, 'Research and Development', 'Integer a nibh. In quis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (163, 101, 63, 'Research and Development', 'Pellentesque ultrices mattis odio.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (164, 101, 64, 'Support', 'Morbi ut odio. Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (165, 101, 65, 'Support', 'Sed vel enim sit amet nunc viverra dapibus. Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (166, 101, 66, 'Product Management', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (167, 101, 67, 'Human Resources', 'Morbi a ipsum. Integer a nibh. In quis justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (168, 101, 68, 'Engineering', 'Suspendisse potenti.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (169, 101, 69, 'Services', 'Pellentesque at nulla.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (170, 101, 70, 'Support', 'Aliquam non mauris. Morbi non lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (171, 101, 71, 'Engineering', 'Phasellus in felis. Donec semper sapien a libero. Nam dui.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (172, 101, 72, 'Services', 'Praesent id massa id nisl venenatis lacinia. Aenean sit amet justo.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (173, 101, 73, 'Support', 'Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (174, 101, 74, 'Business Development', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (175, 101, 75, 'Research and Development', 'Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (176, 101, 76, 'Human Resources', 'Morbi porttitor lorem id ligula.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (177, 101, 77, 'Engineering', 'Aenean lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (178, 101, 78, 'Human Resources', 'Aliquam quis turpis eget elit sodales scelerisque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (179, 101, 79, 'Business Development', 'Pellentesque ultrices mattis odio.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (180, 101, 80, 'Human Resources', 'Aliquam erat volutpat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (181, 101, 81, 'Sales', 'Vivamus in felis eu sapien cursus vestibulum.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (182, 101, 82, 'Accounting', 'Phasellus id sapien in sapien iaculis congue. Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (183, 101, 83, 'Engineering', 'Nullam porttitor lacus at turpis. Donec posuere metus vitae ipsum. Aliquam non mauris.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (184, 101, 84, 'Product Management', 'Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (185, 101, 85, 'Services', 'Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (186, 101, 86, 'Services', 'Aenean lectus. Pellentesque eget nunc.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (187, 101, 87, 'Support', 'Nunc nisl.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (188, 101, 88, 'Accounting', 'In est risus, auctor sed, tristique in, tempus sit amet, sem.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (189, 101, 89, 'Engineering', 'Maecenas pulvinar lobortis est. Phasellus sit amet erat.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (190, 101, 90, 'Product Management', 'Nunc nisl. Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (191, 101, 91, 'Research and Development', 'Duis consequat dui nec nisi volutpat eleifend. Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (192, 101, 92, 'Human Resources', 'Suspendisse potenti. Cras in purus eu magna vulputate luctus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (193, 101, 93, 'Business Development', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (194, 101, 94, 'Marketing', 'Etiam vel augue. Vestibulum rutrum rutrum neque.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (195, 101, 95, 'Support', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (196, 101, 96, 'Engineering', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin risus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (197, 101, 97, 'Marketing', 'Cras in purus eu magna vulputate luctus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (198, 101, 98, 'Support', 'Suspendisse ornare consequat lectus.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (199, 101, 99, 'Accounting', 'Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros. Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue.');
insert into team (team_id, person_id, company_id, team_name, command_description) values (200, 101, 100, 'Training', 'Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.');

-- command table end

-- vacancy table

insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (1, 1, 'Actuary', 'Quisque arcu libero, rutrum ac, lobortis vel, dapibus at, diam.', '$272899.56', 3, 'Contract', '2022/02/12', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (2, 2, 'Chief Design Engineer', 'Suspendisse accumsan tortor quis turpis. Sed ante. Vivamus tortor. Duis mattis egestas metus.', '$437508.54', 18, 'Part-time', '2020/04/29', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (3, 3, 'Social Worker', 'Nunc rhoncus dui vel sem. Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus.', '$775156.73', 8, 'Full-time', '2020/10/27', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (4, 4, 'Engineer II', 'Nunc nisl. Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.', '$276754.47', 16, 'Contract', '2021/10/25', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (5, 5, 'Safety Technician III', 'Integer a nibh. In quis justo. Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet.', '$811858.17', 5, 'Full-time', '2021/06/14', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (6, 6, 'Tax Accountant', 'Suspendisse ornare consequat lectus.', '$11966.51', 17, 'Intern', '2023/01/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (7, 7, 'Structural Analysis Engineer', 'In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit.', '$362861.41', 16, 'Intern', '2020/03/12', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (8, 8, 'Environmental Tech', 'Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.', '$564087.84', 18, 'Intern', '2023/02/20', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (9, 9, 'Design Engineer', 'Donec ut dolor. Morbi vel lectus in quam fringilla rhoncus. Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. Integer aliquet, massa id lobortis convallis, tortor risus dapibus augue, vel accumsan tellus nisi eu orci.', '$467907.61', 20, 'Intern', '2020/06/02', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (10, 10, 'Health Coach I', 'Maecenas tincidunt lacus at velit.', '$457336.68', 5, 'Part-time', '2023/04/18', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (11, 11, 'VP Sales', 'Suspendisse ornare consequat lectus.', '$553509.80', 8, 'Contract', '2021/08/05', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (12, 12, 'Research Assistant III', 'Quisque id justo sit amet sapien dignissim vestibulum. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est.', '$377825.90', 4, 'Contract', '2020/03/11', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (13, 13, 'Accountant II', 'Morbi non quam nec dui luctus rutrum. Nulla tellus.', '$376501.19', 12, 'Part-time', '2021/12/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (14, 14, 'Marketing Assistant', 'Pellentesque viverra pede ac diam. Cras pellentesque volutpat dui.', '$228918.47', 20, 'Part-time', '2020/09/12', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (15, 15, 'Civil Engineer', 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin interdum mauris non ligula pellentesque ultrices.', '$602534.50', 20, 'Intern', '2022/02/02', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (16, 16, 'Analyst Programmer', 'In hac habitasse platea dictumst. Maecenas ut massa quis augue luctus tincidunt.', '$569518.04', 2, 'Contract', '2023/03/09', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (17, 17, 'VP Marketing', 'Ut tellus. Nulla ut erat id mauris vulputate elementum.', '$516533.58', 5, 'Full-time', '2021/07/15', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (18, 18, 'Research Assistant I', 'Nulla justo. Aliquam quis turpis eget elit sodales scelerisque. Mauris sit amet eros. Suspendisse accumsan tortor quis turpis.', '$56399.33', 4, 'Trainee', '2021/07/01', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (19, 19, 'Software Test Engineer II', 'Morbi a ipsum. Integer a nibh.', '$41213.87', 8, 'Trainee', '2022/05/28', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (20, 20, 'VP Sales', 'Nunc rhoncus dui vel sem. Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci. Nullam molestie nibh in lectus.', '$60155.94', 18, 'Part-time', '2021/02/24', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (21, 21, 'Senior Cost Accountant', 'Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi.', '$123790.71', 12, 'Part-time', '2022/10/12', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (22, 22, 'General Manager', 'Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa. Donec dapibus. Duis at velit eu est congue elementum.', '$463969.93', 15, 'Trainee', '2022/10/24', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (23, 23, 'Electrical Engineer', 'Nulla tempus. Vivamus in felis eu sapien cursus vestibulum.', '$94918.63', 15, 'Intern', '2021/10/21', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (24, 24, 'Software Consultant', 'Curabitur in libero ut massa volutpat convallis. Morbi odio odio, elementum eu, interdum eu, tincidunt in, leo. Maecenas pulvinar lobortis est. Phasellus sit amet erat.', '$827100.37', 7, 'Contract', '2020/10/08', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (25, 25, 'Cost Accountant', 'Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi. Nam ultrices, libero non mattis pulvinar, nulla pede ullamcorper augue, a suscipit nulla elit ac nulla.', '$800581.49', 2, 'Full-time', '2023/03/06', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (26, 26, 'Human Resources Manager', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.', '$321690.70', 12, 'Contract', '2020/01/30', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (27, 27, 'Structural Engineer', 'Nulla tempus.', '$734701.64', 9, 'Part-time', '2020/12/08', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (28, 28, 'Quality Control Specialist', 'Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo. Pellentesque viverra pede ac diam. Cras pellentesque volutpat dui.', '$20283.56', 18, 'Trainee', '2020/04/10', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (29, 29, 'Software Engineer IV', 'Mauris lacinia sapien quis libero.', '$102659.44', 5, 'Part-time', '2020/09/20', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (30, 30, 'Analyst Programmer', 'Etiam faucibus cursus urna. Ut tellus.', '$869653.93', 1, 'Trainee', '2020/05/01', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (31, 31, 'Senior Quality Engineer', 'Vivamus vel nulla eget eros elementum pellentesque.', '$585466.97', 19, 'Part-time', '2021/09/16', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (32, 32, 'VP Accounting', 'Nulla tempus. Vivamus in felis eu sapien cursus vestibulum.', '$299298.33', 19, 'Full-time', '2020/03/13', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (33, 33, 'Director of Sales', 'Suspendisse potenti. Cras in purus eu magna vulputate luctus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien.', '$286188.67', 5, 'Contract', '2022/11/15', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (34, 34, 'Design Engineer', 'Nunc nisl.', '$40335.57', 17, 'Part-time', '2022/07/10', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (35, 35, 'Executive Secretary', 'Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.', '$180037.01', 14, 'Part-time', '2020/10/22', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (36, 36, 'Web Developer IV', 'Phasellus in felis.', '$25888.60', 19, 'Part-time', '2020/03/01', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (37, 37, 'Budget/Accounting Analyst III', 'In eleifend quam a odio. In hac habitasse platea dictumst. Maecenas ut massa quis augue luctus tincidunt.', '$96573.61', 16, 'Part-time', '2023/01/15', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (38, 38, 'Biostatistician IV', 'Vivamus vestibulum sagittis sapien.', '$605394.90', 2, 'Full-time', '2021/12/25', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (39, 39, 'Paralegal', 'Vestibulum rutrum rutrum neque. Aenean auctor gravida sem. Praesent id massa id nisl venenatis lacinia.', '$843038.93', 8, 'Contract', '2022/06/19', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (40, 40, 'Safety Technician IV', 'Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue. Etiam justo.', '$643168.71', 7, 'Part-time', '2021/01/23', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (41, 41, 'GIS Technical Architect', 'In hac habitasse platea dictumst.', '$723548.38', 1, 'Contract', '2022/07/27', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (42, 42, 'Civil Engineer', 'Proin eu mi. Nulla ac enim. In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem.', '$972247.24', 6, 'Part-time', '2022/05/27', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (43, 43, 'Clinical Specialist', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.', '$286336.85', 17, 'Full-time', '2022/10/07', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (44, 44, 'Payment Adjustment Coordinator', 'Pellentesque viverra pede ac diam. Cras pellentesque volutpat dui. Maecenas tristique, est et tempus semper, est quam pharetra magna, ac consequat metus sapien ut nunc. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Mauris viverra diam vitae quam.', '$963248.04', 7, 'Contract', '2020/04/03', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (45, 45, 'Director of Sales', 'Phasellus id sapien in sapien iaculis congue. Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl.', '$985246.19', 20, 'Full-time', '2023/01/01', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (46, 46, 'Nurse Practicioner', 'Nulla suscipit ligula in lacus.', '$42671.89', 16, 'Intern', '2020/06/28', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (47, 47, 'Geological Engineer', 'Aenean sit amet justo.', '$258058.46', 14, 'Contract', '2022/05/11', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (48, 48, 'Computer Systems Analyst IV', 'Mauris ullamcorper purus sit amet nulla. Quisque arcu libero, rutrum ac, lobortis vel, dapibus at, diam.', '$976794.60', 2, 'Intern', '2020/02/21', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (49, 49, 'Project Manager', 'In est risus, auctor sed, tristique in, tempus sit amet, sem.', '$859867.67', 3, 'Trainee', '2022/11/05', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (50, 50, 'Director of Sales', 'Morbi sem mauris, laoreet ut, rhoncus aliquet, pulvinar sed, nisl. Nunc rhoncus dui vel sem. Sed sagittis. Nam congue, risus semper porta volutpat, quam pede lobortis ligula, sit amet eleifend pede libero quis orci.', '$528984.81', 6, 'Intern', '2022/04/05', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (51, 51, 'Staff Accountant I', 'Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis. Ut at dolor quis odio consequat varius. Integer ac leo.', '$840617.48', 17, 'Intern', '2022/10/08', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (52, 52, 'Software Engineer III', 'Phasellus sit amet erat. Nulla tempus. Vivamus in felis eu sapien cursus vestibulum.', '$353208.33', 5, 'Contract', '2021/02/06', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (53, 53, 'Compensation Analyst', 'Vivamus in felis eu sapien cursus vestibulum. Proin eu mi.', '$531496.54', 13, 'Part-time', '2022/01/14', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (54, 54, 'Editor', 'Ut at dolor quis odio consequat varius. Integer ac leo. Pellentesque ultrices mattis odio.', '$145439.88', 5, 'Trainee', '2023/04/18', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (55, 55, 'Assistant Manager', 'Donec posuere metus vitae ipsum. Aliquam non mauris. Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet.', '$420920.99', 2, 'Trainee', '2021/07/11', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (56, 56, 'VP Marketing', 'Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus.', '$787370.62', 18, 'Full-time', '2020/11/02', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (57, 57, 'Web Designer III', 'Suspendisse accumsan tortor quis turpis. Sed ante. Vivamus tortor. Duis mattis egestas metus.', '$190745.03', 4, 'Full-time', '2022/01/29', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (58, 58, 'Programmer Analyst I', 'Donec semper sapien a libero. Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis.', '$88465.51', 8, 'Trainee', '2021/05/08', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (59, 59, 'Software Consultant', 'Aenean fermentum. Donec ut mauris eget massa tempor convallis.', '$825750.23', 5, 'Contract', '2020/11/04', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (60, 60, 'Electrical Engineer', 'Mauris lacinia sapien quis libero.', '$243958.66', 4, 'Full-time', '2022/12/31', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (61, 61, 'Geological Engineer', 'In quis justo. Maecenas rhoncus aliquam lacus. Morbi quis tortor id nulla ultrices aliquet. Maecenas leo odio, condimentum id, luctus nec, molestie sed, justo.', '$353969.70', 5, 'Trainee', '2020/06/12', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (62, 62, 'Data Coordinator', 'Nullam sit amet turpis elementum ligula vehicula consequat. Morbi a ipsum. Integer a nibh.', '$645375.58', 1, 'Trainee', '2022/07/10', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (63, 63, 'Internal Auditor', 'Aenean lectus. Pellentesque eget nunc.', '$147533.81', 2, 'Part-time', '2021/12/10', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (64, 64, 'Web Designer I', 'Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem. Fusce consequat.', '$903864.11', 5, 'Contract', '2020/07/19', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (65, 65, 'Librarian', 'Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.', '$285123.99', 8, 'Part-time', '2020/08/06', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (66, 66, 'Community Outreach Specialist', 'Nunc nisl. Duis bibendum, felis sed interdum venenatis, turpis enim blandit mi, in porttitor pede justo eu massa.', '$994665.56', 12, 'Part-time', '2021/12/22', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (67, 67, 'Financial Advisor', 'Vivamus in felis eu sapien cursus vestibulum. Proin eu mi. Nulla ac enim.', '$429275.96', 20, 'Full-time', '2020/09/19', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (68, 68, 'Programmer III', 'Pellentesque at nulla. Suspendisse potenti.', '$831245.76', 14, 'Full-time', '2020/12/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (69, 69, 'Senior Cost Accountant', 'Cras in purus eu magna vulputate luctus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', '$427833.60', 5, 'Trainee', '2022/11/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (70, 70, 'Chemical Engineer', 'Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.', '$59834.90', 20, 'Contract', '2022/12/11', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (71, 71, 'Research Assistant III', 'Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue. Aliquam erat volutpat. In congue.', '$776658.40', 6, 'Intern', '2021/04/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (72, 72, 'Chemical Engineer', 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', '$161359.85', 14, 'Contract', '2022/01/06', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (73, 73, 'Information Systems Manager', 'Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus.', '$330880.48', 7, 'Contract', '2020/10/20', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (74, 74, 'Developer IV', 'Integer ac neque.', '$806242.45', 20, 'Trainee', '2020/11/08', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (75, 75, 'Operator', 'Morbi porttitor lorem id ligula. Suspendisse ornare consequat lectus. In est risus, auctor sed, tristique in, tempus sit amet, sem.', '$399277.89', 12, 'Intern', '2021/04/18', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (76, 76, 'VP Accounting', 'Donec semper sapien a libero. Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis.', '$627511.64', 8, 'Full-time', '2020/03/07', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (77, 77, 'Nurse Practicioner', 'Proin eu mi.', '$237918.74', 9, 'Full-time', '2020/07/04', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (78, 78, 'Recruiting Manager', 'Aenean lectus. Pellentesque eget nunc. Donec quis orci eget orci vehicula condimentum. Curabitur in libero ut massa volutpat convallis.', '$643467.48', 13, 'Intern', '2022/01/18', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (79, 79, 'Safety Technician IV', 'Vivamus metus arcu, adipiscing molestie, hendrerit at, vulputate vitae, nisl. Aenean lectus. Pellentesque eget nunc.', '$168045.71', 11, 'Intern', '2020/02/09', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (80, 80, 'Nuclear Power Engineer', 'Ut at dolor quis odio consequat varius. Integer ac leo. Pellentesque ultrices mattis odio. Donec vitae nisi.', '$835093.69', 20, 'Intern', '2022/07/06', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (81, 81, 'Design Engineer', 'Mauris sit amet eros. Suspendisse accumsan tortor quis turpis. Sed ante.', '$61894.51', 15, 'Part-time', '2022/04/03', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (82, 82, 'Safety Technician IV', 'Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.', '$839273.52', 11, 'Full-time', '2022/11/17', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (83, 83, 'Chemical Engineer', 'Sed vel enim sit amet nunc viverra dapibus. Nulla suscipit ligula in lacus. Curabitur at ipsum ac tellus semper interdum.', '$515717.60', 7, 'Trainee', '2021/08/29', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (84, 84, 'Dental Hygienist', 'In hac habitasse platea dictumst. Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat.', '$63187.59', 8, 'Contract', '2021/12/01', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (85, 85, 'Food Chemist', 'Suspendisse potenti. Cras in purus eu magna vulputate luctus.', '$55656.18', 5, 'Trainee', '2021/03/03', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (86, 86, 'Nurse', 'Quisque ut erat.', '$189941.81', 7, 'Intern', '2020/08/09', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (87, 87, 'Administrative Officer', 'Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue.', '$189299.58', 19, 'Contract', '2022/04/27', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (88, 88, 'Budget/Accounting Analyst III', 'Nulla tempus. Vivamus in felis eu sapien cursus vestibulum. Proin eu mi. Nulla ac enim.', '$716152.27', 5, 'Part-time', '2022/04/13', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (89, 89, 'Analyst Programmer', 'Aliquam erat volutpat. In congue. Etiam justo. Etiam pretium iaculis justo.', '$472708.74', 16, 'Contract', '2020/07/23', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (90, 90, 'Legal Assistant', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla dapibus dolor vel est. Donec odio justo, sollicitudin ut, suscipit a, feugiat et, eros. Vestibulum ac est lacinia nisi venenatis tristique. Fusce congue, diam id ornare imperdiet, sapien urna pretium nisl, ut volutpat sapien arcu sed augue.', '$723664.13', 5, 'Intern', '2021/04/14', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (91, 91, 'Mechanical Systems Engineer', 'Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet.', '$268456.80', 20, 'Trainee', '2020/11/19', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (92, 92, 'Junior Executive', 'Cras non velit nec nisi vulputate nonummy. Maecenas tincidunt lacus at velit. Vivamus vel nulla eget eros elementum pellentesque. Quisque porta volutpat erat.', '$483153.82', 1, 'Intern', '2022/11/19', 'Open');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (93, 93, 'Financial Analyst', 'Proin at turpis a pede posuere nonummy. Integer non velit. Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue.', '$365763.22', 15, 'Part-time', '2020/01/13', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (94, 94, 'Tax Accountant', 'Nulla ut erat id mauris vulputate elementum.', '$184230.61', 9, 'Contract', '2021/08/06', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (95, 95, 'Quality Control Specialist', 'Fusce consequat. Nulla nisl.', '$291631.99', 9, 'Trainee', '2020/04/11', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (96, 96, 'Software Test Engineer I', 'Quisque porta volutpat erat. Quisque erat eros, viverra eget, congue eget, semper rutrum, nulla.', '$60652.49', 11, 'Intern', '2022/04/04', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (97, 97, 'VP Sales', 'Cras mi pede, malesuada in, imperdiet et, commodo vulputate, justo.', '$952838.34', 6, 'Trainee', '2022/10/21', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (98, 98, 'Business Systems Development Analyst', 'Aliquam non mauris. Morbi non lectus.', '$947109.69', 13, 'Intern', '2022/08/25', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (99, 99, 'Systems Administrator IV', 'In eleifend quam a odio. In hac habitasse platea dictumst. Maecenas ut massa quis augue luctus tincidunt.', '$785503.65', 17, 'Part-time', '2022/07/13', 'Close');
insert into vacancy (vacancy_id, team_id, job_title, vacancy_description, salary, experience_level, type_of_employment, job_posting_date, job_status) values (100, 100, 'GIS Technical Architect', 'Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque. Duis bibendum. Morbi non quam nec dui luctus rutrum.', '$655677.94', 1, 'Trainee', '2023/02/22', 'Open');

-- end of vacancy

-- end of vacancy table

-- vacancy_candidate table

insert into vacancy_candidate (vacancy_id, person_id) values (1, 1);
insert into vacancy_candidate (vacancy_id, person_id) values (2, 2);
insert into vacancy_candidate (vacancy_id, person_id) values (3, 3);
insert into vacancy_candidate (vacancy_id, person_id) values (4, 4);
insert into vacancy_candidate (vacancy_id, person_id) values (5, 5);
insert into vacancy_candidate (vacancy_id, person_id) values (6, 6);
insert into vacancy_candidate (vacancy_id, person_id) values (7, 7);
insert into vacancy_candidate (vacancy_id, person_id) values (8, 8);
insert into vacancy_candidate (vacancy_id, person_id) values (9, 9);
insert into vacancy_candidate (vacancy_id, person_id) values (10, 10);
insert into vacancy_candidate (vacancy_id, person_id) values (11, 11);
insert into vacancy_candidate (vacancy_id, person_id) values (12, 12);
insert into vacancy_candidate (vacancy_id, person_id) values (13, 13);
insert into vacancy_candidate (vacancy_id, person_id) values (14, 14);
insert into vacancy_candidate (vacancy_id, person_id) values (15, 15);
insert into vacancy_candidate (vacancy_id, person_id) values (16, 16);
insert into vacancy_candidate (vacancy_id, person_id) values (17, 17);
insert into vacancy_candidate (vacancy_id, person_id) values (18, 18);
insert into vacancy_candidate (vacancy_id, person_id) values (19, 19);
insert into vacancy_candidate (vacancy_id, person_id) values (20, 20);
insert into vacancy_candidate (vacancy_id, person_id) values (21, 21);
insert into vacancy_candidate (vacancy_id, person_id) values (22, 22);
insert into vacancy_candidate (vacancy_id, person_id) values (23, 23);
insert into vacancy_candidate (vacancy_id, person_id) values (24, 24);
insert into vacancy_candidate (vacancy_id, person_id) values (25, 25);
insert into vacancy_candidate (vacancy_id, person_id) values (26, 26);
insert into vacancy_candidate (vacancy_id, person_id) values (27, 27);
insert into vacancy_candidate (vacancy_id, person_id) values (28, 28);
insert into vacancy_candidate (vacancy_id, person_id) values (29, 29);
insert into vacancy_candidate (vacancy_id, person_id) values (30, 30);
insert into vacancy_candidate (vacancy_id, person_id) values (31, 31);
insert into vacancy_candidate (vacancy_id, person_id) values (32, 32);
insert into vacancy_candidate (vacancy_id, person_id) values (33, 33);
insert into vacancy_candidate (vacancy_id, person_id) values (34, 34);
insert into vacancy_candidate (vacancy_id, person_id) values (35, 35);
insert into vacancy_candidate (vacancy_id, person_id) values (36, 36);
insert into vacancy_candidate (vacancy_id, person_id) values (37, 37);
insert into vacancy_candidate (vacancy_id, person_id) values (38, 38);
insert into vacancy_candidate (vacancy_id, person_id) values (39, 39);
insert into vacancy_candidate (vacancy_id, person_id) values (40, 40);
insert into vacancy_candidate (vacancy_id, person_id) values (41, 41);
insert into vacancy_candidate (vacancy_id, person_id) values (42, 42);
insert into vacancy_candidate (vacancy_id, person_id) values (43, 43);
insert into vacancy_candidate (vacancy_id, person_id) values (44, 44);
insert into vacancy_candidate (vacancy_id, person_id) values (45, 45);
insert into vacancy_candidate (vacancy_id, person_id) values (46, 46);
insert into vacancy_candidate (vacancy_id, person_id) values (47, 47);
insert into vacancy_candidate (vacancy_id, person_id) values (48, 48);
insert into vacancy_candidate (vacancy_id, person_id) values (49, 49);
insert into vacancy_candidate (vacancy_id, person_id) values (50, 50);
insert into vacancy_candidate (vacancy_id, person_id) values (51, 51);
insert into vacancy_candidate (vacancy_id, person_id) values (52, 52);
insert into vacancy_candidate (vacancy_id, person_id) values (53, 53);
insert into vacancy_candidate (vacancy_id, person_id) values (54, 54);
insert into vacancy_candidate (vacancy_id, person_id) values (55, 55);
insert into vacancy_candidate (vacancy_id, person_id) values (56, 56);
insert into vacancy_candidate (vacancy_id, person_id) values (57, 57);
insert into vacancy_candidate (vacancy_id, person_id) values (58, 58);
insert into vacancy_candidate (vacancy_id, person_id) values (59, 59);
insert into vacancy_candidate (vacancy_id, person_id) values (60, 60);
insert into vacancy_candidate (vacancy_id, person_id) values (61, 61);
insert into vacancy_candidate (vacancy_id, person_id) values (62, 62);
insert into vacancy_candidate (vacancy_id, person_id) values (63, 63);
insert into vacancy_candidate (vacancy_id, person_id) values (64, 64);
insert into vacancy_candidate (vacancy_id, person_id) values (65, 65);
insert into vacancy_candidate (vacancy_id, person_id) values (66, 66);
insert into vacancy_candidate (vacancy_id, person_id) values (67, 67);
insert into vacancy_candidate (vacancy_id, person_id) values (68, 68);
insert into vacancy_candidate (vacancy_id, person_id) values (69, 69);
insert into vacancy_candidate (vacancy_id, person_id) values (70, 70);
insert into vacancy_candidate (vacancy_id, person_id) values (71, 71);
insert into vacancy_candidate (vacancy_id, person_id) values (72, 72);
insert into vacancy_candidate (vacancy_id, person_id) values (73, 73);
insert into vacancy_candidate (vacancy_id, person_id) values (74, 74);
insert into vacancy_candidate (vacancy_id, person_id) values (75, 75);
insert into vacancy_candidate (vacancy_id, person_id) values (76, 76);
insert into vacancy_candidate (vacancy_id, person_id) values (77, 77);
insert into vacancy_candidate (vacancy_id, person_id) values (78, 78);
insert into vacancy_candidate (vacancy_id, person_id) values (79, 79);
insert into vacancy_candidate (vacancy_id, person_id) values (80, 80);
insert into vacancy_candidate (vacancy_id, person_id) values (81, 81);
insert into vacancy_candidate (vacancy_id, person_id) values (82, 82);
insert into vacancy_candidate (vacancy_id, person_id) values (83, 83);
insert into vacancy_candidate (vacancy_id, person_id) values (84, 84);
insert into vacancy_candidate (vacancy_id, person_id) values (85, 85);
insert into vacancy_candidate (vacancy_id, person_id) values (86, 86);
insert into vacancy_candidate (vacancy_id, person_id) values (87, 87);
insert into vacancy_candidate (vacancy_id, person_id) values (88, 88);
insert into vacancy_candidate (vacancy_id, person_id) values (89, 89);
insert into vacancy_candidate (vacancy_id, person_id) values (90, 90);
insert into vacancy_candidate (vacancy_id, person_id) values (91, 91);
insert into vacancy_candidate (vacancy_id, person_id) values (92, 92);
insert into vacancy_candidate (vacancy_id, person_id) values (93, 93);
insert into vacancy_candidate (vacancy_id, person_id) values (94, 94);
insert into vacancy_candidate (vacancy_id, person_id) values (95, 95);
insert into vacancy_candidate (vacancy_id, person_id) values (96, 96);
insert into vacancy_candidate (vacancy_id, person_id) values (97, 97);
insert into vacancy_candidate (vacancy_id, person_id) values (98, 98);
insert into vacancy_candidate (vacancy_id, person_id) values (99, 99);
insert into vacancy_candidate (vacancy_id, person_id) values (100, 100);

-- end of vacancy_candidate table

-- certification_candidate table

insert into certification_candidate (certification_id, person_id) values (1, 1);
insert into certification_candidate (certification_id, person_id) values (2, 2);
insert into certification_candidate (certification_id, person_id) values (3, 3);
insert into certification_candidate (certification_id, person_id) values (4, 4);
insert into certification_candidate (certification_id, person_id) values (5, 5);
insert into certification_candidate (certification_id, person_id) values (6, 6);
insert into certification_candidate (certification_id, person_id) values (7, 7);
insert into certification_candidate (certification_id, person_id) values (8, 8);
insert into certification_candidate (certification_id, person_id) values (9, 9);
insert into certification_candidate (certification_id, person_id) values (10, 10);
insert into certification_candidate (certification_id, person_id) values (11, 11);
insert into certification_candidate (certification_id, person_id) values (12, 12);
insert into certification_candidate (certification_id, person_id) values (13, 13);
insert into certification_candidate (certification_id, person_id) values (14, 14);
insert into certification_candidate (certification_id, person_id) values (15, 15);
insert into certification_candidate (certification_id, person_id) values (16, 16);
insert into certification_candidate (certification_id, person_id) values (17, 17);
insert into certification_candidate (certification_id, person_id) values (18, 18);
insert into certification_candidate (certification_id, person_id) values (19, 19);
insert into certification_candidate (certification_id, person_id) values (20, 20);
insert into certification_candidate (certification_id, person_id) values (21, 21);
insert into certification_candidate (certification_id, person_id) values (22, 22);
insert into certification_candidate (certification_id, person_id) values (23, 23);
insert into certification_candidate (certification_id, person_id) values (24, 24);
insert into certification_candidate (certification_id, person_id) values (25, 25);
insert into certification_candidate (certification_id, person_id) values (26, 26);
insert into certification_candidate (certification_id, person_id) values (27, 27);
insert into certification_candidate (certification_id, person_id) values (28, 28);
insert into certification_candidate (certification_id, person_id) values (29, 29);
insert into certification_candidate (certification_id, person_id) values (30, 30);
insert into certification_candidate (certification_id, person_id) values (31, 31);
insert into certification_candidate (certification_id, person_id) values (32, 32);
insert into certification_candidate (certification_id, person_id) values (33, 33);
insert into certification_candidate (certification_id, person_id) values (34, 34);
insert into certification_candidate (certification_id, person_id) values (35, 35);
insert into certification_candidate (certification_id, person_id) values (36, 36);
insert into certification_candidate (certification_id, person_id) values (37, 37);
insert into certification_candidate (certification_id, person_id) values (38, 38);
insert into certification_candidate (certification_id, person_id) values (39, 39);
insert into certification_candidate (certification_id, person_id) values (40, 40);
insert into certification_candidate (certification_id, person_id) values (41, 41);
insert into certification_candidate (certification_id, person_id) values (42, 42);
insert into certification_candidate (certification_id, person_id) values (43, 43);
insert into certification_candidate (certification_id, person_id) values (44, 44);
insert into certification_candidate (certification_id, person_id) values (45, 45);
insert into certification_candidate (certification_id, person_id) values (46, 46);
insert into certification_candidate (certification_id, person_id) values (47, 47);
insert into certification_candidate (certification_id, person_id) values (48, 48);
insert into certification_candidate (certification_id, person_id) values (49, 49);
insert into certification_candidate (certification_id, person_id) values (50, 50);
insert into certification_candidate (certification_id, person_id) values (51, 51);
insert into certification_candidate (certification_id, person_id) values (52, 52);
insert into certification_candidate (certification_id, person_id) values (53, 53);
insert into certification_candidate (certification_id, person_id) values (54, 54);
insert into certification_candidate (certification_id, person_id) values (55, 55);
insert into certification_candidate (certification_id, person_id) values (56, 56);
insert into certification_candidate (certification_id, person_id) values (57, 57);
insert into certification_candidate (certification_id, person_id) values (58, 58);
insert into certification_candidate (certification_id, person_id) values (59, 59);
insert into certification_candidate (certification_id, person_id) values (60, 60);
insert into certification_candidate (certification_id, person_id) values (61, 61);
insert into certification_candidate (certification_id, person_id) values (62, 62);
insert into certification_candidate (certification_id, person_id) values (63, 63);
insert into certification_candidate (certification_id, person_id) values (64, 64);
insert into certification_candidate (certification_id, person_id) values (65, 65);
insert into certification_candidate (certification_id, person_id) values (66, 66);
insert into certification_candidate (certification_id, person_id) values (67, 67);
insert into certification_candidate (certification_id, person_id) values (68, 68);
insert into certification_candidate (certification_id, person_id) values (69, 69);
insert into certification_candidate (certification_id, person_id) values (70, 70);
insert into certification_candidate (certification_id, person_id) values (71, 71);
insert into certification_candidate (certification_id, person_id) values (72, 72);
insert into certification_candidate (certification_id, person_id) values (73, 73);
insert into certification_candidate (certification_id, person_id) values (74, 74);
insert into certification_candidate (certification_id, person_id) values (75, 75);
insert into certification_candidate (certification_id, person_id) values (76, 76);
insert into certification_candidate (certification_id, person_id) values (77, 77);
insert into certification_candidate (certification_id, person_id) values (78, 78);
insert into certification_candidate (certification_id, person_id) values (79, 79);
insert into certification_candidate (certification_id, person_id) values (80, 80);
insert into certification_candidate (certification_id, person_id) values (81, 81);
insert into certification_candidate (certification_id, person_id) values (82, 82);
insert into certification_candidate (certification_id, person_id) values (83, 83);
insert into certification_candidate (certification_id, person_id) values (84, 84);
insert into certification_candidate (certification_id, person_id) values (85, 85);
insert into certification_candidate (certification_id, person_id) values (86, 86);
insert into certification_candidate (certification_id, person_id) values (87, 87);
insert into certification_candidate (certification_id, person_id) values (88, 88);
insert into certification_candidate (certification_id, person_id) values (89, 89);
insert into certification_candidate (certification_id, person_id) values (90, 90);
insert into certification_candidate (certification_id, person_id) values (91, 91);
insert into certification_candidate (certification_id, person_id) values (92, 92);
insert into certification_candidate (certification_id, person_id) values (93, 93);
insert into certification_candidate (certification_id, person_id) values (94, 94);
insert into certification_candidate (certification_id, person_id) values (95, 95);
insert into certification_candidate (certification_id, person_id) values (96, 96);
insert into certification_candidate (certification_id, person_id) values (97, 97);
insert into certification_candidate (certification_id, person_id) values (98, 98);
insert into certification_candidate (certification_id, person_id) values (99, 99);
insert into certification_candidate (certification_id, person_id) values (100, 100);

-- end of certification_candidate table

```

# Queries

#### D1: Select all employees working in the Support team; ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue)
```
SELECT e.*
FROM employee e
JOIN person p ON e.person_id = p.person_id
JOIN team t ON t.person_id = p.person_id
WHERE t.team_name = 'Support';
```

#### D2: Select employees working in the company starting with the letter "A"; ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue)
```
SELECT e.*
FROM employee e
INNER JOIN company c ON c.company_id = c.company_id
WHERE c.company_name LIKE 'A%';
```

#### D3: Select from the table with candidates those whose experience is more than 5 years; 

#### RA:
```
{candidate}(years_of_experience > 5)[person_id, job_title, years_of_experience, category, description]
```
#### SQL:
```
SELECT * FROM candidate WHERE years_of_experience > 5;
```

#### D4: Select to me the vacancies for which a person with id 213 applied and for which no one else applied; ![](https://img.shields.io/badge/-C-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue)
```
SELECT v.*
FROM vacancy v
JOIN vacancy_candidate vc ON v.vacancy_id = vc.vacancy_id
WHERE vc.person_id = 21
AND NOT EXISTS (
    SELECT 1
    FROM vacancy_candidate vc2
    WHERE vc2.vacancy_id = v.vacancy_id
    AND vc2.person_id != vc.person_id
)
```
#### D5: Show me employee that worked or at least was part of every company in our database; ![](https://img.shields.io/badge/-D1-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G2-blue) ![](https://img.shields.io/badge/-I1-blue)

#### RA: 
```
{team*candidate}[person_id, company_id] ÷ company[company_id]
```
#### SQL:
```
select distinct person_id
from employee where person_id in(
    select person_id
    from team t where person_id in(
        select person_id from(
            select * from employee e where
            (select count(distinct company_id) from team t where t.person_id=e.person_id)
            =
            (select count(company_id) from company)
        ) result
    )
)
```
#### D6: Select company_name and company_description from the Company table where company_id is 5;

#### RA: 
```
{company}(company_id=5)[company_name, company_description]
```
#### SQL:
```
SELECT company_name, company_description 
FROM company 
WHERE company_id = 5;
```

#### D7: Select job_title and experience_level from the Vacancy table where salary is greater than 50000;
```
SELECT job_title, experience_level 
FROM vacancy 
WHERE salary > money(50000);
```

#### D8: Select job_title and salary from the Vacancy table where job_status is open;

#### RA:
```
{vacancy}(job_status='Open')[job_title, salary]
```
#### SQL:
```
SELECT job_title, salary
FROM vacancy
WHERE job_status = 'Open'
```

#### D9: Select all fields from the Vacancy and Candidate tables where the vacancy is related to a specific candidate with experience greater than 3 years and salary greater than 50000; ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue)
```
SELECT v.*, c.*
FROM vacancy v
INNER JOIN vacancy_candidate vc ON v.vacancy_id = vc.vacancy_id
INNER JOIN candidate c ON vc.person_id = c.person_id
WHERE c.years_of_experience > 3 AND v.salary > cast('50000' as money);
```

#### D10: Select job_title and work_mail from the Employee table where job_title is 'Manager';

#### RA:
```
{employee}(job_title='Project Manager')[job_title, work_mail]
```
#### SQL:
```
SELECT job_title, work_mail
FROM employee
WHERE job_title = 'Project Manager';
```

#### D11: This SQL query retrieves the maximum experience level required for any vacancy. ![](https://img.shields.io/badge/-I1-blue)
```
SELECT MAX(experience_level) FROM Vacancy;
```

#### D12: This SQL query counts the number of candidates with each certification type and group the results by the certification name. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-I1-blue) ![](https://img.shields.io/badge/-I2-blue)
```
SELECT certification.certification_name, COUNT(*) AS candidate_count
FROM certification_candidate
JOIN certification ON certification_candidate.certification_id = certification.certification_id
GROUP BY certification.certification_name;
```

#### D13: Selects all candidates with more than 5 years of experience who also have at least one certification. ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue) ![](https://img.shields.io/badge/-L-blue)
```
CREATE OR REPLACE VIEW high_experience_candidates AS
SELECT *
FROM candidate c
WHERE c.years_of_experience > 5
  AND EXISTS (
    SELECT 1
    FROM certification_candidate cc
    WHERE cc.person_id = c.person_id
  )
WITH CHECK OPTION;

SELECT * FROM high_experience_candidates;
```

#### D14: This SQL query retrieves all candidates with more years of experience than the average experience level of all candidates. ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-I1-blue)
```
SELECT * FROM Candidate WHERE years_of_experience > (SELECT AVG
    (years_of_experience) FROM Candidate);
```

#### D15: The query performs a full outer join between the "vacancy" and "team" tables on the team_id column, returning all columns from both tables. If there are no matching rows in one of the tables, the columns from that table will be null. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F5-blue)
```
SELECT vacancy.*, team.team_name FROM vacancy FULL JOIN team ON vacancy.team_id = team.team_id;
```

#### D16: List all jobs and their respective candidates with more than 5 years of experience using LEFT OUTER JOIN. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F4-blue)
```
SELECT * FROM Person LEFT OUTER JOIN Candidate ON Person.person_id = Candidate.person_id;
```

#### D17: Retrieve all combinations of certifications and candidates. ![](https://img.shields.io/badge/-F3-blue) ![](https://img.shields.io/badge/-I1-blue) ![](https://img.shields.io/badge/-I2-blue) 
```
SELECT v.vacancy_id, v.job_title, COUNT(vc.person_id) AS num_candidates
FROM vacancy v
CROSS JOIN vacancy_candidate vc
WHERE v.vacancy_id = vc.vacancy_id
GROUP BY v.vacancy_id, v.job_title;
```

#### D18: This query retrieves all the information about vacancies except for the vacancies that require candidates with less than five years of experience. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-H2-blue)
```
SELECT *
FROM vacancy
EXCEPT
SELECT v.*
FROM vacancy v
JOIN vacancy_candidate vc ON v.vacancy_id = vc.vacancy_id
JOIN candidate c ON c.person_id = vc.person_id
WHERE c.years_of_experience < 5;
```

#### D19: Retrieve the common information between teams and employees. ![](https://img.shields.io/badge/-H3-blue)

#### RA:
```
{person*employee}(person_id=person_id)[person_id]
```
#### SQL:
```
SELECT person_id FROM Person INTERSECT SELECT person_id FROM Employee;
```
#### D20: Retrieve all people who are also employees. ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue)

#### RA:
```
employee*>person
```
#### SQL:
```
SELECT * FROM person WHERE EXISTS(SELECT * FROM employee WHERE person.person_id = employee.person_id);
```

#### D21: Retrieve all employees with a job title of either 'Manager' or 'Assistant Manager'. ![](https://img.shields.io/badge/-H1-blue)

#### RA:
```
{employee}(job_title='Project Manager'∨job_title='Assistant Manager')[person_id, job_title, work_tel, work_mail]
```
#### SQL:
```
SELECT * FROM employee WHERE job_title = 'Project Manager' UNION SELECT * FROM employee WHERE job_title = 'Assistant Manager';
```

#### D22: Retrieve all the companies with their corresponding company names, descriptions, and country names. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F2-blue)

#### RA:
```
{company*address*city*country}[company_name, company_description, country_name]
```
#### SQL:
```
SELECT company_name, company_description, country_name 
FROM company 
NATURAL JOIN address 
NATURAL JOIN city 
NATURAL JOIN country;
```

#### D23: Delete all vacancies that have not been filled for more than 3 months. ![](https://img.shields.io/badge/-P-blue)
```
DELETE FROM vacancy
WHERE job_posting_date < CURRENT_DATE - INTERVAL '3 months'
AND NOT EXISTS (SELECT 1 FROM employee WHERE employee.job_title = vacancy.job_title);
```

#### D24: This query retrieves data on vacancies, candidates, people, employees, teams, companies, addresses, certifications, cities, and countries, joining multiple tables and filtering the results by experience level and the number of certifications, and finally ordering the results by job posting date. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-F4-blue) ![](https://img.shields.io/badge/-I1-blue) ![](https://img.shields.io/badge/-I2-blue) ![](https://img.shields.io/badge/-K-blue)
```
SELECT p.name, p.surname, v.job_title, v.vacancy_description, MAX(v.salary), v.experience_level, v.type_of_employment, c.certification_name, c.certifying_organization
FROM person p
JOIN employee e ON p.person_id = e.person_id
JOIN vacancy_candidate vc ON p.person_id = vc.person_id
JOIN vacancy v ON vc.vacancy_id = v.vacancy_id
LEFT JOIN certification_candidate cc ON p.person_id = cc.person_id
LEFT JOIN certification c ON cc.certification_id = c.certification_id
WHERE v.job_status = 'Open'
GROUP BY p.person_id, v.job_title, v.vacancy_description, v.experience_level, v.type_of_employment, c.certification_name, c.certifying_organization
HAVING COUNT(c.certification_id) >= 1
ORDER BY MAX(v.salary) DESC;
```

#### D25: Insert a new entry into the "certification_candidate" table, linking the certification with candidates who have at least 5 years of experience and hold the "CVUTFIT" certification. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-N-blue)
```
INSERT INTO certification_candidate (certification_id, person_id)
SELECT c.certification_id, cand.person_id
FROM certification c
INNER JOIN candidate cand ON c.certification_name = 'CVUTFIT'
WHERE cand.years_of_experience >= 5;
```

#### D26: This request is needed to update the number of years of experience for applicants who have a position of "Software Engineer" and who work for a company located in the city of "San Francisco" in the United States. It should increase the value of the "years_of_experience" column by 1 for all corresponding rows of the "Candidate" table. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue) ![](https://img.shields.io/badge/-O-blue)
```
UPDATE Candidate 
SET years_of_experience = years_of_experience + 1 
WHERE job_title = 'Software Engineer' 
AND EXISTS (
    SELECT 1 
    FROM Employee e 
    JOIN Team t ON e.job_title = t.team_name 
    JOIN Company c ON t.company_id = c.company_id 
    JOIN Address a ON c.company_id = a.address_id 
    JOIN City ct ON a.id_city = ct.id_city 
    JOIN Country cn ON cn.id_country = cn.id_country
    WHERE e.person_id = Candidate.person_id 
    AND cn.country_name = 'United States' 
    AND ct.city_name = 'San Francisco'
);
```

#### D27: This request is needed to select vacancies with a high salary for a position related to the Engineering department. It should output job IDs, job titles and salaries for all vacancies in the "vacancy" table, where the salary is greater than the average salary of all vacancies, and these vacancies are associated with the "Engineering" department. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G2-blue) ![](https://img.shields.io/badge/-I1-blue)
```
SELECT vacancy_id, job_title, salary
FROM (SELECT *
      FROM vacancy
      WHERE salary::numeric > (SELECT AVG(salary::numeric) FROM vacancy)) AS high_salary_jobs
INNER JOIN team ON high_salary_jobs.team_id = team.team_id
WHERE team.team_name = 'Engineering';
```

#### D28: This request is needed to select information about vacancies, including the number of qualified applicants who meet certain criteria. The request outputs the job ID, job title, salary, level of experience and the number of qualified applicants with more than 5 years of work experience who meet the requirements of the vacancy. ![](https://img.shields.io/badge/-G3-blue) ![](https://img.shields.io/badge/-I1-blue)
```
SELECT 
    v.vacancy_id, 
    v.job_title, 
    v.salary, 
    v.experience_level,
    (
        SELECT 
            COUNT(*) 
        FROM 
            candidate c 
        WHERE 
            c.job_title = v.job_title AND 
            c.years_of_experience >= 5
    ) AS num_qualified_candidates
FROM 
    vacancy v
WHERE 
    v.job_status = 'Open'
ORDER BY 
    v.salary DESC;
```

#### D29: This query is needed to select all team ids from the "team" table for which there is an entry in the "employee" table associated with the same person id "person_id". ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue) ![](https://img.shields.io/badge/-M-blue)
```
SELECT team_id FROM team t WHERE EXISTS (SELECT 1 FROM employee e WHERE e.person_id=t.person_id);
```

#### D30: These three queries perform the same task - select unique records from the "team" table for which there is a corresponding record in the "employee" table. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G4-blue) ![](https://img.shields.io/badge/-J-blue)
```
SELECT DISTINCT t.* FROM team t JOIN employee e ON e.person_id = t.person_id
ORDER BY team_name ASC, company_id DESC;
-- OR WITH EXISTS
SELECT DISTINCT t.* FROM team t WHERE EXISTS(SELECT * FROM employee e WHERE t.person_id = e.person_id)
ORDER BY team_name ASC, company_id DESC;
-- OR WITH IN
SELECT DISTINCT t.* FROM team t WHERE t.person_id IN (SELECT person_id FROM employee)
ORDER BY team_name ASC, company_id DESC;
```

#### D31: This query selects all candidates who have applied for a specific job (in this case, job ID 20) but have not applied for other jobs. ![](https://img.shields.io/badge/-C-blue) ![](https://img.shields.io/badge/-F1-blue) ![](https://img.shields.io/badge/-H2-blue)

#### RA:
```
{candidate*person*vacancy_candidate}(vacancy_id=10)[person_id, address_id, name, surname, tel,mail]
```
#### SQL:
```
select p.* from candidate c join person p on p.person_id = c.person_id join vacancy_candidate v on v.person_id = c.person_id
where v.vacancy_id=10
except
select p.* from candidate c join person p on p.person_id = c.person_id join vacancy_candidate v on v.person_id = c.person_id
where v.vacancy_id!=10
```

#### D32: Show me people's name, surname, email and phone number that are not in any team. We need to fire them, they are useless. ![](https://img.shields.io/badge/-B-blue) ![](https://img.shields.io/badge/-G1-blue)

#### RA:
```
team!*>person
```
#### SQL:
```
select p.*
from person p
where person_id not in (
    select person_id from team
);
```

#### D33: Checking a request from category D1. ![](https://img.shields.io/badge/-A-blue) ![](https://img.shields.io/badge/-D2-blue) ![](https://img.shields.io/badge/-F2-blue) ![](https://img.shields.io/badge/-F3-blue) ![](https://img.shields.io/badge/-G1-blue) ![](https://img.shields.io/badge/-G2-blue) ![](https://img.shields.io/badge/-H2-blue) ![](https://img.shields.io/badge/-I1-blue)
```
SELECT DISTINCT person_id
FROM (
    SELECT DISTINCT person_id,
                    company_id
    FROM TEAM
    NATURAL JOIN CANDIDATE
) R1
EXCEPT
SELECT DISTINCT person_id
FROM (
    SELECT DISTINCT *
    FROM (
        SELECT DISTINCT person_id
        FROM (
            SELECT DISTINCT person_id,
                            company_id
            FROM TEAM
            NATURAL JOIN CANDIDATE
        ) R1
    ) R2
    CROSS JOIN (
        SELECT DISTINCT company_id
        FROM COMPANY
    ) R3
    EXCEPT
    SELECT DISTINCT person_id,
                    company_id
    FROM TEAM
    NATURAL JOIN CANDIDATE
) R4

except

select distinct person_id
from employee where person_id in(
    select person_id
    from team t where person_id in(
        select person_id from(
            select * from employee e where
            (select count(distinct company_id) from team t where t.person_id=e.person_id)
            =
            (select count(company_id) from company)
        ) result
    )
)
```
|  Category  | Description | Covered by |
|---|---|---|
| A | A - Positive query on at least two joined tables | ![](https://img.shields.io/badge/-D1-blue) ![](https://img.shields.io/badge/-D2-blue) ![](https://img.shields.io/badge/-D9-blue) ![](https://img.shields.io/badge/-D12-blue) ![](https://img.shields.io/badge/-D15-blue) ![](https://img.shields.io/badge/-D16-blue) ![](https://img.shields.io/badge/-D18-blue) ![](https://img.shields.io/badge/-D22-blue) ![](https://img.shields.io/badge/-D24-blue) ![](https://img.shields.io/badge/-D25-blue) ![](https://img.shields.io/badge/-D26-blue) ![](https://img.shields.io/badge/-D27-blue) ![](https://img.shields.io/badge/-D30-blue) ![](https://img.shields.io/badge/-D33-blue) |
| AR | A (RA) - Positive query on at least two joined tables | ![](https://img.shields.io/badge/-D22-blue) |
| B | B - Negative query on at least two joined tables | ![](https://img.shields.io/badge/-D32-blue) |
| C | C - Select only those related to... | ![](https://img.shields.io/badge/-D4-blue) ![](https://img.shields.io/badge/-D31-blue) |
| CR | C (RA) - Select only those related to... | ![](https://img.shields.io/badge/-D31-blue) |
| D1 | D1 - Select all related to - universal quantification query | ![](https://img.shields.io/badge/-D5-blue) |
| D2 | D2 - Result check of D1 query | ![](https://img.shields.io/badge/-D33-blue) |
| F1 | F1 - JOIN ON | ![](https://img.shields.io/badge/-D1-blue) ![](https://img.shields.io/badge/-D2-blue) ![](https://img.shields.io/badge/-D4-blue) ![](https://img.shields.io/badge/-D9-blue) ![](https://img.shields.io/badge/-D12-blue) ![](https://img.shields.io/badge/-D18-blue) ![](https://img.shields.io/badge/-D24-blue) ![](https://img.shields.io/badge/-D25-blue) ![](https://img.shields.io/badge/-D26-blue) ![](https://img.shields.io/badge/-D27-blue) ![](https://img.shields.io/badge/-D30-blue) ![](https://img.shields.io/badge/-D31-blue) |
| F1R | F1 (RA) - JOIN ON | ![](https://img.shields.io/badge/-D31-blue) |
| F2 | F2 - NATURAL JOIN/JOIN USING | ![](https://img.shields.io/badge/-D22-blue) ![](https://img.shields.io/badge/-D33-blue) |
| F2R | F2 (RA) - NATURAL JOIN/JOIN USING | ![](https://img.shields.io/badge/-D22-blue) |
| F3 | F3 - CROSS JOIN | ![](https://img.shields.io/badge/-D17-blue) ![](https://img.shields.io/badge/-D33-blue) |
| F4 | F4 - LEFT/RIGHT OUTER JOIN | ![](https://img.shields.io/badge/-D16-blue) ![](https://img.shields.io/badge/-D24-blue) |
| F5 | F5 - FULL (OUTER) JOIN | ![](https://img.shields.io/badge/-D15-blue) |
| G1 | G1 - Nested query in WHERE clause | ![](https://img.shields.io/badge/-D4-blue) ![](https://img.shields.io/badge/-D5-blue) ![](https://img.shields.io/badge/-D13-blue) ![](https://img.shields.io/badge/-D14-blue) ![](https://img.shields.io/badge/-D20-blue) ![](https://img.shields.io/badge/-D26-blue) ![](https://img.shields.io/badge/-D27-blue) ![](https://img.shields.io/badge/-D29-blue) ![](https://img.shields.io/badge/-D30-blue) ![](https://img.shields.io/badge/-D32-blue) ![](https://img.shields.io/badge/-D33-blue) |
| G1R | G1 (RA) - Nested query in WHERE clause | ![](https://img.shields.io/badge/-D5-blue) ![](https://img.shields.io/badge/-D20-blue) ![](https://img.shields.io/badge/-D32-blue) |
| G2 | G2 - Nested query in FROM clause | ![](https://img.shields.io/badge/-D5-blue) ![](https://img.shields.io/badge/-D27-blue) ![](https://img.shields.io/badge/-D33-blue) |
| G2R | G2 (RA) - Nested query in FROM clause | ![](https://img.shields.io/badge/-D5-blue) |
| G3 | G3 - Nested query in SELECT clause | ![](https://img.shields.io/badge/-D28-blue) |
| G4 | G4 - Relative nested query (EXISTS/NOT EXISTS) | ![](https://img.shields.io/badge/-D4-blue) ![](https://img.shields.io/badge/-D13-blue) ![](https://img.shields.io/badge/-D20-blue) ![](https://img.shields.io/badge/-D26-blue) ![](https://img.shields.io/badge/-D29-blue) ![](https://img.shields.io/badge/-D30-blue) |
| G4R | G4 (RA) - Relative nested query (EXISTS/NOT EXISTS) | ![](https://img.shields.io/badge/-D20-blue) |
| H1 | H1 - Set unification - UNION | ![](https://img.shields.io/badge/-D21-blue) |
| H2 | H2 - Set difference - MINUS or EXCEPT | ![](https://img.shields.io/badge/-D18-blue) ![](https://img.shields.io/badge/-D31-blue) ![](https://img.shields.io/badge/-D33-blue) |
| H2R | H2 (RA) - Set difference - MINUS or EXCEPT | ![](https://img.shields.io/badge/-D31-blue) |
| H3 | H3 - Set intersection - INTERSECT | ![](https://img.shields.io/badge/-D19-blue) |
| I1 | I1 - Aggregate functions (count/sum/min/max/avg) | ![](https://img.shields.io/badge/-D5-blue) ![](https://img.shields.io/badge/-D11-blue) ![](https://img.shields.io/badge/-D12-blue) ![](https://img.shields.io/badge/-D14-blue) ![](https://img.shields.io/badge/-D17-blue) ![](https://img.shields.io/badge/-D24-blue) ![](https://img.shields.io/badge/-D27-blue) ![](https://img.shields.io/badge/-D28-blue) ![](https://img.shields.io/badge/-D33-blue) |
| I1R | I1 (RA) - Aggregate functions (count/sum/min/max/avg) | ![](https://img.shields.io/badge/-D5-blue) |
| I2 | 	I2 - Aggregate function over grouped rows - GROUP BY (HAVING) | ![](https://img.shields.io/badge/-D12-blue) ![](https://img.shields.io/badge/-D17-blue) ![](https://img.shields.io/badge/-D24-blue) |
| J | J - Same query in 3 different sql statements | ![](https://img.shields.io/badge/-D30-blue) |
| K | K - All clauses in one query - SELECT FROM WHERE GROUP BY HAVING ORDER BY | ![](https://img.shields.io/badge/-D24-blue) |
| L | L - View | ![](https://img.shields.io/badge/-D13-blue) |
| M | M - Query over a view | ![](https://img.shields.io/badge/-D29-blue) |
| N | N - INSERT, which insert a set of rows, which are the result of another subquery (an INSERT command which has VALUES clause replaced by a nested query. | ![](https://img.shields.io/badge/-D25-blue) |
| O | O - UPDATE with nested SELECT statement | ![](https://img.shields.io/badge/-D26-blue) |
| P | P - DELETE with nested SELECT statement | ![](https://img.shields.io/badge/-D23-blue) |


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
