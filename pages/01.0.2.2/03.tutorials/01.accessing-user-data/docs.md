---
title: Accessing user data
taxonomy:
    category: docs
---

If you are trying to access the current user's info on the server-side (in PHP), you can directly access the object. For example, 

```
echo $loggedInUser->user_id;
echo $loggedInUser->email;
```

If you need to access it the user's information from the client-side, i.e. in Javascript somewhere, you can get this information from `api/load_current_user.php`.  For convenience, there is a Javascript function `loadCurrentUser()`, that will load information for the currently logged-in user.  It can be found in `js/userfrosting.js`.  This function returns a JSON object containing the user fields.
