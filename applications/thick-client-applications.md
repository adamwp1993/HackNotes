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

### Reverse Engineering Thick Client Apps Example

#### Key Takeaways:

When finding interesting executables, process monitor can identify interesting temp files which may hold more executables.&#x20;

de4dot de-obfuscates .exe files so they can be investigated with source code.&#x20;

dnspy allows us to look at the source code.&#x20;

in some instances you may not need to go through all the steps below, and just plugging the exe into dnspy may get you what you need, or some combination of these steps below.&#x20;

#### Get Temp .bat file&#x20;

1. Identify an interesting .exe file
2. Use ProcMon64 to monitor the process and look for a temp file creation.&#x20;
   1.

       `C:\Users\Matt\AppData\Local\Temp`.

       <figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
3. Change permissions of Temp folder to disallow file deletions so we can capture the temp file:
   1.  `Properties` -> `Security` -> `Advanced` -> `cybervaca` -> `Disable inheritance` -> `Convert inherited permissions into explicit permissions on this object` -> `Edit` -> `Show advanced permissions`, we deselect the `Delete subfolders and files`, and `Delete` checkboxes.

       <figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
4. Run the executable now that the temp file cant be deleted with ProcMon. We should see new temp files generated, for example:
   1.

       ```cmd-session
       C:\Apps>dir C:\Users\cybervaca\AppData\Local\Temp\2

       ...SNIP...
       04/03/2023  02:09 PM         1,730,212 6F39.bat
       04/03/2023  02:09 PM                 0 6F39.tmp

       ```
   2.

       ```batch
       @shift /0
       @echo off

       if %username% == matt goto correcto
       if %username% == frankytech goto correcto
       if %username% == ev4si0n goto correcto
       goto error

       :correcto
       echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
       echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
       <SNIP>
       echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

       echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
       powershell.exe -exec bypass -file c:\programdata\monta.ps1
       del c:\programdata\monta.ps1
       del c:\programdata\oracle.txt
       c:\programdata\restart-service.exe
       del c:\programdata\restart-service.exe
       ```
5.  Notice the batch file is deleting two files that get generated by the service. Lets modify the batch file so we can capture these two other files and analyze them, by removing the lines that delete the file:&#x20;

    <pre class="language-batch"><code class="lang-batch">@shift /0
    @echo off

    echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
    echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
    &#x3C;SNIP>
    echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

    <strong>echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
    </strong></code></pre>
6. Run the program again and we should see the temp oracle.txt and monta.ps1 files that get generated:
   1.

       ```powershell-session
       cat C:\programdata\monta.ps1

       $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida))
       ```
   2. Reviewing the code here, its decoding oracle.txt into restart-service.exe executable, giving us the final executable. We can execute this monta.ps1 to get the final exe.
7. Now we can investigate this restart-service.exe via procmon64 and see it is querying multiple things in the registry:
   1.

       <figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
8.  start `x64dbg`, navigate to `Options` -> `Preferences`, and uncheck everything except `Exit Breakpoint`:

    <figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>
9.  By unchecking the other options, the debugging will start directly from the application's exit point, and we will avoid going through any `dll` files that are loaded before the app starts. Then, we can select `file` -> `open` and select the `restart-service.exe` to import it and start the x64dbg. Once imported, we right click inside the `CPU` view and `Follow in Memory Map`:

    \


    <figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
10. Checking the memory map, in this case of interest is the map with `0000000000003000` with a type of `MAP` and protection set to `-RW--`

    <figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>
11. Memory-mapped files allow apps to access other large files without having to load the entire file into memory at once. the file gets mapped to a memory address block that the application can read and write to as if it was loading in RAM. These are great places to find hardcoded credentials. In this case we can see that this memory mapped file is a DOS MZ executable by the MZ magic bytes in the ASCII column:

    <figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
12. Return to the memory map pane, right click on the selected item and select dump to file. Running strings on this file leads to some interesting information indicating this is a source code file:

    ```powershell-session
    C:\> C:\TOOLS\Strings\strings64.exe .\restart-service_00000000001E0000.bin

    <SNIP>
    "#M
    z\V
    ).NETFramework,Version=v4.0,Profile=Client
    FrameworkDisplayName
    .NET Framework 4 Client Profile
    <SNIP>
    ```
13. We can use `De4Dot` to reverse `.NET` executables back to the source code by dragging the `restart-service_00000000001E0000.bin` onto the `de4dot` executable.

    ```cmd-session
    de4dot v3.1.41592.3405

    Detected Unknown Obfuscator (C:\Users\cybervaca\Desktop\restart-service_00000000001E0000.bin)
    Cleaning C:\Users\cybervaca\Desktop\restart-service_00000000001E0000.bin
    Renaming all obfuscated symbols
    Saving C:\Users\cybervaca\Desktop\restart-service_00000000001E0000-cleaned.bin


    Press any key to exit...
    ```
14. Now we can open the file with DnSpy to read the source code and find credentials:
    1.

        <figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### Exploiting Web Vulnerabilties in Thick-Client applications&#x20;

when thick client apps use a three-tier instead of two-tier architecture, this can open up common web vulnerabilities created by the middlewear sitting between the client and the database.

A walkthrough can be found here: [https://academy.hackthebox.com/module/113/section/2164](https://academy.hackthebox.com/module/113/section/2164)

### Modifying JAR executable source code&#x20;

1. drag and drop the .jar file onto jd-gui.exe and file -> save all sources
2. Apply your code changes in the \*.jar.src folder that the previous step created.
3. Compile your code changes
   1. ```
      javac -cp fatty-client-new.jar fatty-client-new.jar.src\htb\fatty\client\gui\ClientGuiTest.java
      ```
   2. fatty-client-new.jar is the old JAR file we decompiled with jd-gui, and the source from the .jar.src is the code we changed. we are compiling our new code using the old JAR's framework.&#x20;
4. From there we extract our old JAR file into a new folder. create your folder, copy paste the file, extract here.&#x20;
5. Copy paste the newly compiled class files from our .jar.src folder into the folder we just extracted the old jar into to overwrite the old classes with the new ones.&#x20;
   1. `mv -Force fatty-client-new.jar.src\htb\fatty\client\gui*.class raw\htb\fatty\client\gui\`
6. Build our new jar file with the updated class files
   1. ```
      jar -cmf META-INF\MANIFEST.MF new.jar .
      ```
7. Run your new JAR!

### Source Code analysis&#x20;

If we can get our hands on the JAR file, we can decompile and analyze the source code for vulnerabilities like SQL injections or other common web vulnerabilities. Looking at User, login, database functions and classes can be very helpful here.&#x20;
