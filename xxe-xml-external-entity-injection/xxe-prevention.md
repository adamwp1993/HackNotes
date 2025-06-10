# XXE prevention

### Avoid Outdated Components

XXE is typically caused by outdated XML libraries. Its highly important to keep these up to date.&#x20;

You can find a list of all known vulnerable XML libraries and functions at the following link:

{% embed url="https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html#php" %}

Its important ot update any components that parse XML input, including API libraries (i.e SOAP), document or file processors, etc. Using the latest XML libraries and web development components can greatly reduce XXE vulnerabilites (as well as other vulnerabilities)&#x20;

### Safe XML Configuration

* Disable referencing custom `Document Type Definitions (DTDs)`
* Disable referencing `External XML Entities`
* Disable `Parameter Entity` processing
* Disable displaying runtime errors in web servers to web app users
* Consider using JSON or YAML instead
* Utilize a web application firewall&#x20;
* Disable support for `XInclude`
* Prevent `Entity Reference Loop`

