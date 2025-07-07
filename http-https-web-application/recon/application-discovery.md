# Application Discovery

### Scanning for applications

Large enterprise networks can have hundreds of applications running on them and to manually discovery and sift through them would be very tedious and time consuming. Some tactics can be used to make this process less painful. We can scan the network for applications with nmap or masscan and then feed that output into tools such as [EyeWitness](https://github.com/RedSiege/EyeWitness) or [Aquatone](https://github.com/michenriksen/aquatone) which will access the application and take screenshots of the applications running, which is then assembled into a report for much faster viewing. This can then be used to create a more targeted list of applications that we should focus on.&#x20;

```
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list

masscan 10.0.0.0.1 -p 80,443,8000,8080,8180,8888,10000 -oX <filename>

# Multiple targets specific scan
# - Known ports
# - Fast rate 100.000
# - Banner grabbing and another source IP
# - Only open ports
# - Modified user-agent
masscan <target1> <target2> <target3> -p 80,433,8000,8080,8180,8888,10000 --rate 100000 --banners --open-only\
--http-user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0"\
--source-ip 192.168.100.200 -oL "output.txt" 
```

Massscan Cheatsheet: [https://cheatsheet.haax.fr/network/port-scanning/masscan\_cheatsheet/](https://cheatsheet.haax.fr/network/port-scanning/masscan_cheatsheet/)

### Automated Screenshotting with EyeWitness and Aquatone

Using nmaps XML output we can scan all the targets for live web applications, and take and save screenshots.&#x20;

```
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness

cat web_discovery.xml | ./aquatone -nmap

```

Its important to review the output of these tools and determine which of the applications are high value targets. Custom web applications may be home grown and many times may contain a wide array of vulnerabilties.&#x20;
