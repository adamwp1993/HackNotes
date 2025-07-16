# Tomcat

### Discovery/Footprinting

Many times if discovered via eyewitness it will be flagged as a High Value Target. These installs are typically not public internet facing, but are found many times on internal assessments.&#x20;

By requesting an invalid page, many times the tomcat server will return the server and version:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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

### Login Brute Force

