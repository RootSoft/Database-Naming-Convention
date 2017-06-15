# Database Naming Convention

Most of the recommendations below should be equally valid for other relational databases such as MySQL, Oracle, or Microsoft SQL Server.
A lot of them will also apply to NoSQL databases, though not everything. For example, the suggestion below to use full english words goes against the [recommended approach](https://docs.mongodb.com/manual/faq/fundamentals/#how-do-i-optimize-storage-use-for-small-documents "recommended approach") for naming fields in MongoDB. When in doubt, find a guide for your specific database type.

**Warning! This is a fairly opinionated post and I welcome feedback from people suggesting alternatives.**

## Importance of naming conventions
---
### Names are long lived

Data structures are meant to last much longer than application code. Anyone that has worked on a long running system can attest to that.
Well defined data structures and table layouts will outlive any application code. It's not uncommon to see an application completely rewritten without any changes done to its database schema.

### Names are contracts

Database objects are referenced by their names, thus object names are part of the contract for an object. In a way you can consider your database table and column names to be the API to your data model.
Once they are set, changing them may break dependent applications. This is all the more reason to name things properly before the first use.

### Developer context switching

Having consistent naming conventions across your data model means that developers will need to spend less time looking up the names of tables, views, and columns. Writing and debugging SQL is easier when you know that ```person_id``` must be a foreign key to the ```id``` field of the ```person``` table.

## Naming conventions
---

**Avoid quotes.** If you have to quote an identifier then you should rename it. Quoted identifiers are a serious pain. Writing SQL by hand using quoted identifiers is frustrating and writing dynamic SQL that involves quoted identifiers is even more frustrating.
This also means that you should never include whitespace in identifier names.

Ex: **Avoid** using names like ```"FirstName"``` or ```"All Employees"```.

**Lowercase.** Identifiers should be written entirely in lower case. This includes tables, views, column, and everything else too. Mixed case identifier names means that every usage of the identifier will need to be quoted in double quotes (which we already said are not allowed).

Ex: Use ```first_name```, **not** ```"First_Name"```.

**Data types are not names.** Database object names, particularly column names, should be a noun describing the field or object. Avoid using words that are just data types such as ```text``` or ```timestamp```. The latter is particularly bad as it provides zero context.

**Underscores separate words.** Object name that are comprised of multiple words should be separated by underscores (ie. [snake case](https://en.wikipedia.org/wiki/Snake_case "snake case")).

Ex: Use ```word_count``` or ```team_member_id```, **not** ```wordcount``` or ```wordCount```.

**Full words, not abbreviations.** Object names should be full English words. In general avoid abbreviations, especially if they're just the type that removes vowels. Most SQL databases support at least *30-character names* which should be more than enough for a couple English words. PostgreSQL supports up to *63-character* for identifiers.

Ex: Use ```middle_name```, **not** ```mid_nm```.

**Use common abbreviations.** For a few long words the abbreviation is both more common than the word itself. *"Internationalization"* and *"localization"* are the two that come up most often as ```i18n``` and ```l10n``` respectively. In these cases use the abbreviation.
If you're in doubt, use the full English word. It should be obvious where the abbreviation makes sense.

**Avoid reserved words.** Avoid using any word that is considered a reserved word in the database that you are using. There aren't that many of them so it's not too much effort to pick a different word. Depending on the context, reserved words may require quoting. This means sometimes you'll write ```"user"``` and sometimes just ```user```.

Another benefit of avoiding reserved words is that less-than-intelligent editor syntax highlighting won't erroneously highlight them.

Ex: Avoid using words like ```user```, ```lock```, or ```table```.

Here are the list of reserved words for [PostgreSQL](https://www.postgresql.org/docs/9.3/static/sql-keywords-appendix.html "PostgreSQL"), [MySQL](https://dev.mysql.com/doc/refman/5.7/en/reserved-words.html "MySQL"), [Oracle](http://docs.oracle.com/database/121/SQLRF/ap_keywd.htm#SQLRF022 "Oracle"), and [MSSQL](https://technet.microsoft.com/en-us/library/ms189822.aspx "MSSQL").

## Singular relations
---
**Tables, views, and other relations that hold data should have singular names, not plural.** This means our tables and views would be named ```team```, not ```teams```.
Rather than going into the relational algebra explanation of why this is correct I'll give a few practical reasons.

**They're Consistent.** It's possible to have a relation that holds a single row. Is it still plural?

**They're unambiguous.** Using only singular names means you don't need to determine how to pluralize nouns.

Ex: Does a "Person" object go into a "Persons" relation or a "People" one? How about an "Octopus" object? Octopuses? Octopi? Octopodes?

**Straightforward 4GL Translation.** Singular names allow you to directly translate from 4GL objects to database relations. You may need to remove some underscores and switch to camel case but the name translation will always be straight forward.

Ex: ```team_member``` unambigously becomes the class ```TeamMember``` in Java or the variable ```team_member``` in Python.

## Key Fields
---
### Primary Keys

Single column primary key fields should be named ```id```. It's short, simple, and unambiguous. This means that when you're writing SQL you don't have to remember the names of the fields to join on.

```sql
CREATE TABLE person (
  id            bigint PRIMARY KEY,
  full_name     text NOT NULL,
  birth_date    date NOT NULL);
```

Some guides suggest prefixing the table name in the primary key field name, ie. ```person_id``` vs ```id```. The extra prefix is redundant. All field names in non-trivial SQL statements (i.e. those with more than one table) should be explicitly qualified and prefixing as a form of namespacing field names is a bad idea.

### Foreign Keys

Foreign key fields should be a combination of the name of the referenced table and the name of the referenced fields. For single column foreign keys (by far the most common case) this will be something like ```foo_id```.

```sql
CREATE TABLE team_member (
  team_id       bigint NOT NULL REFERENCES team(id),
  person_id     bigint NOT NULL REFERENCES person(id),
  CONSTRAINT team_member_pkey PRIMARY KEY (team_id, person_id));
```

## Explicit Naming
---

Some database commands that create database objects do not require you specify a name. An object name will be generated either randomly *(ex: fk239nxvknvsdvi)* or via a formula *(ex: foobar_ix_1)*. Unless you know exactly how a name will be generated and you are happy with it, you should be explicitly specifying names.

This also includes names generated by ORMs. Many ORMs default to creating indexes and constraints with long gibberish generated names. The couple minutes of time savings in the short run are not worth the head ache in remembering what *(ex: fk239nxvknvsdvi)*  refers to in the long run.

### Indexes

Indexes should be explicitly named and include both the table name and the column name(s) indexed. Including the column names make it much easier to read through SQL explain plans. If an index is named ```foobar_ix1``` then you would need to look up what columns that index covers to understand if it is being used correctly.

```sql
CREATE TABLE person (
  id          bigserial PRIMARY KEY,
  email       text NOT NULL,
  first_name  text NOT NULL,
  last_name   text NOT NULL,
  CONSTRAINT person_ck_email_lower_case CHECK (email = LOWER(email)));

CREATE INDEX person_ix_first_name_last_name ON person (first_name, last_name);
```

Explain plans will now be easy to understand. We can clearly see that the index on first name and last name, ie. ```person_ix_first_name_last_name```, is being used:

```sql
=# EXPLAIN SELECT * FROM person WHERE first_name = 'alice' AND last_name = 'smith';

                                          QUERY PLAN                                          
----------------------------------------------------------------------------------------------
 Index Scan using person_ix_first_name_last_name on person  (cost=0.15..8.17 rows=1 width=72)
   Index Cond: ((first_name = 'alice'::text) AND (last_name = 'smith'::text))
(2 rows)
```

### Constraints

Similar to indexes, constraints should given descriptive names. This is especially true for check constraints. It's much easier to diagnose an errant insert if the check constraint that was violated lets you know the cause.

```sql
CREATE TABLE team (
  id          bigserial PRIMARY KEY,
  name        text NOT NULL);

CREATE TABLE team_member (
  team_id     bigint REFERENCES team(id),
  person_id   bigint REFERENCES person(id),
  CONSTRAINT team_member_pkey PRIMARY KEY (team_id, person_id));
```

Notice how PostgreSQL does a good job of giving descriptive names to the foreign key constraints.

```sql
=# \d team_member
   Table "public.team_member"
  Column   |  Type  | Modifiers 
-----------+--------+-----------
 team_id   | bigint | not null
 person_id | bigint | not null
Indexes:
    "team_member_pkey" PRIMARY KEY, btree (team_id, person_id)
Foreign-key constraints:
    "team_member_person_id_fkey" FOREIGN KEY (person_id) REFERENCES person(id)
    "team_member_team_id_fkey" FOREIGN KEY (team_id) REFERENCES team(id)
```

If we try inserting a row that violates one of these constraints we immediately know the cause just based on the constraint name:

```sql
INSERT INTO team_member(team_id, person_id) VALUES (1234, 5678);
ERROR:  insert or update on table "team_member" violates foreign key constraint "team_member_team_id_fkey"
DETAIL:  Key (team_id)=(1234) is not present in table "team".
```

Similarly, if we try inserting an email address that is not lower case into the person table created above, we'll get a constraint violation error that tells us exactly what is wrong:

```sql
-- This insert will work:
INSERT INTO person (email, first_name, last_name) VALUES ('alice@example.com', 'Alice', 'Anderson');
INSERT 0 1

-- This insert will not work:
INSERT INTO person (email, first_name, last_name) VALUES ('bob@EXAMPLE.com', 'Bob', 'Barker');
ERROR:  new row for relation "person" violates check constraint "person_ck_email_lower_case"
DETAIL:  Failing row contains (2, bob@EXAMPLE.com, Bob, Barker).
```

---
### Final thoughts

If you're starting a new project then I suggest you follow the conventions outlined here. If you're working on an existing project then you need to be a bit more careful with any new objects you create.

The only thing worse than bad naming conventions is multiple naming conventions. If your existing project already has a standard approach to naming its database objects **then keep using it.**


All credits belong to Sehrope Sarkuni from [launchbylunch.com](https://launchbylunch.com "launchbylunch.com").

  

