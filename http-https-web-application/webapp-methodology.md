---
description: Checkbook for enumerating web applications
---

# WebApp Methodology

* [ ] Passive Recon
* [ ] Active Recon
  * [ ] Can you identify what underlying technologies the web server is using with wappalyzer, nikto, nmap, WhatWeb, BuiltWith, Netcat?
    * [ ] Any known vulnerabilites / well known technologies?
    * [ ] What programming languages and web frameworks does the application use?
    * [ ] what makes up the back end? databases, CMS, etc.
  * [ ] Analyze source code - Sensitive data disclosure? Front end input validation?
    * [ ] Robots.txt
  * [ ] Fuzzing Vhosts, Subdirectories, Subdomains, Crawling
    * [ ] Can you identify any hidden administrator portals or endpoints?
  * [ ] Attacking Login Portals&#x20;
    * [ ] Can you identify possible users and usernames?
    * [ ] Can you password Spray these and compromise a user?
    * [ ] Can you register a user account?
    * [ ] If so, what does this give you access too? are there any exploitable functionalities?
  * [ ] Walk the happy path - explore the application from a user perspective
    * [ ] Can you bypass access control?
    * [ ] Can you identify all places in the application that take user input?
    * [ ] Can you identify all URL parameters?
    * [ ] Can you identify all HTTP request parameters?
  * [ ] Display user input on page - Possible XSS?
  * [ ] Request interacting with Database - SQL Injection?
  * [ ] Parameter is adjusting content on page from a file - LFI/RFI?
    * [ ] i.e lang=eng.php
  * [ ] User input is used to execute some utility/ pass input to command line - Command Injection?
  * [ ] Application takes and processes XML input on the backend - XXE?
  * [ ] 403 errrors - Can we use HTTP verb tampering to bypass?
    * [ ] If injections are being filtered, can we use HTTP verb tampering to try to bypass the filtering?&#x20;
