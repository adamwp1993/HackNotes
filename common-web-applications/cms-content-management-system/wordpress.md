# Wordpress

WordPress is an extremely common CMS. It is written in PHP and typically runs with  Apache and MySQL.

## Discovery/Enumeration

### Identify Wordpress

Robots.txt:

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```

The presence of /wp-admin and /wp-content directories are good indicators that the site is running wordpress.

Wappalyzer can also indicate what technologies the web app is using.

We can also look for wordpress in the source code of the page:

```shell-session
Adam-162@htb[/htb]$ curl -s http://blog.inlanefreight.local | grep WordPress

<meta name="generator" content="WordPress 5.8"  
```

### Wordpress User Accounts&#x20;

1. Administrator: This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code.
2. Editor: An editor can publish and manage posts, including the posts of other users.
3. Author: They can publish and manage their own posts.
4. Contributor: These users can write and manage their own posts but cannot publish them.
5. Subscriber: These are standard users who can browse posts and edit their profiles.

Getting access to an administrator is usually sufficient to obtain code execution on the server. Editors and authors might have access to certain vulnerable plugins, which normal users don’t.

### Enumerating Users

Default Wordpress login is found at /wp-login.

Attempt a few login attempts and see if the error message indicates that a user is not registered on the site. If this is the case, we can enumerate for valid usernames.&#x20;

### Plugins/Themes

Wordpress stores its plugins in wp-content/plugins directory, and themes are stored in wp-content/themes. these should be investigated for vulnerable plugins and themes which can lead to RCE.

Once we've compiled a list of potential plugins and themes, it's crucial to check for publicly disclosed vulnerabilities associated with these components. Sites like WPScan’s Vulnerability Database can be instrumental in identifying any known security issues, helping us determine potential vectors for exploitation.

Finding Plugins:

```shell-session
Adam-162@htb[/htb]$ curl -s http://blog.inlanefreight.local/ | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.8' id='validation-engine-en-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8' id='validation-engine-js'></script>
		<link rel='stylesheet' id='mm_frontend-css'  href='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/js/index.js?ver=5.4.2' id='contact-form-7-js'></script>
```

Identifying plugin versions:

Navigating too [http://blog.inlanefreight.local/wp-content/plugins](http://blog.inlanefreight.local/wp-content/plugins/mail-masta/)/\<plugin> and checking the readme.txt can give us information on the version of the plugin running.&#x20;

###

