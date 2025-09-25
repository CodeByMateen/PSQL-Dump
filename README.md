# PostgreSQL Database Dump and Restore Guide

This guide provides clear steps for creating a PostgreSQL dump from a
remote server and restoring it into a local PostgreSQL database. It is
written for Windows users but the same commands work on Linux/macOS with
minor adjustments.

------------------------------------------------------------------------

## 1. Prerequisites

-   PostgreSQL installed on your local machine (with `psql` and
    `pg_dump` available in PATH).
-   Database credentials (username, password, host, port, database
    name).
-   Command line access (Command Prompt, PowerShell, or terminal).

------------------------------------------------------------------------

## 2. Create a Dump from Remote Database

Run the following command to dump a remote database into a `.sql` file:

``` bash
pg_dump "postgres://<username>:<password>@<host>:<port>/<database>" > prod_dump.sql
```

### Example:

``` bash
pg_dump "postgres://root:password123@182.184.67.82:5466/postgres" > prod_dump.sql
```

-   `pg_dump` connects to the remote PostgreSQL server.\
-   The dump is saved in the file `prod_dump.sql`.\
-   Progress is not shown, but once the prompt returns, the dump is
    complete.

------------------------------------------------------------------------

## 3. Restore Dump into Local Database

### Step 1: Drop and Recreate the Local Database

First, connect to your PostgreSQL instance:

``` bash
psql -U postgres
```

Inside `psql`, drop and recreate the target database:

``` sql
DROP DATABASE IF EXISTS stickball;
CREATE DATABASE stickball;
```

### Step 2: Import the Dump

Exit `psql` (type `\q`) and run this from the terminal:

``` bash
psql -h localhost -U postgres -d stickball -f prod_dump.sql
```

-   `-h localhost`: Connects to local server.\
-   `-U postgres`: Uses the `postgres` user (change if needed).\
-   `-d stickball`: Target local database.\
-   `-f prod_dump.sql`: Executes SQL commands from the dump file.

------------------------------------------------------------------------

## 4. Verify the Data

After the restore, connect to your local database and verify:

``` bash
psql -U postgres -d stickball
```

Inside `psql`:

``` sql
\dt            -- List tables
SELECT * FROM users LIMIT 10;
```

Or use a GUI client like Beekeeper Studio, DBeaver, or pgAdmin to browse
the data.

------------------------------------------------------------------------

## 5. Notes

-   Ensure your local PostgreSQL version is compatible with the remote
    version to avoid restore issues.\
-   If the dump file is very large, consider compressing it using
    `gzip`:

``` bash
pg_dump "postgres://..." | gzip > prod_dump.sql.gz
gunzip prod_dump.sql.gz
```

-   For schema-only or data-only dumps, use:

``` bash
pg_dump --schema-only "connection_string" > schema.sql
pg_dump --data-only "connection_string" > data.sql
```

------------------------------------------------------------------------

## 6. Common Issues

-   **Password prompt**: You may be asked for the database password if
    not provided in the connection string.\
-   **Permission errors**: Ensure the user has `CONNECT`, `CREATE`, and
    `DROP` privileges.\
-   **Errors in restore**: If errors occur, check the PostgreSQL version
    compatibility or clean up conflicting schemas before importing.

------------------------------------------------------------------------

## 7. Example Workflow

``` bash
# Dump from remote server
pg_dump "postgres://root:secret@182.184.67.82:5466/postgres" > prod_dump.sql

# Drop and recreate local database
psql -U postgres -c "DROP DATABASE IF EXISTS stickball;"
psql -U postgres -c "CREATE DATABASE stickball;"

# Restore into local
psql -h localhost -U postgres -d stickball -f prod_dump.sql
```

------------------------------------------------------------------------

## 8. References

-   PostgreSQL Documentation:
    https://www.postgresql.org/docs/current/app-pgdump.html
-   Beekeeper Studio: https://www.beekeeperstudio.io/
