# XXE - XML External Entity Injection

XXE vulnerabilities are when XML data is sent from the front end to the back end for processing, without properly sanitizing the input or safely parsing it, which may allow us to exploit XML features.&#x20;

### XML - Extensible Markup Language&#x20;

Language that is formatted similar to html that is designed for transfer and storage of data and documents. This is frequently used for holding application configurations but can be used for many other things as well.&#x20;

Example XML:

```
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <time>10:00 am UTC</time>
  <sender>john@inlanefreight.com</sender>
  <recipients>
    <to>HR@inlanefreight.com</to>
    <cc>
        <to>billing@inlanefreight.com</to>
        <to>payslips@inlanefreight.com</to>
    </cc>
  </recipients>
  <body>
  Hello,
      Kindly share with me the invoice for the payment made on January 1, 2022.
  Regards,
  John
  </body> 
</email>
```

### XML DTD (Document Type Definition)

Document which is used to validate the structure of an XML document that is compared against the DTD, can either be included in the XML doc itself or an external file.&#x20;

example:&#x20;

```
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  <!ELEMENT recipients (to, cc?)>
  <!ELEMENT cc (to*)>
  <!ELEMENT date (#PCDATA)>
  <!ELEMENT time (#PCDATA)>
  <!ELEMENT sender (#PCDATA)>
  <!ELEMENT to  (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
]>
```

### XML Entities

Custom variables in XML DTD's which allow refactoring of variables and reduce redundant data in the XML. This is done by using the ENTITY keyword.

example:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

Entities are then referenced in between an ampersand and semicolon:

```
&company!
```

External XML Entities can be referenced with the SYSTEM keyword ( or also PUBLIC) to load external resources. This can be used to reference a file on the back end server, which could then be disclosed to us when the entity is referenced.&#x20;





### Links and Resources

{% embed url="https://book.hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html" %}

{% embed url="https://portswigger.net/web-security/xxe" %}
