# LDAP

Directory protocol that is used to access and manage directory information. Active Directory utilizes this protocol. it is used to store information on users, groups, machines and provide centralized authentication.

> OpenLDAP is an open source directory software that is also widely used&#x20;
>
> Active directory is the most prominent example and is used almost everywhere.

| **LDAP**                                                                                                                                   | **Active Directory (AD)**                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| A `protocol` that defines how clients and servers communicate with each other to access and manipulate data stored in a directory service. | A `directory server` that uses LDAP as one of its protocols to provide authentication, authorisation, and other services for Windows-based networks.                                                         |
| An `open and cross-platform protocol` that can be used with different types of directory servers and applications.                         | `Proprietary software` that only works with Windows-based systems and requires additional components such as DNS (Domain Name System) and Kerberos for its functionality.                                    |
| It has a `flexible and extensible schema` that allows custom attributes and object classes to be defined by administrators or developers.  | It has a `predefined schema` that follows and extends the X.500 standard with additional object classes and attributes specific to Windows environments. Modifications should be made with caution and care. |
| Supports `multiple authentication mechanisms` such as simple bind, SASL, etc.                                                              | It supports `Kerberos` as its primary authentication mechanism but also supports NTLM (NT LAN Manager) and LDAP over SSL/TLS for backward compatibility.                                                     |

LDAP uses a client-server Architecture where clients send requests to the LDAP server and recieve responses.

`LDAP requests` are `messages` that clients send to servers to `perform operations` on data stored in a directory service. An LDAP request is comprised of several components:

1. `Session connection`: The client connects to the server via an LDAP port (usually 389 or 636).
2. `Request type`: The client specifies the operation it wants to perform, such as `bind`, `search`, etc.
3. `Request parameters`: The client provides additional information for the request, such as the `distinguished name` (DN) of the entry to be accessed or modified, the scope and filter of the search query, the attributes and values to be added or changed, etc.
4. `Request ID`: The client assigns a unique identifier for each request to match it with the corresponding response from the server.

Once the server receives the request, it processes it and sends back a response message that includes several components:

1. `Response type`: The server indicates the operation that was performed in response to the request.
2. `Result code`: The server indicates whether or not the operation was successful and why.
3. `Matched DN:` If applicable, the server returns the DN of the closest existing entry that matches the request.
4. `Referral`: The server returns a URL of another server that may have more information about the request, if applicable.
5. `Response data`: The server returns any additional data related to the response, such as the attributes and values of an entry that was searched or modified.

After receiving and processing the response, the client disconnects from the LDAP port.

### ldapsearch

ldapsearch is a great tool to enumerate ldap directories and pull information out of them:

```shell-session
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```

### LDAP Injection

`LDAP injection` is an attack that `exploits web applications that use LDAP` (Lightweight Directory Access Protocol) for authentication or storing user information. The attacker can `inject malicious code` or `characters` into LDAP queries to alter the application's behaviour, `bypass security measures`, and `access sensitive data` stored in the LDAP directory.

| Input    | Description                                                                                                                                                                                                                                          |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`      | An asterisk `*` can `match any number of characters`.                                                                                                                                                                                                |
| `( )`    | Parentheses `( )` can `group expressions`.                                                                                                                                                                                                           |
| `\|`     | A vertical bar `\|` can perform `logical OR`.                                                                                                                                                                                                        |
| `&`      | An ampersand `&` can perform `logical AND`.                                                                                                                                                                                                          |
| `(cn=*)` | Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields. |

These are very similar to SQL injection attacks but we are looking for fields that interact with an ldap directory instead of a database. \


an example:

```
# vulnerable LDAP query web application uses:
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))

# with the wildcard, the query returns true as long as a valid username is passed
$username = "dummy";
$password = "*";
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

#### Prevention:

To mitigate the risks associated with LDAP injection attacks, it is crucial to `thoroughly validate` and `sanitize user input` before incorporating it into LDAP queries. This process should involve `removing LDAP-specific special characters` like `*` and `employing parameterised queries` to ensure user input is `treated solely as data`, not executable code.



### Scanning

```shell-session
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 14:43 SAST
Nmap scan report for 10.129.204.229
Host is up (0.18s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Login
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 149.73 seconds
```

we can guess that the web application may be integrated with LDAP in this instance.&#x20;
