🇧🇷 [Português](README.md) | 🇺🇸 English

# Linux and SQL — log analysis and database queries

Sprint 6 project from the TripleTen QA Engineer program: using the **Linux terminal** to analyze web server logs and **SQL (PostgreSQL)** to query a database of Chicago taxi rides.

## Part 1 — Linux terminal (Apache server logs)

**Task 1.** Search the December 2019 logs for requests from IPs starting with `233.201`, using `cd` and `grep -R` with a regular expression. 2 requests were found (on the 18th and 21st).

**Task 2.** Split the errors from one day of logs by status code:

```bash
mkdir bug1
mkdir bug1/events

grep '[45]00' ~/logs/2019/12/apache_2019-12-30.txt > ~/bug1/main.txt

grep '400' ~/bug1/main.txt > ~/bug1/events/400.txt
grep '500' ~/bug1/main.txt > ~/bug1/events/500.txt
```

The results were checked with `wc`, `head -3` and `tail -3`: `500.txt` had 160 lines and `400.txt` was empty.

## Part 2 — Database (PostgreSQL)

**Task 1. Number of cars:** 5529.

```sql
SELECT count(*) AS COUNT from cabs;
```

**Task 2. Companies with fewer than 100 cars** (51 companies):

```sql
select company_name, COUNT(*) AS cnt FROM cabs
GROUP BY company_name
HAVING count(*)<100
ORDER BY COUNT(*) DESC;
```

**Task 3. Weather condition for each hour on 11/05/2017** (`GOOD` or `BAD`):

```sql
SELECT CASE WHEN description LIKE '%rain%' THEN 'BAD'
       WHEN description LIKE '%STORM%' THEN 'BAD'
       ELSE 'GOOD'
       END AS weather_conditions, TS
FROM weather_records
WHERE weather_records.TS BETWEEN '2017-11-05 00:00:00' AND '2017-11-05 23:59:59';
```

**Task 4. Rides per company on 11/15 and 11/16/2017** (64 companies, from 19,558 rides for Flash Cab down to 2 for RC Andrews Cab):

```sql
select cabs.company_name, COUNT(*) AS trips_amount FROM cabs
INNER JOIN trips on trips.cab_id=cabs.cab_id
where trips.start_ts BETWEEN '2017-11-15 00:00:00' AND '2017-11-16 23:59:59'
group by cabs.company_name
ORDER BY COUNT(*) DESC;
```

All full results are in the project PDF (in Portuguese).

## Improvements suggested in the review

The project was approved. The reviewer suggested these improvements. They change the results, and the versions below **have not yet been run** in the course environment.

**1. Filter the status code with context in `grep`.** Searching only for `'500'` also matches lines where 500 appears in the response size or the IP (false positives). That is why the file ended up with 160 lines instead of 156.

```bash
grep ' HTTP/1.1" 400 ' ~/bug1/main.txt > ~/bug1/events/400.txt
grep ' HTTP/1.1" 500 ' ~/bug1/main.txt > ~/bug1/events/500.txt
```

**2. Count unique cars with `COUNT(DISTINCT ...)`.** `COUNT(*)` counts every row, including duplicates.

```sql
SELECT COUNT(DISTINCT vehicle_id) AS cnt FROM cabs;
```

**3. Watch out for case in `LIKE`.** In PostgreSQL, `LIKE` is case-sensitive, so `'%STORM%'` does not match "storm".

```sql
WHEN description LIKE '%rain%' OR description LIKE '%storm%' THEN 'BAD'
```

## Files

- [`projeto-sprint-6-linux-e-sql.pdf`](projeto-sprint-6-linux-e-sql.pdf): the full project, with all commands, queries and results (in Portuguese).

## What I learned

- **Command line:** navigating folders, creating directories and filtering logs with `grep`, `wc`, `head` and `tail`.
- **SQL:** queries with `COUNT`, `GROUP BY`, `HAVING`, `CASE`, `BETWEEN` and `INNER JOIN`.
- **PostgreSQL:** querying a real database with several related tables.
- **Precise filters:** a generic filter can produce false positives, and a good test has to check whether the result makes sense.
