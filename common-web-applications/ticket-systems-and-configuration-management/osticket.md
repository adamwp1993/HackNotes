# osTicket

Open source support ticket program written in PHP with a mySQL backend.&#x20;

### Discovery/Footprinting/Enumeration

Besides the obvious of looking at the page and seeing it, can also be identified by:&#x20;

Footer with powered by and the words `Support Ticket System`.&#x20;

cookie named OSTSESSID that is set when visiting the page&#x20;

### Options for attacking osTicket&#x20;

1. Social Engineering (If in scope)
2. Known vulnerabilties i.e [https://nvd.nist.gov/vuln/detail/CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881)
3. Obtaining an email address with the companies domain for use in another application
   1. [https://www.youtube.com/watch?v=gbs43E71mFM](https://www.youtube.com/watch?v=gbs43E71mFM)
   2. [https://medium.com/intigriti/how-i-hacked-hundreds-of-companies-through-their-helpdesk-b7680ddc2d4c](https://medium.com/intigriti/how-i-hacked-hundreds-of-companies-through-their-helpdesk-b7680ddc2d4c)

### Creating email with target orgs domain via osTicket

Submitting a ticket provides an email address which can be used to make updates to the ticket:&#x20;

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Any emails sent to this address will be added to the ticket, so we could use this to register with another application, and check the ticket for the incoming registration emails.&#x20;

### Sensitive data exposure&#x20;

If we are able to find compromised credentials anywhere, such as using dehashed, we should attempt to use them to access the application. you may find interesting information.&#x20;

for example:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

> many of these types of applications may contain an address book with usernames/email addresses. we should always attempt to collect these.&#x20;
