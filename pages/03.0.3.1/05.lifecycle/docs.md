---
title: Application lifecycle
---

The lifecycle of the UserFrosting application extends the standard lifecycle of a Slim application.  Here is the breakdown:

1. PHP `session_start` is called (`config-userfrosting.php`)
2. The Slim application is instantiated (`config-userfrosting.php`)
3. Set database, path, and URL config information (`config-userfrosting.php`)
4. Set database table aliases (`initialize.php`)
5. Initialize database table columns and names (`initialize.php`)
6. Load site settings, either default or from the database (`initialize.php`)
7. Create page schema (`initialize.php`)
8. Setup a guest user and environment, including translator, message stream, and error handling (`setupGuestEnvironment` in `UserFrosting.php`)
9. Setup Twig custom functions and site setting variable (`setupTwig` in `UserFrosting.php`)
10. Initialize plugins (`initialize.php`) 
11. **`slim.before` hook is called**
12. Set current user to logged in user, if one is available (`middleware/usersession/UserSession.php`)
13. Setup logged-in user environment, including translator, message stream, error handling, and Twig user variables (setupAuthenticatedEnvironment in `UserFrosting.php`)
14. **Slim routes the request**
