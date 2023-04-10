# Multi Factor Vulnerabilities

## 2FA Simple Bypass

For this lab, login with the credentials provided.  After playing with the login, you can see that if after you send the verification code and go back to the main menu, the application assumes the verification is complete.  Repeat with the other users credentials.

## 2FA Broken Logic

We can see that the verification process for this lab is broken down into two parts.  First, the applicaton sends a POST to `/login` with the username and password.  Then, the application sends a POST to `/login2` to generate an MFA code.  However the exploit here is that the application allows you to change the cookie for the `verify` paramter, which tells the application which user is being verified.  
To bypass the fact that we don't have the password for `carlos`, we can login as `wiener` and intercept the 2fa POST request to `/login2` and change the `verify` cookie to carlos.  This means the application generates a 2fa code for `carlos` instead.  We can verify this by checking the email inbox and seeing that there is no password generated for `wiener`.   
We can then brute force the MFA since it's only four digits via the `mfa-code` header.  Make sure to set the minimum length to 4, and grep extract for `Incorrect security code` as a successful login will not have it. 
Finally to login to the account, go to dev tools and manually change the `verify` cookie to carlos and use the MFA code.