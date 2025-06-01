# Preventing IDOR

IDOR vulnerabilities are the result of weak access control on the back-end of the web application.&#x20;

### Object Level Access Control&#x20;

We should implement a RBAC (Role based access control) system to all objects and resources on the back end server, to prevent un-intended access to sensitive resources on the server. Additonally, the access control needs to be implemented on the back end, as the client side can be modified and bypassed.&#x20;

### Object Referencing&#x20;

Avoiding direct object referencing is preferable where possible - if we have to use a direct reference, we need to ensure that we have a robust access control system implemented, and we must aboid using clear text or easily guessed patterns for our references (i.e uid=1).

1. avoid direct object reference where possible&#x20;
2. ensure a robust access control system is implemented&#x20;
3. do not use easily guesses patterns but instead use salted hashes for references&#x20;
4. do not calculate hashes on the front end as it will be easily reversed&#x20;

