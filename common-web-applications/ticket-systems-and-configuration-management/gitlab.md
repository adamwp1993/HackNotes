# Gitlab

### Strategy

> Are there any public facing, internal only, or private repos that can be accessed?
>
> Can you log in with previously compromised user credentials?
>
> Can you register an account by getting an email address via another application like osTicket?&#x20;
>
> Can you find SSH keys, API keys, passwords and secrets in source code or configuration files hosted in gitlab/github/bitbucket etc.?

### Footprinting/Discovery

the /help page when logged in is the only way to get the version number.&#x20;

Gitlab has experienced a few serious vulnerabilties but they should not be attempted unless you are certain about the version number.&#x20;

### Enumeration

> without valid credentials there typically is not much you can do with a gitlab install.&#x20;

check /explore for any public projects for review

check groups, snippets, and help.

Check to see if you can register an account if you do not already have a valid one.&#x20;

The Registration form can be used to enumerate valid users

Additional reading:&#x20;

{% embed url="https://tillsongalloway.com/finding-sensitive-information-on-github/index.html" %}

### Username Enumeration&#x20;

This script automates username enumeration:

{% embed url="https://github.com/dpgg101/GitLabUserEnum" %}

> Be careful about locking people out, in versions before 16.6 the lockout was set to 10 attempts. in versions 16.6 and newer, this is configurable by admins.&#x20;

### Authenticated RCE&#x20;

Community versions 13.0.2 and older are vulnerable:&#x20;

{% embed url="https://hackerone.com/reports/1154542" %}

{% embed url="https://www.exploit-db.com/exploits/49951" %}

