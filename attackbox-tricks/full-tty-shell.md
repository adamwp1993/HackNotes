# Full TTY shell

Sometimes you may get a shell without full functionality when executing a payload.

We can upgrade the shell to full TTY:

`python -c 'import pty; pty.spawn("/bin/bash")'`

`SHELL=/bin/bash script -q /dev/null`

Additional links:

{% embed url="https://0xffsec.com/handbook/shells/full-tty/" %}

{% embed url="https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#generating-reverse-shell-commands" %}
