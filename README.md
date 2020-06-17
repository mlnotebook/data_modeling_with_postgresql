# Purpose

Sparkify are a new music streaming app. They've been collecting data on the songs in their catalogue as well as user activity. The data are currently found in a directory of JSON logs on user activity, as well as a directory with JSON metadata on the songs in their app. The analytics team wish to understand what songs users are listening to, but there's currently no straightforward and simply way to query the data in its current form.

# Schema
This project creates a PostgreSQL database schema and ETL pipeline to perform this analysis. The database is a "Star Schema" and is designed to optimize queries on song play analysis. The star schema comprises fact and dimension tables. Dimension tables hold information about different components of the database, _e.g._ users or songs, while the fact table pulls this information together to gather rich information about the data. The Schema is also shown in Figure 1.

**Fact Table**:
* `songplays`
**Dimension Tables**:
* `users`
* `songs`
* `artists`
* `time`

![StarSchema](./images/project1_schema.png#center) 
Figure 1: The Star Schema modeled in this project with one fact table and 4 dimension tables.
      
# Usage

1. First the `create_tables.py` script must be executed in order to `CREATE` the tables and populate the `INSERT` queries.
2. Then `etl.py` may be executed which processes all of the song and log datafiles and `INSERT`s the data into the database.
3. The lines in `test.ipynb` can be used to ensure that the data has been correctly inserted into the tables.
<mark>Note: The `songplays` table will only contain ONE field where `song_id` and `artist_id` are not `None`. This is expected behaviour as this is a subset of a much larger dataset.</mark>
4. `examples.ipynb` show examples of querying the database.

# Examples

The `example.ipynb` notebook contains an example of reading the tables and querying them using the iPython notebook magic `%sql` commands.

# Script Details

## `sql_queries.py`

This file contains the queries that are used to:
* Drop tables (if they exist)
* Create tables

### `DROP` Tables

The syntax `DROP TABLE IF EXISTS <table_name>` allows us to first check if `<table_name>` exists, and if it does, remove it. If the `IF EXISTS` were omitted, then PostgreSQL would throw an error tell us that `<table_name>` does not exist.

The `DROP` queries are used to destroy each table before new versions are created by `create_tables.py`. Each of the `DROP` queries are executed after a connection to the database is established, but before creating the tables again from scratch.

### `CREATE` Tables

The usual syntax is used to create each table:
```
"""CREATE TABLE IF NOT EXISTS <table_name> (<column1_name> <column1_type>, <column2_name> <column2_type>);"""
```
The `IF NOT EXISTS` clause is included to ensure that no errors are thown if the table already exists. However, these tables are created after the `DROP TABLE` queries are executed, so this should not be necessary. It is included in case the `CREATE TABLE` queries are called from elsewhere.

### `INSERT` records

The data are inserted into the tables using the syntax:
```
"""INSERT INTO <table_name> (<column1_name>, <column2_name>) VALUES (%s, %s)"""
```
The placeholders `%s` are filled by the ETL pipeline when reading from the `.csv` and `.json` files.

## `etl.py`

### `process_song_file`

This function reads a 'song file' saved as `.json` into a Pandas DataFrame. It extracts the columns, by name, which are relevant for the `songs` and `artists` tables then `INSERT`s them into the corrresponding tables. The columns for each table are:
* `songs`: `'song_id', 'title', 'artist_id', 'year', 'duration'`
* `artists`: `'artist_id', 'artist_name', 'artist_location', 'artist_latitude', 'artist_longitude'`

### `process_log_file`

This function reads a 'log file' saved as `.json` into a Pandas DataFrame and filters by the `NextSong` action. It adds data to two tables, `time` and `songplays`.

* `time`: The `timestamp` is parsed by converting to `datetime` with Pandas `.to_datetime` method from which 'hour', 'day', 'week_of_year', 'month', 'year', and 'weekday' values are extracted. A new Pandas DataFrame is constructed from the time data and `INSERT`ed into the `time` table.
* `songplays`: The 'song', 'artist' and 'duration' are used to query the `songs` table  in order to find the `song_id` and `artist_id` - this is accomplished with a `JOIN` on the `artist_id` column for the `songs` and `artists` tables. The `song_id` and `artist_id` as well as relevant information from the log DataFrame are `INSERT`ed into the `songplays` table.

### `process_data`

This function globs a list of the song or log files in the data directory. It iterates through each file and calls the appropriate `process*` function from above.

## `test.ipynb`

The functions in this notebook allow us to see if the tables have been created and if the data have been correctly inserted into the appropriate tables.
