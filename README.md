# Filtering with AND, OR, and NOT in SQL

> **Disclaimer:** This project was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario and database come from the course. The queries and explanations are my own documentation of the work I performed.

## Project description
My team needed specific information about login activity, employees, and their departments to investigate potential security issues and schedule computer updates. Using SQL in a MariaDB shell, I combined filters with the `AND`, `OR`, and `NOT` operators, along with `LIKE` pattern matching, to retrieve only the records each task required. Filtering on more than one condition lets an analyst narrow a large dataset to the exact records that matter, such as failed logins after business hours.

## Database Tables
**`log_in_attempts`** (user activity)

| Column | Description |
|---|---|
| `event_id` | Unique identifier for the login event |
| `username` | Account that attempted to log in |
| `login_date` | Date of the attempt |
| `login_time` | Time of the attempt |
| `country` | Country the attempt came from |
| `ip_address` | IP address of the attempt |
| `success` | Whether the attempt succeeded (`1` = TRUE, `0` = FALSE) |

**`employees`** (employee information). The tasks in this lab filter on the `department` and `office` columns.

## Operators Used
| Operator | Purpose |
|---|---|
| `AND` | Returns a record only if **both** conditions are true |
| `OR` | Returns a record if **either** condition is true |
| `NOT` | Returns records that do **not** match a condition |
| `LIKE` | Matches a pattern in a column |
| `%` | Wildcard that stands for any number of characters |

MySQL and MariaDB store Boolean values as `1` for TRUE and `0` for FALSE. Boolean values are not placed in single quotation marks because they are not strings.

## Retrieve after-hours failed login attempts
Office hours end at 18:00. To find unsuccessful logins after business hours, I used `AND` to require both conditions: a login time after 18:00 and a failed attempt (`success = 0`).

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00' AND success = 0;
```
Result: **19** failed login attempts after 18:00.

## Retrieve login attempts on specific dates
A suspicious event occurred on 2022-05-09, so I wanted every login attempt from that day and the day before. I used `OR` because a record can match either date:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';
```
Result: **75** login attempts across the two days.

## Retrieve login attempts outside of Mexico
The `country` column contains both `MEX` and `MEXICO` for Mexico. To catch both spellings, I used `LIKE` with the pattern `'MEX%'`, then put `NOT` in front to exclude them:

```sql
SELECT *
FROM log_in_attempts
WHERE NOT country LIKE 'MEX%';
```
Result: **144** login attempts that did not originate in Mexico.

## Retrieve employees in Marketing
The team was updating machines for Marketing employees in the East building, with offices such as `East-170` or `East-320`. I used `AND` to require the Marketing department, and `LIKE` with `'East%'` to match any East building office:

```sql
SELECT *
FROM employees
WHERE department = 'Marketing' AND office LIKE 'East%';
```
Result: the first Marketing employee in the East building is `elarson`.

## Retrieve employees in Finance or Sales
A different update was needed for the Finance and Sales departments. I used `OR`, writing out the full condition on `department` both times:

```sql
SELECT *
FROM employees
WHERE department = 'Finance' OR department = 'Sales';
```
Result: the first Sales employee returned is `lrodriqu`.

## Retrieve all employees not in IT
The Information Technology department already had the update, so I used `NOT` to retrieve everyone else:

```sql
SELECT *
FROM employees
WHERE NOT department = 'Information Technology';
```
Result: **161** employees are not in the Information Technology department.

## Findings
| Task | Query result |
|---|---|
| Failed logins after 18:00 | 19 attempts |
| Logins on 2022-05-08 and 2022-05-09 | 75 attempts |
| Logins outside of Mexico | 144 attempts |
| First Marketing employee in the East building | `elarson` |
| First Sales employee returned (Finance or Sales query) | `lrodriqu` |
| Employees not in Information Technology | 161 employees |

- Combining `login_time` and `success` with `AND` reduced the whole table to the small set of failed after-hours attempts, which is the group worth investigating first for brute force or credential guessing.
- Using `LIKE 'MEX%'` instead of matching a single spelling made sure both `MEX` and `MEXICO` were excluded, so the outside-of-Mexico count was not inflated by inconsistent data entry.
- `LIKE 'East%'` matched every office in the East building with one condition, instead of listing each office number.

## Full Query Sequence
```sql
SELECT * FROM log_in_attempts WHERE login_time > '18:00' AND success = 0;
SELECT * FROM log_in_attempts WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';
SELECT * FROM log_in_attempts WHERE NOT country LIKE 'MEX%';
SELECT * FROM employees WHERE department = 'Marketing' AND office LIKE 'East%';
SELECT * FROM employees WHERE department = 'Finance' OR department = 'Sales';
SELECT * FROM employees WHERE NOT department = 'Information Technology';
```

## Summary
I used `AND`, `OR`, `NOT`, and `LIKE` to filter login and employee data for a security investigation and a round of computer updates. I found 19 failed login attempts after business hours, 75 attempts across the two days around the suspicious event, and 144 attempts from outside Mexico. I then identified the Marketing employees in the East building, the Finance and Sales employees, and the 161 employees outside the IT department who still needed updates. These filters let me retrieve exactly the records each task called for instead of working through the full tables by hand.

## Key Takeaways
- `AND` narrows results because both conditions must be true, while `OR` widens them because either condition can be true.
- `NOT` excludes matching records, which is useful for "everything except" questions like finding employees outside one department.
- `LIKE` with the `%` wildcard matches patterns, which handles inconsistent data such as `MEX` and `MEXICO` in the same column.
- When using `OR` on the same column, write out the full condition both times (`department = 'Finance' OR department = 'Sales'`).
- Boolean values (`1` and `0`) and numbers go without quotation marks, while strings, dates, and times use single quotes.
- Failed logins outside business hours are a common starting point when looking for brute force attempts or compromised accounts.

## Tools
- MariaDB shell
- SQL
