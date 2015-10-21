# Introduction to databases

## Olav Vahtras

### Software Carpentry Workshop Potsdam 2015-09-25

---

layout: false

## Data Storage Solutions


### Text files

*  Simple
*  Work well with version control
*  Need to write own tools for analysis

### Spread sheets

* Good for simple analysis
* Not so for large complex data sets

### Databases

* Powerful tools for search and analysis
* Another language to learn
* Different database engines (Oracle, Microsoft, Mysql...) with incompatible commands
* We use SQLite http://www.sqlite.org

---

## Setup for exercise

* Install `sqlite3`
```bash
$ apt-get install sqlite3
```

* Get database
```bash
$ wget http://files.software-carpentry.org/survey.db
```

* Test setup
```bash
$ sqlite3 survey.db
SQLite version 3.8.7.4 2014-12-09 01:34:36
Enter ".help" for usage hints.
sqlite> 
```


---

## sqlite and SQL

Two sets of commands:

### sqlite

Commands beginning with a dot are for this database implementation

* Get help
```bash
sqlite> .help
.backup ?DB? FILE      Backup DB (default "main") to FILE
.bail on|off           Stop after hitting an error.  Default OFF
.clone NEWDB           Clone data into NEWDB from the existing database
...
```
* Initial configuration
```bash
sqlite> .header on
sqlite> .mode columns
```
* Quit, back to shell
```bash
sqlite> .quit
$
```

---

* What tables are defined?
```bash
sqlite> .tables
Person   Site     Survey   Visited
```

* How are the tables defined?
```sql
sqlite> .schema
CREATE TABLE Person(
	ident    text,
	personal text,
	family	 text
);
CREATE TABLE Site(
	name text,
	lat  real,
	long real
);
CREATE TABLE Visited(
	ident integer,
	site  text,
	dated text
);
CREATE TABLE Survey(
	taken   integer,
	person  text,
	quant   text,
	reading real
);
```
*Example of SQL commands: Sequential Query Language*

---

## This example

* *Person*: who took readings
* *Site*: locations readings where taken
* *Survey*: actual readings
* *Visited*: when readings where taken
---

### Databases

* Arranged as tables
* Table columns = fields
* Rows = records of data
* Created and manipulated with SQL commands

#### Examples

* Select everything from a table

```sql
sqlite> SELECT * FROM Person;
ident       personal    family    
----------  ----------  ----------
dyer        William     Dyer      
pb          Frank       Pabodie   
lake        Anderson    Lake      
roe         Valentina   Roerich   
danforth    Frank       Danforth  
```
---
* Select a few columns

```sql
sqlite> SELECT personal, family  FROM Person;
personal    family    
----------  ----------
William     Dyer      
Frank       Pabodie   
Anderson    Lake      
Valentina   Roerich   
Frank       Danforth  
```
```sql
sqlite> SELECT personal FROM Person;
personal    family    
----------  ----------
William     Dyer      
Frank       Pabodie   
Anderson    Lake      
Valentina   Roerich   
Frank       Danforth  
```

---

## Removing duplicates

* What quantities where measured?
* Use `DISTINCT` to remove duplicates

```sql
sqlite> SELECT DISTINCT quant FROM Survey;
quant     
----------
rad       
sal       
temp      
```

---

## Sorting by a certain field

* List persons sorted by last name
* Use `ORDER BY`

```sql
sqlite> SELECT * from Person ORDER BY family;
ident       personal    family    
----------  ----------  ----------
danforth    Frank       Danforth  
dyer        William     Dyer      
lake        Anderson    Lake      
pb          Frank       Pabodie   
roe         Valentina   Roerich   
```

--

### Exercise

1. Write a query that selects distinct dates from the Visited table.

---

## Filtering

Example: which measuremts (ident) where made at a certain cite (DR-1)

* Use `WHERE` to fileter

```sql
sqlite> SELECT ident FROM Visited WHERE site='DR-1';
ident     
----------
619       
622       
844       
```
* Selections are made on another field than displayed

