# SQL Injection Attacks

Note: the final query in the examples is simply a guess of the general structure of what it looks like behind the scenes as we aren't given the source code for the actual queries.  I just found it helpful to visually see code highlighting for some of the query schenanigans that were taking place.

For these attacks I used BurpSuite proxy to intercept requests.  Simply load the lab, and before sending a request to a vulnerable part of the site, enable intercept.  Then when you see the body of the request, send it to repeater.  Then you can see the result of the modified payload when sending using the repeater.

1. [Basics](./basic-injections.md)
2. [Union attack](./union-attacks.md)
3. [Blind injection](./blind-injection.md)