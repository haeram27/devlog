# Postgresql: useful admin commands

## check alive is postgresql

```sql
SELECT 1 AS is_alive
```

## set permission login of roles

reference:  
<https://www.postgresql.org/docs/current/sql-alterrole.html>

syntax:

```sql
ALTER ROLE <role-name> [LOGIN|NOLOGIN];
```

example:

```sql
ALTER ROLE postgres NOLOGIN;
```

how to check status of login permit of roles:

```sql
SELECT rolcanlogin FROM pg_roles WHERE rolname = 'postgres';
```

## psql

### psql syntax

```bash
PGPASSWD=<password> psql -h <ip> -p <port> -U <user> -d <db> -c "<sql>"
```

```bash
sudo postgres -i env PGPASSWORD=<password> psql -h <ip> -p <port> -U <user> -d <db> -c "<sql>"
```

```bash
docker exec -e PGPASSWD=<password> <psql-container> psql -h <ip> -p <port> -U <user> -d <db> -c "<sql>"
```

If `-h <ip>` option is omitted, postgres recognizes a request comes from localhost.
Please consider pg_hba.conf, this configuration defines rules which user and which condition can allow to access to postgres.

### sanitize psql response

```bash
psql -d <db-name> -p <port> -At -c "select 1 as is_alive limit 1;"
```

|options|meaning|
|:---|:---|
|-t|(tuples‑only) remove header, footer, row number|
|-A|(unaligned) remove spaces for alignment between attribute|
|-c|execute following single string as SQL|

LIMIT 1 (optional) remaining only 1 of tuple