<center>
<img src="fig/sql-filter.svg" alt="SQL Filtering in Action" width="400"/>
</center>

---

* Combine several filters with logic: `OR`, `AND`

```sql
sqlite> SELECT * FROM Visited WHERE (site='DR-1' OR site='DR-3');
ident       site        dated     
----------  ----------  ----------
619         DR-1        1927-02-08
622         DR-1        1927-02-10
734         DR-3        1939-01-07
735         DR-3        1930-01-12
751         DR-3        1930-02-26
752         DR-3                  
844         DR-1        1932-03-22
```

* Using wildcards: `LIKE`

```sql
sqlite> SELECT * FROM Visited WHERE site LIKE 'DR-%';
ident       site        dated     
----------  ----------  ----------
619         DR-1        1927-02-08
622         DR-1        1927-02-10
734         DR-3        1939-01-07
735         DR-3        1930-01-12
751         DR-3        1930-02-26
752         DR-3                  
844         DR-1        1932-03-22
```
---

#### Exercise
Write a query that selects all records from Survey with salinity values outside the range [0, 1].

--

#### Solution

```sql
sqlite> SELECT * FROM Survey WHERE (quant='sal' AND (reading<0 OR reading>1));
taken       person      quant       reading   
----------  ----------  ----------  ----------
752         roe         sal         41.6      
837         roe         sal         22.5      
```

---

## Calculating new values

Consider calculating a new value from the data

* Example: add a column with temperature readings in Fahrenheit

--

```sql
sqlite> SELECT *, 32+1.8*reading 'reading(F)' FROM Survey WHERE quant='temp';
taken       person      quant       reading     reading(F)
----------  ----------  ----------  ----------  ----------
734         pb          temp        -21.5       -6.7      
735                     temp        -26.0       -14.8     
751         pb          temp        -18.5       -1.3      
752         lake        temp        -16.0       3.2       
```

---

## Missing data

Consider the Visted table
```sql
sqlite> SELECT * FROM Visited;
ident       site        dated     
----------  ----------  ----------
619         DR-1        1927-02-08
622         DR-1        1927-02-10
734         DR-3        1939-01-07
735         DR-3        1930-01-12
751         DR-3        1930-02-26
752         DR-3                  
837         MSK-4       1932-01-14
844         DR-1        1932-03-22
```

* The date is missing by one of its entriese
* A missing entry is referenced with the special variable  `NULL`

```
sqlite> SELECT * FROM Visited WHERE dated IS NULL;
ident       site        dated     
----------  ----------  ----------
752         DR-3                  
```

---

### Aggregation of data

* Combines several values of one type into one
* `MIN`, `MAX`, `AVG`, `COUNT`, `SUM`

#### Examples

* Latest reading

```sql
sqlite> SELECT *, MAX(dated) FROM Visited;
ident       site        dated       MAX(dated)
----------  ----------  ----------  ----------
734         DR-3        1939-01-07  1939-01-07

* Number of readings
qlite> SELECT COUNT(*) FROM Survey;
COUNT(*)  
----------
21        
```

---

* Averages can be done on subsets with `GROUP BY`: e.g. average reading
per person for each quantity

```sql
sqlite> SELECT taken, person, quant, AVG(reading) 
   ...> FROM Survey 
   ...> GROUP BY person, quant;

person      quant       AVG(reading)
----------  ----------  ------------
            sal         0.06        
            temp        -26.0       
dyer        rad         8.81        
dyer        sal         0.11        
lake        rad         1.825       
lake        sal         0.1125      
lake        temp        -16.0       
pb          rad         6.66        
pb          temp        -20.0       
roe         rad         11.25       
roe         sal         32.05       
```

---


### Summary

* SQL allows you to manipulate datasets at a level not possible with spreadsheets.
* SQL is a large programming language, a few selected commands gets you far.
`SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`
* sqlite is one of several database enginges (Microsoft, Oracle, Mysql...)
and interacts with a file rather than a server.
