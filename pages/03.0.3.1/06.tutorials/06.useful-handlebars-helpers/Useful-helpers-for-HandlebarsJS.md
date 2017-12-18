```
/**
 * format a string as a 7- or 10-digit phone number using regular expressions.
 * usage: {{phoneNumberFormat phone_number}}
 */
Handlebars.registerHelper("phoneNumberFormat", function(phoneNumber) {  
    // Strip out all non-digit characters
    phoneNumber = phoneNumber.replace(/[^0-9]/g, '');
    // Format as a 7- or 10- digit phone number
    var len = phoneNumber.length;
    if (len == 7)
        phoneNumber = phoneNumber.replace(/([0-9]{3})([0-9]{4})/g, '$1-$2');
    else if (len == 10)
        phoneNumber = phoneNumber.replace(/([0-9]{3})([0-9]{3})([0-9]{4})/g, '($1) $2-$3'); 
    return phoneNumber; 
});
```