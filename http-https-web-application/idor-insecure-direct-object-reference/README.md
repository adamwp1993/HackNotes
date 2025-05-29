# IDOR - Insecure Direct Object Reference

### Summary

These vulnerabilities are extremely common and occur when the web application exposes direct access to back-end resources and objects, and lacks sufficient access control on the back end to them. Allowing the end-user to directly control or gain access to things they were not intended to have access too.&#x20;

**example:**

```
download.php?file_id=123
```

in this case if there was an IDOR vulnerability, by simply fuzzing the file\_id we could access other users objects and files that were not intended for us.

These vulnerabilities can be used to:

access sensitive data - IDOR Information Disclosure Vulnerability&#x20;

Elevate privileges - IDOR Insecure Function Calls&#x20;
