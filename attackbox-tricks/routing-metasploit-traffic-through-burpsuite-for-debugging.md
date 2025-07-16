# Routing Metasploit traffic through BurpSuite for Debugging

```shell-session
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set PROXIES HTTP:127.0.0.1:8080
```
