# Thick Client Applications

Thick client applications run locally on the machine and are not accessible via web browser.&#x20;

For example project management systems, customer relationship management systems, inventory management systems, productivity software, etc. which are installed locally on the machine and accessed via the client application on the machine instead of via a web browser.

#### Two Tier & Three Tier Architecture

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

> Thick client applications are considered less secure than web applications with many possible attack vectors:
>
>
>
> * Improper Error Handling.
> * Hardcoded sensitive data.
> * DLL Hijacking.
> * Buffer Overflow.
> * SQL Injection.
> * Insecure Storage.
> * Session Management.

### Information Gathering&#x20;

1. Identify application architecture, programming languages and frameworks
2. Understand generally how the application and infrastructure work
3. Identify technologies that are used on the client and server sides and find entry points points and user inputs.

Tools to assist with information gathering on thick client applications:&#x20;

{% embed url="https://github.com/horsicq/Detect-It-Easy" %}

{% embed url="https://learn.microsoft.com/en-us/sysinternals/downloads/procmon" %}

{% embed url="https://learn.microsoft.com/en-us/sysinternals/downloads/strings" %}

{% embed url="https://ntcore.com/explorer-suite/" %}

### Client Side Attacks

Sensitive information like usernames, passwords, tokens or strings for communication with other services might be stored in the applications local files, source code etc. We can reverse-engineer .net, java applications with exe, dll, jar, class, and war formats.&#x20;

|                                         |                                      |                                        |                                                |
| --------------------------------------- | ------------------------------------ | -------------------------------------- | ---------------------------------------------- |
| [Ghidra](https://www.ghidra-sre.org/)   | [IDA](https://hex-rays.com/ida-pro/) | [OllyDbg](http://www.ollydbg.de/)      | [Radare2](https://www.radare.org/r/index.html) |
| [dnSpy](https://github.com/dnSpy/dnSpy) | [x64dbg](https://x64dbg.com/)        | [JADX](https://github.com/skylot/jadx) | [Frida](https://frida.re/)                     |

### Network Side Attacks

We may be able to capture sensitive information from the application communicating with the server.

| [Wireshark](https://www.wireshark.org/) | [tcpdump](https://www.tcpdump.org/) | [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview) | [Burp Suite](https://portswigger.net/burp) |
| --------------------------------------- | ----------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------ |

### Server Side Attacks

Similar to web application attacks.

### Reverse Engineering Thick Client Apps

1. Identify an interesting .exe file
2. Use ProcMon64 tp monitor the prcoess and look for a temp file creation.&#x20;
   1.

       `C:\Users\Matt\AppData\Local\Temp`.

       <figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
3. Change permissions of Temp folder to disallow file deletions so we can capture the temp file:
   1.  `Properties` -> `Security` -> `Advanced` -> `cybervaca` -> `Disable inheritance` -> `Convert inherited permissions into explicit permissions on this object` -> `Edit` -> `Show advanced permissions`, we deselect the `Delete subfolders and files`, and `Delete` checkboxes.

       <figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
4. Run the executable now that the temp file cant be deleted with ProcMon.
