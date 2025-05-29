# Detecting and Enumerating IDOR

#### URL paramters and APIs&#x20;

Look for parameters that appear to be direct object references. for example:

```
?uid=1 or ?filename=file_1.pdf
```

These parameters can then be incremented or fuzzed.

#### AJAX Calls

Look for unused parameters or API's in the front end code in the form of JavaScript AJAX calls. these may have been accidentally placed or left here and by modifying the HTTP request, we can possibly find an IDOR vulnerability.&#x20;

```
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

#### Hashing/Encoding

For parameter input, the application may not always use a plain text value and instead could encode or hash the reference. If this is found the value could possibly be deoded, changed, and then re encoded to attempt to access sensitive data.&#x20;

We can also check the front end code to detect if a reference is being hashed:

```
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```

This can make it easy for us to calculate other reference values we would like to fuzz. If we cannot find the hash algorithm in an ajax call in the front end code, then we can use some hash identifier tools to attempt to learn the hash type.

#### User Roles

By registering multiple users and comparing their HTTP requests and object references, this may allow us to better understand how the URL parameters and object references are formatted, so we can use this to our advantage when looking for IDOR.

```
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```

