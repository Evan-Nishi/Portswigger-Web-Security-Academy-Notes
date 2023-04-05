# Password Based Vulnerabilities

## Username Enumeration Via Error Code

We can see that a 302 error is returned when a user isn't valid.  Therefore we can see which users are valid by using burp intruder and simply filtering until we get a 200 error code which means the request is OK.  Then the password can be easily bruteforced.

## Username Enumeration Via Different Application Behaviour

After playing around with the application, you can see that if the username is invalid, the message `Invalid username or password.` appears.  We can exploit this to find which username is real by checking each username with the list provided by using the intruder function.  This can be done with burp intruder by checking if the message appears via Grep-Match.  We see that for one username `adam` it appears 0 times, meaning the message changed and that user is real.  In this case, it seems that the period at the end is missing for login attempts with a real username.
Now that we have the username, we can bruteforce with the password list and see which password works by checking for the abscence of the new error message.  

## Username Enumeration Via Request Timing