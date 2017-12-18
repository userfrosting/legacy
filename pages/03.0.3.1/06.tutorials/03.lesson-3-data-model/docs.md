---
title: Lesson 3: Extending the Data Model
---

Ok, so you've patiently completed lessons [1](/0.3.1/tutorials/lesson-1-new-page) and [2](/0.3.1/tutorials/lesson-2-process-form), and hopefully you're starting to understand the basic concepts of the [model-view-controller (MVC)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) architecture. We've dived a bit into Twig, which handles the "view" part of "model-view-controller".

We've also talked a lot about controllers, using them to send data **to** the view (to render a "page"), and also capturing data **from** `POST` requests to perform some operations on our data model (i.e., deleting a user, updating the name of a group, etc).

What we _haven't_ really talked about yet is the **model**. We've interacted with it briefly, though. In lesson 1 we used `$app->user`, which represents the currently logged-in user with an instance of the `User` class, to get a list of groups for that user. In lesson 2, we used the Eloquent query builder to obtain collections of `User` and `Group` objects. We then modified these objects and used the `->save()` method to update their information in the database.

But of course, your application probably consists of more than just users and groups! After all, your users are probably supposed to _do_ something in your system, and interact with other sorts of data. So, how do we extend the UserFrosting data model to include, for example, `Transaction`s or `Event`s or `Cat`s or `NuclearMissile`s?

In this tutorial, we will create a new type of data object, `StaffEvent`, which will represent events such as meetings, parties, luncheons, etc. We will design our model so that our users can be assigned to these events.

## Setting up the Database Tables

First, we need to represent events in our database. To do this, we will manually create a table, `staff_event` (or `uf_staff_event` if you want to stick with the default table prefix), where each row will represent a unique event and its properties. We will define the following columns for this table as well:

- `id` (int, primary key, autoincrement, unique, not null)
- `date` (datetime, not null)
- `location` (text)
- `description` (text)
- `created_at` (timestamp)
- `updated_at` (timestamp)

**In UserFrosting, every table must have a primary key called `id`!** UserFrosting's base classes for loading and manipulating the database will not work otherwise.

Hopefully, you know how to create a database table in whichever system you are using. If not, please consult your system's documentation.

A user can be assigned to more than one event, and events can have more than one user. Thus, this constitutes a **many-to-many** relationship. To model a many-to-many relationship, we need an additional table called the **link table**. We will name the table using the convention of placing an underscore between the names of the tables to be linked. Thus, we will call this table `staff_event_user` (or `uf_staff_event_user`, again if you want to stick with the default table prefix). This table will have three columns:

- `id` (int, primary key, autoincrement, unique, not null)
- `user_id` (int, not null)
- `staff_event_id` (int, not null)

Great! We're now ready to register our tables in UserFrosting.

## Registering the New Tables

To work with our new tables in UserFrosting, we need two pieces of information: the names of the tables, and the names of their columns. We will register this information all in one place in the code, in `userfrosting/initialize.php`.

To do this, we will first create a new instance of the `DatabaseTable` class:

```
$table_staff_event = new \UserFrosting\DatabaseTable("staff_event", [
    "date",
    "location",
    "description",
    "created_at",
    "updated_at"
]);
```

Notice that the first argument is the name of the table. If you are using table prefixes, you can either hardcode the prefix as part of your table name here (e.g., `"uf_staff_event"`), or dynamically grab it from the configuration (`$app->config('db')['db_prefix'] . "staff_event"`).

