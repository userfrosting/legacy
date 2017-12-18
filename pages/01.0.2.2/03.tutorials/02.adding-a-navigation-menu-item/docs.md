---
title: Adding a navigation menu item
taxonomy:
    category: docs
---

There is currently no frontend interface for adding/changing the navigation menu items.  Therefore, you must modify the database directly.

## Adding the new menu item

Navigation menu items are stored in the `nav` table in the database.  To add a new menu item, you need to enter the following information:

- `menu`: The menu you'd like to place your item in.  Currently, can only be `left`, `left-sub`, or `top-main-sub` (the dropdown menu in the upper right-hand corner of the screen).
- `page`: The url, relative to the site root, of the link.  Use `#` if you don't want it to link to anything.
- `name`: The name of the menu item.  Can be simple text, or use a placeholder (e.g. `#USERNAME#` to render dynamic content).
- `position`: The relative position of the menu item in the menu.  These numbers do not necessarily need to be consecutive.
- `class-name`: A unique identifier for this menu item, that UF will use to reference in formatting.  Is also used by the `renderMenu` function to determine which menu item is currently active.
- `icon`: The icon class for this menu item.  Can be any CSS icon class, e.g. from FontAwesome.
- `parent_id`: The id of the parent menu item.  For top-level menu items, set this to `0`.

## Associating the new item with groups

__Once you add a new menu item, you must associate it with one or more groups__.  You need to tell UF which groups will see your new menu item.  To do this, add an entry to `nav_group_matches`.  Specify the `menu_id` of the new menu item (from the `id` column of the `nav` table), along with the `group_id` of the group you'd like to associate with this menu item.  You can associate the same menu item with more than one group.

## Setting up the navigation menu in the corresponding page
Once you add the new menu item, you will want it to be highlighted when you load the corresponding page.  To do this, the parameter called in `renderMenu` must be the same as the `class-name` specified for the menu item.  For example, the following is the beginning of `account/dashboard.php`:

```
<!DOCTYPE html>
<html lang="en">
  <?php
    echo renderAccountPageHeader(array("#SITE_ROOT#" => SITE_ROOT, "#SITE_TITLE#" => SITE_TITLE, "#PAGE_TITLE#" => "Dashboard"));
  ?>

  <body>

    <div id="wrapper">

      <!-- Sidebar -->
        <?php
          echo renderMenu("dashboard");
        ?>
```
Notice that parameter `"dashboard"` in `echo renderMenu("dashboard");` is the `class-name` specified for the dashboard menu item in the `nav` table.

## Access control

After you create a new page, be sure to grant access to the new page by logging in as the root user, and going to "Site Settings->Authorization->Page-level authorization".  You should see your new page listed.  Check the "private" box to make this page visible only to logged-in users.  Then, check the appropriate boxes for the groups you want to permit access of this page.

If you do not see your new page listed at this point, check that it is in one of the directories in the `filelist` table.
