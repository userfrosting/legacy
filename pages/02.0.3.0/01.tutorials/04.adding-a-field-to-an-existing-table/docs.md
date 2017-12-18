---
title: Adding a field to an existing table
---

You've installed User Frosting and now you want to add a field to an existing table?  Let's add a description to groups table.  Here is how:

## Step 1

Using phpmyadmin **add new column** at the end of the uf_group table, call it "description"

## Step 2

Go the the file **group-update.json** and add the validator for the new column:

```
    "description" : {
        "validators" : {
            "length" : {
                "min" : 1,
                "max" : 140
            }
        }
    },
```

## Step 3

Go to the file **group-info-form.html** and add this to the form so that you can edit the content of the field from UserFrosting settings:

```
    {% if 'description' not in fields.hidden %}        
      <div class="col-sm-6">
          <div class="form-group ">
              <label>description</label>
              <div class="input-group">
                  <span class="input-group-addon">/</span>
                  <input type="text" class="form-control" name="description" autocomplete="off" value="{{group.description}}" placeholder="neighborhood description" {% if 'description' in fields.disabled %}disabled{% endif %}>
              </div>
          </div>
      </div>
    {% endif %}
```

## Step 4

Go to **initialize.php** and look for the row where `$table_group` lists the column names and add "description" to it like this:

```
$table_group = new \UserFrosting\DatabaseTable($app->config('db')['db_prefix'] . "group", [
    "name",
    "is_default",
    "can_delete",
    "theme",
    "landing_page",
    "new_user_title",
    "icon",
    "description"
]);
```

## Step 5

That's it...now if you want to **render the description** of the primary group that the user belongs to using twig you can just use:

```
{{ user.primary_group.name }}
{{ user.primary_group.description }}
```
