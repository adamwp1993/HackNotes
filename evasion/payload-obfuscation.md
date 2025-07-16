# Payload Obfuscation

the following web shell is flagged as malicious by most security vendors: [https://github.com/SecurityRiskAdvisors/cmd.jsp](https://github.com/SecurityRiskAdvisors/cmd.jsp)



A simple change such as changing:

Code: java

```java
FileOutputStream(f);stream.write(m);o="Uploaded:
```

to:

Code: java

```java
FileOutputStream(f);stream.write(m);o="uPlOaDeD:
```

results in 0/58 security vendors flagging the `cmd.jsp` file as malicious at the time of writing.
