# postgres specific
## installation
```sh
apt-get install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

apt-get update
apt-get install postgresql postgresql-contrib
```
manual installation, only use if needed, prefer docker if possible

## container
```sh
docker pull postgres:latest
```
pull docker image
```sh 
docker run -d -e POSTGRES_USER={role} -e POSTGRES_PASSWORD={pw} --name postdb -p 5454:5432  postgres
```
run new container 
```sh
docker pull dpage/pgadmin4
```
pull pgadmin if needed (SPOILER: you don't need it)
```sh
docker exec -it postgres psql -U myself
```
enter container, very important
```sh
netstat -ntlp
```
see wich ports are used (sometimes needed in debugging)
## postgres commands
```sh
sudo -i -u postgres
```
log into postgres account
```sh
sudo su - postgres
```
change user to postgres (postgres user is super admin of postgres db)
```sh
psql {dbname}
```
login to postgres console as current user, users database (default) or specified db
### inside postgres console
```
postgres-# 
```
``` 
\q
```
quit
```
\conninfo
```
see connection info
```
\l
```
list databases
```
\du
```
list users
```
\d
```
list all tables
```
\d {tablename}
```
info about columns in table
```
\createuser
```
create user, many options, look in [docs create user](https://www.postgresql.org/docs/9.1/app-createuser.html)
options ```--interactive``` for easy interactive set up ```--pwprompt``` to enable/ require password
```
\i /file/path/to.sql
```
executes sql-commands from file

# SQL commands
## creating users and dbs
```sql
CREATE USER {name}
```
create user
```sql  
CREATE DATABASE {name} OWNER {name}
```
create db
## inserting entries
```sql
INSERT INTO tableX (columnA, columnB, [...]) VALUES (valueA, valueB, [...]);
```
insert new entry in table 
### conflict handling
```sql
INSERT INTO [...] VALUES [...] ON CONFLICT (columnX, [...]) DO NOTHING;
```
do nothing if constraint is on columnX is violated
#### upsert
```sql
INSERT INTO tableX (columnY, [...]) VALUES ('valueY',[...]) ON CONFLICT (columnY) DO UPDATE SET columnY = EXCLUDED.columnY;
```
if entry conflicts perform update of conficting column instead of insertion of new entry, 
```EXCLUDED``` is keyword for a table of the conflicting entries
## selection (read)
```sql
SELECT {column} FROM {table}
```
select statement
```sql
SELECT DISTINCT {column} FROM {table}
```
no duplicates
```sql
SELECT {column} FROM {table} ORDER BY {column} ASC / DESC
```
sort
```sql
SELECT {column} FROM {table} WHERE {column} = {condition} 
```
### logical conditions
search for condition
```sql
SELECT {column} FROM {table} WHERE {column} = {condition1} AND ({column} = {condition2} OR {column} = {condition3})
```
featuring logic: condition1 and (condition2 or condition3) [docs 9.1 Logical Operators](https://www.postgresql.org/docs/9.1/functions-logical.html)
```sql
SELECT {column} FROM {table} WHERE {column} IN ({condition1},{condition2},{condition3})
```
search in array of conditions
```sql
= <= < > >= != <>
```
comparison operators, can also use english words to descibe logic like ```a BETWEEN x AND y``` [docs 9.2 Comparison Operators](https://www.postgresql.org/docs/9.1/functions-comparison.html) 
```sql
SELECT {column} FROM {table} WHERE {column} BETWEEN {condition 1} AND {condition2}
SELECT {column} FROM {table} WHERE {column} > {condition1} AND {column} < {condition2}
```
range search synomyms
### apply math on database level
```
+ - * / ^ ROUND()
```
mathmatical operators including more complex math like ```sin(x)``` [docs 9.3 mathematical functions](https://www.postgresql.org/docs/9.1/functions-math.html)
```sql
max() min() avg() count()
```
aggregate functions can do more complex operation [9.20 Aggregate Functions](https://www.postgresql.org/docs/9.5/functions-aggregate.html)

### string comparison
```sql
SELECT {column} FROM {table} WHERE {column} LIKE {pattern}
```
search substing case sensitive
```sql
SELECT {column} FROM {table} WHERE {column} ILIKE {pattern}
```
search substring case insensitive
```
%
```
variable length wildcard
```
_
```
single character wildcard
### return subset of entries
```sql
SELECT {column) FROM {table} LIMIT {number}
SELECT {column} FROM {table} FETCH FRIST {number} ROW ONLY
```
limit amount of returned rows, ```LIMIT``` is not SQL standart, correct to use ```FETCH```
```sql
SELECT {column} FROM {table} OFFSET {number}
```
only return values after certain entry
```sql
SELECT name FROM employee ORDER BY salary OFFSET 1 LIMIT 1;
```
example: get name of employee with second highest salary in company
### count identical entries
```sql
SELECT {column}, COUNT({column}||*) FROM {table} GROUP BY {column}
```
count number of identical column entries
```sql
SELECT {column}, COUNT({column ||*} FROM {table} GROUP BY {column} HAVING {condition}
```
count number with condition (applied to the GROUP not the single ROW)
### returning default if missing
```sql
SELECT COALESCE(column, "default")
SELECT COALESCE(column, "1","2","3","4", ...etc)
```
Coalesce can be used to give default value if no value is found, return either colum entry or "default"
several defaults are possible
```sql
NULLIF(value1,value2)
```
returns a null value is value1 equals value2, otherwise return value1
```sql
SELECT COALESCE(10 / NULLIF(0,0),0);
```
return 0 instead of error if dividing by zero
more info on ```COALESCE```and ```NULLIF``` in [doc 9.17 Conditional Expressions](https://www.postgresql.org/docs/9.5/functions-conditional.html)
## time
```sql
NOW()
``` 
function creates current timestamp ```NOW()::DATE``` show only date ```NOW()::TIME``` show only time
more in [docs 8.5 Date/Time Types](https://www.postgresql.org/docs/9.5/datatype-datetime.html)
```sql
SELECT NOW() +/- INTERVAL ' 1 YEAR 20 MONTHS 1 DAY 30 HOURS 1 MINUTE 21 SECONDS';
```
add or subtract time interval from date
```sql
SELECT EXTRACT(CENTURY/YEAR/DAY/MINUTE FROM NOW());
```
extract only specific information from timestamp
```sql
AGE(timestamp)
AGE(timestamp,timestamp)
```
shows either difference timestamp to now() or difference between two timestamps, more information in [docs 9.9 Date/Time Functions and Operators](https://www.postgresql.org/docs/9.5/functions-datetime.html)
## delete entries
```sql
DELETE FROM tableX WHERE columnY = $1;
```
delete entry from tableX where columnY is $1
## UPDATE entries
```sql
UPDATE tableX SET columnY = 'newValue' WHERE columnZ = $1;
UPDATE tableX SET columnY = 'firstValue', columnZ = 'secondValue' [...] WHERE columnA = $1;
```
update single or multiple values

## modify table
```sql
ALTER TABLE ...
````
```sql
ALTER TABLE tableX ADD PRIMARY KEY (columnY);
```
make column primary key
```sql
ALTER TABLE tableX ADD CONSTRAINT constraintNameY UNIQUE (columnZ);
ALTER TABLE tableX ADD UNIQUE (columnY);
```
adds unique constraint to columnY in tableX, second lets postgres pick the constraint name
```sql
ALTER TABLE tableX ADD CONSTRAINT constraintNameY CHECK (columnY = 'something' OR columnY = 'other'); 
```
## foreign keys
```sql
columnX BIGINT REFERENCES tableY(columnZ)
```
regular foreign key 
```sql
columnX BIGINT REFERENCES tableY(columnZ) NOT NULL
```
must have relation
```sql
columnX BIGINT REFERENCES tableY(columnZ),
UNIQUE(columnX)
```
unique "owner"
