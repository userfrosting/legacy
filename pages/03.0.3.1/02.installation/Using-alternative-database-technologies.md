## Change connection driver

Change the connection driver in [initialize.php](https://github.com/userfrosting/UserFrosting/blob/master/userfrosting/initialize.php#L42-L51).

```
$capsule->addConnection([
    'driver' => 'sqlsrv',
    'host' => $dbx['db_host'],
    'database' => $dbx['db_name'],
    'username' => $dbx['db_user'],
    'password' => $dbx['db_pass'],
    'charset' => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix' => ''
]);
```

## Create tables

Manually create the tables, which would normally be created by the installer:

[userfrosting/models/database/Database.php](https://github.com/userfrosting/UserFrosting/blob/master/userfrosting/models/database/Database.php#L167-L268)

## Override installation status

In the `configuration` table, set the value for `install_status` to `pending`.

You should now be able to visit `/public/install` and set up the master account.