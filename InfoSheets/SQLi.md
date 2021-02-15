# SQLi Guide
# Based on [this article from OWASP](https://owasp.org/www-community/attacks/SQL_Injection)

### Overview
A SQL injection attack consists of insertion or "injection" of a SQL query via the input data from the client to the application. A successful SQLi can read sensitive data from the DB, modify DB (insert, update, delete), execute admin operations on the database (like bring it down), recover the content of a given file present on the DBMS FS and in some cases issue commands to the OS


### Types [(Link)](https://www.acunetix.com/websitesecurity/sql-injection2/)

##### Classic (In-band)
Most common and easy to exploit. Occurs when an attacker is able to use the same communication channel to both launch the attack and gather results.
  - Error-based SQLi: relies on error messages thrown by the DB to obtain information about structure of the database. In some cases, error-based SQL injection alone is enough for an attacker to enumerate an entire database. While erros are very useful during the development phase of a webapp, they should be disabled on a live site, or logged to a file with restricted access instead.
  - Union-based SQLi: leverages the UNION SQL operator to combine the results of two or more SELECT statements into a single result which is then returned as part of the HTTP response

##### Blind/Inferential
May take longer for an attacker to exploit, but it is just as dangerous. In this type of attack, no data is actually transferred via the webapp and the attacker would not be able to see the result of an attack in-band. Instead, an attacker is able to reconstruct the database structure by sending payloads, observing the webapps response and the resulting behavior of the database server
  - Boolean-based (content-based): relies on sending a SQL query to the DB which forces the app to return a different result depending on whether the query returns a TRUE or a FALSE result. Depending on the result, the content within the HTTP response will change, or remain the same. This allows an attacker to infer if the payload used returned true or false, even though no data from the DB is returned
  - Time-based: relies on sending a SQL query to the DB which forces it to wait for a specified amount of time (in seconds) before responding. The response time will indicate to the attacker whether the result of the query is TRUE or FALSE. Depending on the result, an HTTP response will be returned with a delay, or returned immediately

##### Out-of-band
Not very common, because it depends on features being enabled on the database server being used by the web app. Out-of-band SQLi occurs when an attacker is unable to use the same channel to launch the attack and gather results. Offers the attacker an alternative to inferential time-based techniques, especially if the server responses are not very stable. Out-of-band SQLi techniques would rely on the database server's ability to make DNS or HTTP requests to deliver data to an attacker. Such is the case with Microsoft SQL Server's xp_dirtree command, which can be used to make DNS requests to a server an attacker controls; as well as Oracle Database's UTL_HTTP package, which can be used to send HTTP requests from SQL and PL/SQL to a server an attacker controls.


### Defense and Prevention [(Link)](https://portswigger.net/web-security/sql-injection)
Most instances of SQLi can be prevented by using parameterized queries (also known as PREPARED STATEMENTS) instead of string concatenation within the query. Parameterized queries can be used for any situation where untrusted input appears as data within the query, including the WHERE clause and values in an INSERT or UPDATE statement. They can't be used to handle untrusted input in other parts of the query, such as table or column names, or the ORDER BY clause. App functionality that places untrusted data into those parts of the query will need to take a different approach, such as white-listing permitted input values, or using different logic to deliver the required behavior. For a parameterized query to be effective in preventing SQLi, the string that is used in the query must always be a hard-coded constant, and must never contain any variable data from any origin. Do not be tempted to decided case-by-case whether an item of data is trusted, and continue using string concatenation within the query for cases that are considered safe. It is all too easy to make mistakes about the possible origin of data, or for changes in other code to violate assumptions about what data is tainted.


### SQLi Practice [(Link)](https://injection.pythonanywhere.com/sql1/)

##### Level 1
  - Task: must show all of the shoes listed in the table
  - searching for `' OR 1=1--` solves the challenge
  - single tick to stop the user input string, "OR 1=1" will always evaluate to true, double dash at the end to comment out the rest of the query

##### Level 2
  - Task: must log in successfully without knowing username/password
  - again, typing `' OR 1=1--` into both the username and password fields solves the challenge
  - works with just sending the username as the injection string, doesn't have to have the password as well since it most likely uses a "WHERE username=*stuff* AND password=*stuff*"
    - and if we comment out the rest of the query after our username, we don't even need to put in a password (unless there's some client-side checks first)
