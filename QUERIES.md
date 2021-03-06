The top level name is called a measurement. These names can contain any characters. Then there are field names, field values, tag keys and tag values, which can also contain any characters. Because of this, anywhere a measurement name, field name, field value, tag name, or tag value appears should be able to get wrapped in double quotes to deal with special characters.

# Databases & retention policies

```sql
-- create a database
CREATE DATABASE <name>

-- create a retention policy
CREATE RETENTION POLICY <rp-name> ON <db-name> DURATION <duration> REPLICATION <n> [DEFAULT]

-- alter retention policy
ALTER RETENTION POLICY <rp-name> ON <db-name> (DURATION <duration> | REPLICATION <n> | DEFAULT)+

-- drop a database
DROP DATABASE <name>

-- drop a retention policy
DROP RETENTION POLICY <rp-name> ON <db.name>
```

# Users and permissions

```sql
-- create user
CREATE USER <name> WITH PASSWORD <password>

-- grant privilege on a database
GRANT <privilege> ON <db> TO <user>

-- grant cluster admin privileges
GRANT ALL [PRIVILEGES] TO <user>

-- revoke privilege
REVOKE <privilege> ON <db> FROM <user>

-- revoke all privileges for a DB
REVOKE ALL [PRIVILEGES] ON <db> FROM <user>

-- revoke all of user's privileges (all DBs and/or cluster admin)
REVOKE ALL [PRIVILEGES] FROM <user>

-- delete a user
DROP USER <name>
```
where `<privilege> := READ | WRITE | All [PRIVILEGES]`.

# Select

```sql
SELECT mean(value) from cpu WHERE host = 'serverA' AND time > now() - 4h GROUP BY time(5m)

SELECT mean(value) from cpu WHERE time > now() - 4h GROUP BY time(5m), region
```

## Group By

# Delete

# Series

## Destroy

```sql
DROP MEASUREMENT <name>
DROP MEASUREMENT cpu WHERE region = 'uswest'
```

## Show

Show series queries are for pulling out individual series from measurement names and tag data. They're useful for discovery.

```sql
-- show all databases
SHOW DATABASES

-- show measurement names
SHOW MEASUREMENTS
SHOW MEASUREMENTS LIMIT 15
SHOW MEASUREMENTS LIMIT 10 OFFSET 40
SHOW MEASUREMENTS WHERE service = 'redis'
-- LIMIT and OFFSET can be applied to any of the SHOW type queries

-- show all series across all measurements/tagsets
SHOW SERIES

-- get a show of all series for any measurements where tag key region = tak value 'uswest'
SHOW SERIES WHERE region = 'uswest'

SHOW SERIES FROM cpu_load WHERE region = 'uswest' LIMIT 10

-- returns the 100 - 109 rows in the result. In the case of SHOW SERIES, which returns 
-- series split into measurements. Each series counts as a row. So you could see only a 
-- single measurement returned, but 10 series within it.
SHOW SERIES FROM cpu_load WHERE region = 'uswest' LIMIT 10 OFFSET 100

-- show all retention policies on a database
SHOW RETENTION POLICIES mydb

-- get a show of all tag keys across all measurements
SHOW TAG KEYS

-- show all the tag keys for a given measurement
SHOW TAG KEYS FROM cpu
SHOW TAG KEYS FROM temperature, wind_speed

-- show all the tag values. note that a single WHERE TAG KEY = '...' clause is required
SHOW TAG VALUES WITH TAG KEY = 'region'
SHOW TAG VALUES FROM cpu WHERE region = 'uswest' WITH TAG KEY = 'host'

-- and you can do stuff against fields
SHOW FIELD KEYS FROM cpu

-- but you can't do this
SHOW FIELD VALUES
-- we don't index field values, so this query should be invalid.

-- show all users
SHOW USERS
```

Note that `FROM` and `WHERE` are optional clauses in most of the show series queries.

And the show series output looks like this:

```json
[
    {
        "name": "cpu",
        "columns": ["id", "region", "host"],
        "values": [
            1, "uswest", "servera",
            2, "uswest", "serverb"
        ]
    },
    {
        "name": "reponse_time",
        "columns": ["id", "application", "host"],
        "values": [
            3, "myRailsApp", "servera"
        ]
    }
]
```

# Continuous Queries

Continous queries are going to be inspired by MySQL `TRIGGER` syntax:

http://dev.mysql.com/doc/refman/5.0/en/trigger-syntax.html

Instead of having automatically-assigned ids, named continuous queries allows for some level of duplication prevention,
particularly in the case where creation is scripted.

## Create

    CREATE CONTINUOUS QUERY <name> AS SELECT ... FROM ...

## Destroy

    DROP CONTINUOUS QUERY <name>

## List

    SHOW CONTINUOUS QUERIES
