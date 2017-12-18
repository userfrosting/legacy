---
title: Adding a new user field
---

This tutorial describes how user fields are added and to a certain extent the general idea behind the different steps.

## 1. Adding a new column to the table in the database

As all user fields are stored in a table, I figured it might be a good idea to start by adding a new column to the right table in the SQL database. All user information such as passwords etc. are stored in the table `user` with the corresponding prefix. I left the standard settings so the table is called uf_users in my case.

I want to add a column called "comment", which should be text and by default empty. Adding columns to databases is beyond the scope of this tutorial. However, the SQL code I used is the following:

```
ALTER TABLE `uf_user` ADD `comment` TEXT NULL DEFAULT NULL AFTER `password`;
```

## 2. Registering the new column

Following [tutorial 3](/0.3.1/tutorials/lesson-3-data-model/) on the official userfrosting page, all tables with their corresponding columns have to be registered in `userfrosting/initialize.php`.

In the `initialize.php`, I found the bit of code that registers the user table and added my "comment" column:

```
$table_user = new \UserFrosting\DatabaseTable($app->config('db')['db_prefix'] . "user", [
    "user_name",
    "display_name",
    "email",
    "title",
    "locale",
    "primary_group_id",
    "secret_token",
    "flag_verified",
    "flag_enabled",
    "flag_password_reset",
    "created_at",
    "updated_at",
    "password",
    "comment"
]);
```

## 3. Interaction with the new column

After registering the column, you are already able to access the comment column and, by way of example, echo the comment for one specific user:

`echo user::where('id', $this->_app->user->id)->select('comment')->first();`

However, UF comes with a variety of public functions in the UserController class which can be found in `/userfrosting/controllers/UserController.php`. What each public function does is described in comments in the `UserController.php`. You can decide what users or administrators should be able to do with that new column.

I want the individual user and root to view and edit comments. In order to do that, I edited the line where the `fields` variable is set in the public function pageUser and formUserEdit to look like that:

`$fields = ['display_name', 'email', 'title', 'locale', 'groups', 'primary_group_id', 'comment'];`

To actually have the column show somewhere we also need to include the new column in the correct twig files. This part is a bit tricky because the public function pageUser renders the user-info.twig file wich in turn includes some other twig files. The one you acutally want to edit is the `/userfrosting/templates/themes/default/components/common/user-info-form.twig`

In to order to make the comment column show, I added the following:

```
{% if 'comment' not in fields.hidden %}             
<div class="col-sm-6">
    <div class="form-group ">
        <label>Comment</label>
        <div class="input-group">
            <span class="input-group-addon"><i class="fa fa-edit"></i></span>
            <input type="text" class="form-control" name="comment" autocomplete="off" value="{{target_user.comment}}" placeholder="" {% if 'title' in fields.disabled %}disabled{% endif %}>
        </div>
    </div>
</div>
{% endif %}
```

## 4. Validating the submitted input

Input validation in UF is handled by [Fortress](https://github.com/userfrosting/fortress) and was described further in [tutorial 2](/0.3.1/tutorials/lesson-2-process-form/) on the official userfrosting page.

I added to the `/userfrosting/schema/forms/user-update.json` the following lines:

```
    "comment" : {
        "validators" : {
            "length" : {
                "max" : 200,
                "message" : "Comment has to be shorter than 200 characters."
            }
        }
    },
```
If you now log in as root, go to users and select any given user, you will be able to view and edit the "comment" field.

## 5. Adding the new column as a field to the account settings

To allow the user to view and edit the comment column in his account settings, you have to do two things. Firstly, you need to add a field to the `/userfrosting/templates/themes/default/account/account-settings.twig` file.

In my case, I added the following:

```
    {% if checkAccess('update_account_setting', {('user'): user, ('property'): 'comment'}) %}
    <div class="form-group">
        <label for="input_display_comment" class="col-sm-4 control-label">Comment</label>
        <div class="col-sm-8">
            <input type='text' id="input_display_comment" class="form-control" name="comment" value='{{user.comment}}'>
            <p class="help-block">A comment</p>
        </div>
    </div>
    {% endif %}
```
As you can see, it checks whether the user has the permission to alter the "comment" column. The permissions are stored in authorize_group table with the corresponding prefix. You can edit it there manually or you can log in as root and go to "Configuration" -> "Groups" -> Click "Actions" on the default primary group (or whatever you want it to be) -> "Authorization rules" ->  Click "Actions" on the `update_account_settings` hook -> "Edit rule" -> change it to:

```
"equals(self.id, user.id)&&in(property,["email","locale","password","comment"])"
```

And you are set. The root user as well as the individual user can now view and edit the new "comment" column.
