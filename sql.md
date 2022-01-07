## How to create a database from scratch

I used instructions from [this page](https://mungingdata.com/sqlite/create-database-load-csv-python/)


```python
import sqlite3
import pandas as pd

### create SQLite DB and create table
!touch ../data/velib.sqlite
conn = sqlite3.connect('../data/velib.sqlite')
c = conn.cursor()
c.execute("""
CREATE TABLE stations (datetime text, stationCode int, meca int, elec int, park int)
""")

### write CSV table to SQLite DB

# import table from CSV (10 seconds)
stations = pd.read_csv('../data/data-5m.csv', low_memory=False)

# clean the table with pandas (15 seconds)
stations['datetime'] = pd.to_datetime(stations.datetime, format='%Y-%m-%d %H-%M')
for col in ['stationCode', 'meca', 'elec', 'park']:
    stations[col] = pd.to_numeric(stations[col], errors='coerce')
stations = stations.dropna()
for col in ['stationCode', 'meca', 'elec', 'park']:
    stations[col] = stations[col].astype(int)

# write table to SQLite DB (90 seconds)
stations.to_sql('stations', conn, if_exists='append', index=False)

### test simple query with sqlite3 only (7 ms)
c.execute("""
SELECT * FROM stations
""")
c.fetchone()

### test query with pandas (< 2 seconds)
query = """
SELECT s.datetime, s.meca + s.elec AS bikes, s.park
FROM stations s
WHERE s.stationCode = 12001
"""
pd.read_sql_query(query, conn).head()

```

## Queries

### Date related queries
From [this page](https://www.sqlitetutorial.net/sqlite-date/):
> SQLite does not support built-in date and/or time storage class. 
>
> Instead, it leverages some built-in date and time functions to use other storage classes such as `TEXT`, `REAL`, or `INTEGER` for storing the date and time values.


```SQL
SELECT datetime('now')
-- 2022-01-01 14:30:15
SELECT datetime('now', 'localtime')
-- 2022-01-01 15:33:08
```

### *Sanitized* injection
```python
query = """
        SELECT s.datetime, s.meca + s.elec AS bikes, s.park
        FROM stations s
        WHERE (s.stationCode = ? 
            AND s.datetime BETWEEN ? AND ?)
    """
    params = (str(station_code), str(first_dt), str(last_dt))
    df = pd.read_sql_query(query, conn, params=params, parse_dates="datetime", index_col="datetime")
```

## Proof of concept
Let query be:
```SQL
SELECT s.datetime, s.meca + s.elec AS bikes, s.park
FROM stations s
WHERE (s.stationCode = 12001 AND s.datetime BETWEEN '2021-11-01' AND '2021-11-08')
```
This was WAYYYY more rapid than pandas
```python
### Import and connect (< 200 ms)
import sqlite3
import pandas as pd
conn = sqlite3.connect('../data/velib.sqlite')
c = conn.cursor()

# query --> see SQL code above
### this takes 1.40 s
df = pd.read_sql_query(query, conn, parse_dates="datetime", index_col="datetime")

### standard rendering (< 1 s)
import matplotlib.pyplot as plt 
import seaborn as sns

sns.lineplot(data=df, x='datetime', y='bikes', color='green')
sns.lineplot(data=df, x='datetime', y='park', color='purple')
plt.xlabel('')
plt.ylabel('')

for label in plt.gca().get_xticklabels():
    label.set_rotation(45)
    label.set_ha('right')
```

## Production
- nothing to be put in requirements.txt: `sqlite` is part of the standard library
> *after stubbornly trying to download SQLITE3 from pip for several minutes, i remembered that python is batteries included.* -- quote from [this stackOF post](https://stackoverflow.com/questions/35605677/python-sqlite-no-matching-distribution-found-for-sqlite)
- manage connector: 
    - I had an error because of multi-threads, which can be overcome with `check_same_thread=False`
    - use a `with sqlite3.connect(DB_PATH) as conn` statement is best
    - more on [this stackOF post](https://stackoverflow.com/questions/48218065/programmingerror-sqlite-objects-created-in-a-thread-can-only-be-used-in-that-sa)