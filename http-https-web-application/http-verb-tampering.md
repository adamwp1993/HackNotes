# HTTP Verb Tampering

## Vulnerability Explanation&#x20;

Web applications may be insecurely coded or configured to accept HTTP requests with HTTP verbs that were not intended by the developer, which may be able to be abused.&#x20;

HTTP verbs:

1. GET - Retrieves information/data.
2. HEAD - Identical to GET, but the response only includes the header, not the body.
3. OPTIONS - Shows different options accepted by web server
4. TRACE - Echoes back the received request, used for diagnostic purposes to see what is received at the server.
5. PUT - Writes request payload to specified location
6. DELETE - Removes the specified resource from the server.
7. POST - Sends data to back end for processing or storage, etc.&#x20;
8. PATCH - Apply partial modifications to the resource at specified location
9. CONNECT - Establishes a tunnel to the server, typically used for SSL (HTTPS) connections through a proxy.

If enabled, the Web Distributed Authoring and Version [(WebDAV)](http://www.webdav.org/specs/rfc2518.html) [extensions](https://tools.ietf.org/html/rfc4918) permit several more HTTP methods:

* [`PROPFIND`](http://www.webdav.org/specs/rfc2518.html#METHOD_PROPFIND)
* [`PROPPATCH`](http://www.webdav.org/specs/rfc2518.html#METHOD_PROPPATCH)
* [`MKCOL`](http://www.webdav.org/specs/rfc2518.html#METHOD_MKCOL)
* [`COPY`](http://www.webdav.org/specs/rfc2518.html#METHOD_COPY)
* [`MOVE`](http://www.webdav.org/specs/rfc2518.html#METHOD_MOVE)
* [`LOCK`](http://www.webdav.org/specs/rfc2518.html#METHOD_LOCK)
* [`UNLOCK`](http://www.webdav.org/specs/rfc2518.html#METHOD_UNLOCK)

#### Insecure server configuration

Insecure web server configurations can introduce HTTP Verb Tampering vulnerabilities by limiting authentication to specific methods like `GET` and `POST`. This allows attackers to use other methods, such as `HEAD`, to bypass authentication and access restricted areas. Properly securing all HTTP methods is crucial to prevent these vulnerabilities.

XML of vulnerable server configuration::

```xml
<Limit GET POST>
    Require valid-user
</Limit>
```

#### Insecure Coding&#x20;

Verb tampering occurs when security measures only apply to certain types of HTTP requests, such as GET requests, while neglecting others, like POST requests. In this case, the application only sanitizes GET parameters to prevent SQL Injection. However, the application uses `$_REQUEST["code"]`, which also accepts POST data, making it susceptible to injections.

**Vulnerability Explanation**

* **Issue**: The code only filters GET parameters, but the query relies on `$_REQUEST`, which includes both GET and POST data.
* **Risk**: Attackers can send a malicious POST request to bypass the GET filter, potentially injecting SQL commands.
* **Solution**: Ensure input validation and sanitization apply uniformly across all HTTP methods, not just GET.

Strengthen security by validating and sanitizing inputs consistently, regardless of the HTTP

## Exploitation

This Vulnerability can be used many times to bypass authentication to restricted resources. IF we identify a file or directory that we get a 403 or 401 error for, we can capture the request with Burp Suite and modify the request header to test the verbs, and look for a 200 response:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### [HTTP Verbs/Methods Fuzzing](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/403-and-401-bypasses.html#http-verbsmethods-fuzzing) <a href="#http-verbsmethods-fuzzing" id="http-verbsmethods-fuzzing"></a>

Try using **different verbs** to access the file: `GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE, PATCH, INVENTED, HACK`

* Check the response headers, maybe some information can be given. For example, a **200 response** to **HEAD** with `Content-Length: 55` means that the **HEAD verb can access the info**. But you still need to find a way to exfiltrate that info.
* Using a HTTP header like `X-HTTP-Method-Override: PUT` can overwrite the verb used.
* Use **`TRACE`** verb and if you are very lucky maybe in the response you can see also the **headers added by intermediate proxies** that might be useful.

If enabled, the Web Distributed Authoring and Version [(WebDAV)](http://www.webdav.org/specs/rfc2518.html) [extensions](https://tools.ietf.org/html/rfc4918) permit several more HTTP methods:

* [`PROPFIND`](http://www.webdav.org/specs/rfc2518.html#METHOD_PROPFIND)
* [`PROPPATCH`](http://www.webdav.org/specs/rfc2518.html#METHOD_PROPPATCH)
* [`MKCOL`](http://www.webdav.org/specs/rfc2518.html#METHOD_MKCOL)
* [`COPY`](http://www.webdav.org/specs/rfc2518.html#METHOD_COPY)
* [`MOVE`](http://www.webdav.org/specs/rfc2518.html#METHOD_MOVE)
* [`LOCK`](http://www.webdav.org/specs/rfc2518.html#METHOD_LOCK)
* [`UNLOCK`](http://www.webdav.org/specs/rfc2518.html#METHOD_UNLOCK)

### Bypassing Security Filters

Security Filters may be in place to detect malicious requests to detect injection attempts on POST parameters. in some cases, the application may have been insecurely coded to only enforce these on expected HTTP verbs, but not others. Look for blocked requests when attempting injections, these will typically provide some type of error that the request was denied or malicious of some sort. **when attempting some sort of injection attack that is being filtered, we should also see if we can bypass the security filters by using different HTTP verbs.**



### Prevention

Verb tampering vulnerabilities are typically produced either by insecure server configuration or insecure coding (or both).&#x20;

#### Insecure Server Configuration

The following server configuration examples demonstrate an authentication bypass vulnerability via HTTP verb tampering. in these examples, the authentication for the admin directory is required, but only on get requests. making it possible to access these resources via other HTTP verbs, bypassing authentication requirements.&#x20;

Apache:

```xml
<Directory "/var/www/html/admin">
    AuthType Basic
    AuthName "Admin Panel"
    AuthUserFile /etc/apache2/.htpasswd
    <Limit GET>
        Require valid-user
    </Limit>
</Directory>
```

Tomcat:

```xml
<security-constraint>
    <web-resource-collection>
        <url-pattern>/admin/*</url-pattern>
        <http-method>GET</http-method>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>
```

ASP.net:

```xml
<system.web>
    <authorization>
        <allow verbs="GET" roles="admin">
            <deny verbs="GET" users="*">
        </deny>
        </allow>
    </authorization>
</system.web>
```

In order to prevent these, we can specify a single allowed method using safe keywords such as:

Apache: LimitExcept

Tomcat: http-method-omission

ASP.net: add/remove

Also consider disabling or denying all head requests, unless specifically needed.&#x20;

#### Insecure Coding

In the following example, we see where our command injection filter is only applied by the web applications code on POST requests, making requests via other method bypass the filtering.

```php
if (isset($_REQUEST['filename'])) {
    if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {
        system("touch " . $_REQUEST['filename']);
    } else {
        echo "Malicious Request Denied!";
    }
}
```

In order to prevent this, we must be consistent with our use of HTTP methods and ensure all methods are utilizing the filtering.

| Language | Function                        |
| -------- | ------------------------------- |
| PHP      | `$_REQUEST['param']`            |
| Java     | `request.getParameter('param')` |
| C#       | `Request['param']`              |

## Links and Resources

{% embed url="https://academy.hackthebox.com/module/134" %}

{% embed url="https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/403-and-401-bypasses.html#http-verbsmethods-fuzzing" %}

{% embed url="https://nmap.org/nsedoc/scripts/http-method-tamper.html" %}
nmap script for detection
{% endembed %}

{% embed url="https://github.com/ShutdownRepo/httpmethods" %}

{% embed url="https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/03-Testing_for_HTTP_Verb_Tampering" %}
