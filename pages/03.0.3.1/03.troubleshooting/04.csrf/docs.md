---
title: CSRF tokens
---

On a regular basis people are faced with an unexplained server error 400 when trying to `POST` data in UF. More often than not this is due to a missing csrf token.

Here is a non-exhaustive list of ways to include a csrf token in your post request.

## 1. ufFormSubmit() Function

UF comes with a build in function that appends the csrf token to submitted data. This is discussed in [lesson 2](/0.3.1/tutorials/lesson-2-process-form/).

An example of how to apply the `ufFormSubmit()` function, taken from said tutorial, is shown here:

```
    {% block page_scripts %}
        <script>
        $(document).ready(function() { 
            // Load the validator rules for this form
            var validators = {{validators | raw}};
            ufFormSubmit(
              $("form[name='titles']"),
              validators,
              $("#userfrosting-alerts"),
              function(data, statusText, jqXHR) {
                  // Reload the page on success
                  window.location.reload(true);   
              }
            );
        });
    </script>
    {% endblock %}
```

A common problem is that a developer has some Javascript errors on a page that calls `ufFormSubmit`.  Depending on where the error is, this can cause your browser to quietly submit your form using the "traditional method" rather than invoking `ufFormSubmit`.  Since `ufFormSubmit` is also responsible for appending the CSRF token to the request, this can lead to unexpected `400` errors.

To check for these "silent" Javascript errors, you should open up your browser's console.  If you don't know how to use your browser's console, now is a good time to Google it!

## 2. Appending the CSRF token manually in AJAX

Sometimes, in particular when you are using third party widgets, it might not be possible for you to use the `ufFormSubmit()` function. One way to proceed in such cases is to append the token to the array or object your are handing over to AJAX. In fact, that is what the `ufFormSubmit()` function [does](https://github.com/userfrosting/UserFrosting/blob/3.1/public/js/userfrosting.js#L106-L115). An example is shown here:
```
function(item) {
    var csrf_token = $("meta[name=csrf_token]").attr("content");

    item['csrf_token'] = csrf_token;

    return $.ajax({
      type: "POST",
      url: "{{site.uri.public}}/your/controller",
      data: item,
      dataType: "json" 
    });                    
}
```

## 3. Submitting the token in a hidden field

One of the features (or in this case limitation) of AJAX is that it passes the data to the url without leaving or refreshing the page. The response to the request is sent to the AJAX callback. If you want to redirect to a different page and pass data, you can add the csrf token to a hidden field:

```
<input type="hidden" name="{{csrf_key}}" value="{{csrf_token}}">
```
