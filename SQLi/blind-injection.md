# Blind Injection Attacks

Somtimes, SQL queries don't actually modify the HTML on the page itself or return direct results that an attacker can see (hence the name).  This requires some creativity as shown below.

## Blind SQL Injection With Conditionals
>
>We can do this lab by injecting the TrackingID cookie as we were given the query in the lesson: `SELECT TrackingId FROM TrackedUsers WHERE TrackingId = '[input here]'`.  We can also see that a "welcome back" message appears if you reload the page, which means that the welcome message is dependant upon the TrackingId cookie.  We can then use this welcome message as the output of our attack given we can't directly see the application. Think of the "welcome back" message being a single bit of information that we can directly get from the backend.
>
>We're already given information that there is a `users` table with a `username` and `password` columns, but it would be wise to check if the admin user is actually there.
>
>We can inject into the query `[cookie here]' AND (SELECT 'a' FROM users WHERE username='administrator')='a`.
>How this works is the query in the parethesis will only resolve to an 'a' if the users table actually has a user called adminstrator.  If not it will just retrieve nothing as the entry doesn't exist.  Therefore we can compare it to 'a' to check if the user actually exists.
>
>The query would end up being something like this:
>```SQL
> SELECT TrackingId FROM TrackedUsers WHERE TrackingID='lxTJ2U7Fz8arF9k3' AND (SELECT 'a' FROM users WHERE username = 'adminstrator') = 'a'
>```
>Once we see that the user exists, we should try to see the length of the password so we can use conditionals to crack it.  We can do this with the burp intruder and the following payload: `TrackingId=[cookie here]' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=1)='a`.  We can then use burp intruder to keep sending the request and incrementing the length comparison by one each request.  When the condition is true (ie the password is the length of the comparison), the "welcome back" message will display.  In this case the password is 20 characters long.

>The query would end up being something like this:
>```SQL
> SELECT TrackingId FROM TrackedUsers WHERE TrackingID=[cookie here]' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=1)='a'
>```
>Next we have to make burp intruder guess letter of the password.  We are able to guess the password one character at a time (making this mathematically possible) using to the substring operator.  Instead of guessing, a binary search would be more effiecient and would be helpful when dealing with a site with rate limits or brute force detection, but for this example isn't really necessary.
>
>NOTE: SQL indexes start at 1 
>
>For guessing each character, we can use a payload to make the following query:
>```SQL
> SELECT TrackingId FROM TrackedUsers WHERE TrackingID=[cookie here]' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'
>```
>Then you use burp intruder with the "cluster bomb" attack type to iterate through each character and each position of the password.
>Solution: nemafk5jeq0q5rf4zsmm