The second argument is an array containing the names of the columns. This is not strictly necessary, but it is useful if we want to allow creating new objects from an array of user-supplied data. To protect our database from users trying to set any arbitrary column in the record during an object's creation, Eloquent requires us to specify which columns can be mass-assigned from an array (such as the array of POSTed data). See Eloquent's [mass assignment](https://laravel.com/docs/5.4/eloquent#mass-assignment) documentation for more details.

In practice, to satisfy Eloquent, we should just specify all table columns in this list. For most situations we will control which fields can be set via a `RequestSchema`, and perhaps using conditions in a call to `checkAccess`. The latter is typically done when we want to fine-tune different users' permissions on the same resource. Thus, we do not need to rely on Eloquent to do any additional whitelisting.

Once we define our `DatabaseTable` object, we can register it with UserFrosting:

```
\UserFrosting\Database::setSchemaTable("staff_event", $table_staff_event);
```

The first parameter for `setSchemaTable` is simply a **handle** that we will use to refer to the table throughout the code. The convention is to give it the same name as the table itself, but you may name it however you like. The second parameter is the `DatabaseTable` object that we just created, which will be registered with that handle.

We will do the same thing for the link table, but with one minor difference - we won't bother specifying the column names. This is because we usually will not model the rows in our link table as data objects, like we will do with events. Instead, [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) for the link table will be managed through the data objects that it links.

All we will do for the link table, then, is register the table name:

```
$table_staff_event_user = new \UserFrosting\DatabaseTable("staff_event_user");
\UserFrosting\Database::setSchemaTable("staff_event_user", $table_staff_event_user);
```

Great, now we have an organized, sane way to access information **about** our tables (but not the data in those tables, yet).

## Modeling the Event Object

You may be used to simply writing and executing a SQL query every time you need to interact with the database, or perhaps writing functions that encapsulate this behavior. In UserFrosting, we will go one step further and use **objects** to encapsulate all the information about a particular row in the database, along with the functionality to modify and store it in the database. This gives us a uniform, consistent way to represent and manipulate data without repetitive code.

If you look in the `userfrosting/models` directory, you will notice classes called `Group` and `User`. These are the classes used to model groups and users, and they both inherit the basic functionality of their base class, `UFModel`, which itself inherits from Eloquent's [`Model`](http://laravel.com/docs/5.4/eloquent#basic-usage) class. If you don't know what "inherit" means, now is a good time to [learn a little about object-oriented programming](https://en.wikipedia.org/wiki/Class-based_programming#Inheritance).

We will create a new class, in a new file, called `StaffEvent`, which will also inherit from UFModel:

**userfrosting/models/database/StaffEvent.php**

```
<?php

namespace UserFrosting;

use \Illuminate\Database\Capsule\Manager as Capsule;

class StaffEvent extends UFModel
{
    protected static $_table_id = "staff_event"; 
}
```

**Note that since we are creating a new class, you must have Composer installed and run `composer update` to have your new class autoloaded.** See [here](/0.3.1/navigating/#composer) for more information.

Notice that we set a static `$_table_id` property in our class. UserFrosting will use this to look up the `DatabaseTable` object containing the information about our table based on the handle that we assigned it in `initialize.php`.

With just these few lines, we can create new `StaffEvent` objects easily:

```
$new_event = new StaffEvent([
    "date" => "2015-12-24 14:00:00",
    "location" => "Room 101",
    "description" => "Mandatory Christmas party for all employees!"
]);
```

And we can store the new event to the database:

```
$new_event->save();
$id = $new_event->id;
```

Notice how all the SQL queries are taken care of for us. Event objects can also be used to update properties in the table:

```
$new_event->location = "Torture Chamber Alpha";
$new_event->save();
```

And we can even delete events from the database:

```
$new_event->delete();
```

Ok, so that covers the "C", "U", and "D" in CRUD. But what about the "R" (read)?

Well, it turns out that our new class `StaffEvent` is also a query builder! Want to get a list of all staff events in December, sorted by date? No problem:

```
$december_events = StaffEvent::whereBetween('date', [
    "2015-12-01 00:00:00",
    "2016-01-01 00:00:00"
])
->get();
```

## Modeling Relationships

That covers the basic CRUD operations for events. But, we still haven't modeled the relationships **between** events and users! An event can have multiple users, and a user can have multiple events. It would be nice to be able to take a given `StaffEvent` object, and get an array of all users who are assigned to that event. Or, we might want to take a specific `User` object, and get an array of all events associated with that user. To do this, we will modify the `StaffEvent` and `User` objects.

First, we have to ask ourselves some questions. When we load a particular `StaffEvent` from the database, do we want to immediately load all of its assigned users? Or, should we wait until we actually need them?

The second method is commonly called **lazy loading**, and is the method that I prefer. Why? Because it saves us unnecessary querying. If in a given request, we don't care about the users assigned to an event, then we won't waste time querying the database for that information.

To implement lazy loading, all we need to do is implement a method in `StaffEvent` called `users`:

```
public function users()
{
    $link_table = Database::getSchemaTable('staff_event_user')->name;
    return $this->belongsToMany('UserFrosting\User', $link_table);
}
```

This is Eloquent's way of providing access to a many-to-many relationship. All we need to do is get the name of the link table from our database schema, and then call `belongsToMany` on the current `StaffEvent` object. The first argument is the name of the class that models our User object, and the second argument is the name of our link table. This method will then return a `Relation`, which we can further query with Eloquent's [other query-building methods](https://laravel.com/docs/5.4/queries) if desired. Alternatively, we can call the method without the parentheses, directly accessing the collection of User objects for a particular `StaffEvent` as a "dynamic property".

```
// Fetch event 1
$event = StaffEvent::find(1);

// Load users for event 1.  This returns a Collection, which we can iterate over just like an array.
$users = $event->users;
foreach ($users as $user) {
    echo $user->display_name . PHP_EOL;
}
```

### Accessing Relations in Twig

If we try to access the `users` for a particular `StaffEvent` in PHP, we can just call `$my_event->users`. In Twig, the corresponding syntax would be `{{my_event.users}}`. Unfortunately, if you were to do something like `$my_event = StaffEvent::find(1);` and pass `$my_event` into Twig, you'd find that the Twig variable `my_event.users` is always empty (even if you know that your event is associated with some users).

The problem is that Twig decides whether a property (like `users`) exists for an object using the `isset` method. And, it turns out that if we call `isset` on a dynamic property (like `users`) that hasn't been accessed yet, Eloquent will return false! To get around this, we need to let Twig know that the property does in fact, "exist" - it just hasn't been loaded yet. To do this we can implement our `StaffEvent` model's [magic `__isset` method](http://php.net/manual/en/language.oop5.magic.php):

```
public function __isset($name)
{
    if (in_array($name, [
        "users"
    ])
        return true;
    else
        return parent::__isset($name);
}
```

Now, when Twig calls `isset()` on `$my_event->users`, it will return true. Thus, Twig will know that the dynamic property `users` exists, and when it tries to access this property for the first time, Eloquent will automatically perform the necessary query.

### Refreshing Related Data

Ok, but what if I modify a related `user` after I've loaded the `users` property? Fortunately, Eloquent provides the `fresh` method as an easy way for us to reload an object's data from the database, along with any relationships we care about:

```
$my_event = $my_event->fresh(['users']);
```

Thus, we can do things like:

```
// Fetch event 1
$event = StaffEvent::find(1);

// Load users for event 1
$users = $event->users;

// Change one of those users
$users[0]->title = "The New Kid in Town";
$users[0]->save();

// Refresh the event, updating the user info
$event = $event->fresh(['users']));
```

### Modifying Relationships

Ok, so we've seen how to get the related users for an event, but how do we add and remove related users? To do this, we can use the `attach` and `detach` methods:

```
// Fetch user 1 into $user
$user = User::find(1);

// Fetch event 1
$event = StaffEvent::find(1);

// Associate $user with event 1
$event->users()->attach($user->id);

// De-associate $user with event 1
$event->users()->detach($user->id);
```

And if we wanted to use the current user logged into our app instead?

```
// Fetch event 1
$event = StaffEvent::find(1);

// Associate current user with event 1
$event->users()->attach($app->user->id);

// De-associate current user with event 1
$event->users()->detach($app->user->id);
```

We might want to automatically delete any relationships when we delete a particular object. So, we will overload the `delete` method in `StaffEvent`:

```
public function delete()
{        
    // Remove all user associations
    $this->users()->detach();
    
    // Delete the event itself        
    $result = parent::delete();
    
    return $result;
}
```

And that's it! Our `StaffEvent` data is now a full-fledged, relational data model that can keep track of which users are assigned to it in a sane manner. We can make the same modifications to `User` to create methods like `events()`, etc as necessary.

Combining what you've learned here with what you learned in Lesson 2, you should now be able to implement a controller with all the routes you need to create, update, delete, and view/list events, as well as assign users to events.
