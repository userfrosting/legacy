---
title: FAQ
---

## :wrench: Install, Config, and Basics for Beginners

#### Q: I use Windows, what's the best way to install Composer?
A: Composer offers a handy installer for windows users. It's available from their [download page](https://getcomposer.org/download/). (When installing, choosing the option to add context menus can be very helpful.)


#### Q: How should I set up UserFrosting for development?
A: You should have two copies of your UserFrosting project. One you keep on your local dev machine, and the public one people will be using. Your local copy will be the place where you do all your coding and testing. Once it's perfect, you upload to the public server. With this method you leave little to no chance of accidentally breaking the site that has active users.

#### Q: What are ".twig" files??
A: These files contain the "View" for your site. All of your HTML sits in these files. Why the "twig" extension, you ask? Because these files hold more than vanilla HTML. You can use Twig code to perform basic programmatic tasks like iterating over arrays, using variables, and more of the basics.

#### Q: How do I put php code into my twig file?
A: You don't. UserFrosting is based on and requires that you follow the Model-View-Controller format. Anything you need to do with PHP is probably best suited for your controller or model, not the view. In the controller you will generate the necessary variables for your twig template to render.

#### Q: But I just want a little bit of php to format some text!
A: Okay, there might already be [an extension to do what you need](http://twig.sensiolabs.org/doc/extensions/index.html#extensions-install). If there isn't a twig extension for what you need, you can write your own filter like so (this would sit in the setupTwig() function within `/models/UserFrosting.php`):
```
$twig = $app->view()->getEnvironment();

$twig->addFilter(new \Twig_SimpleFilter('fancy_quoted', function ($input) {
    return "&ldquo;".$input."&rdquo;";
}));
```
This filter just adds those fancy quotes (the kind that makes copying and pasting code a headache) around your text,

`{{ "Here is my whole quote" | fancy_quoted }}` => `“Here is my whole quote”`

#### Q: I hear logins should never take place over http, but always https. How can I do this with UF?
A: Use a service that provides ssl certificates for an https connection. They are free nowadays with letsencrypt, cloudflare, and other providers.

#### Q: Where do I find "config.js" or "theme.css"?
A: These are [dynamically generated files](https://github.com/userfrosting/UserFrosting/blob/master/public/index.php#L402-L412).
* To edit theme.css: Each Theme/Sub-Theme has their own theme.css file within `templates/themes/THEME_NAME/`.
* To edit config.js: Modify the `configJS()` function within UserFrosting's [BaseController](https://github.com/userfrosting/UserFrosting/blob/master/userfrosting/controllers/BaseController.php#L62-L68)


#### Q: What's the difference between a user's primary group and the other groups they belong to? 
A: The primary group is the main group that the user is assigned to. This controls certain things like their theme. The other groups that a user belongs to are more like "roles" that offer more granular controls of users' abilities.

## :sweat_smile: Things you should have learned from [the tutorials](http://www.userfrosting.com/tutorials/)

#### Q: How do I add a new page?
A: Try [tutorial 1, Creating a New Page](http://www.userfrosting.com/tutorials/lesson-1-new-page/).

#### Q: How do I create a new form that edits some data in the database?
A: Try [tutorial 2, Processing a Form Submission](http://www.userfrosting.com/tutorials/lesson-2-process-form/).

#### Q: How do I use a new database table, custom to my app?
A: Try [tutorial 3, Extending the Data Model](http://www.userfrosting.com/tutorials/lesson-3-data-model/).

## :key: Security

#### Q: "What are some guidelines for maintaining responsible session security with PHP?
**" There's information all over the web and it's about time it all landed in one place!"**

A: See the [stackoverflow question and answer](http://stackoverflow.com/questions/328/php-session-security)

## :mag: Validation Tips

#### Q: How do I validate an array for a value?

A: If your posted data looks something like:
```
$post = [
    'id': 25,
    'token': sometokenhere,
    'an_array_of_options': [
        'title': 'This is my item',
        'desc': 'This item is great'
    ]
];
```
..then two schemas are needed: 1 that validates just the first layer (so, it would only do the `id` field), then another for the `an_array_of_options`. You will need to run the validator twice (one for each schema)

## :diamonds: Code Tips and Tricks

#### Q: How do I enable Twig's dump() function?
A: Twig is configured through Slim\Views\Twig. Options you would normally pass to `new Twig_Environment()` are instead passed to parserOptions in Slim\Views\Twig. To access the parserOptions, you'll need to modify the "Instantiate the Slim application" section of `config-userfrosting.php`
```
$app = new \UserFrosting\UserFrosting([
        'view' =>           $view = new \Slim\Views\Twig(),
        'mode' =>           'dev'   // Set to 'dev' or 'production'
    ]);
   
$view->parserOptions = array(
	'debug' => true
);
$view->parserExtensions = array(     //similar to parserOptions, this line is necessary to enable debug (and dump())
	'Twig_Extension_Debug'
);
```

#### Q: How do I take advantage of the updated_at and created_at columns in my database table?
A: UFModel disables timestamps by default. Your data model most likely extends from this so you'll need to reenable them on a model-by-model basis. To do this, add the following line within the class that defines your model:
`public $timestamps = true;`

#### Q: How do I access settings in my config?
A: `$app->config('parameter')['deeper parameter']` so if you want to find out the "user_id_master", just do `$app->config('user_id_master')`. If you want to find the site's public uri, use `$app->config('uri')['public']`.

#### Q: How do I not use SMTP to send mail?
A: In config-userfrosting.php, change `'mail' => 'smtp'` to `'mail' => ''`

#### Q: How does UserFrosting accomplish _this_?
A: Most of UF's functionality can be understood by traversing through the code. The best way to do this is not to open each file and search through it. Rather, use a text editor that allows you to search for text within a set of folders or search through the code on github. (eg: [How does UF send emails](https://github.com/userfrosting/UserFrosting/search?utf8=%E2%9C%93&q=send+email)?) This works really well because both UF and the many vendors that provide code have included either helpful comments or named the functions/classes in a manner that just makes sense.

## :suspect: Why is _this_ happening?

#### When I submit a form it takes me to a blank screen.
What is probably happening: You are submitting a form without including the CSRF token. Most of the time you'll be submitting forms via AJAX. To do this, you'll generally want to use the [ufFormSubmit function](https://github.com/userfrosting/UserFrosting/blob/c292d0e54710852a4ecc2374db0a526850128955/public/js/userfrosting.js#L79-L138) to get your form ready. See it below
```
// Process a UserFrosting form, displaying messages from the message stream and executing specified callbacks
function ufFormSubmit(formElement, validators, msgElement, successCallback, msgCallback, beforeSubmitCallback)
```

A great and very simple example of this function in use is on the [Login page.](https://github.com/userfrosting/UserFrosting/blob/master/userfrosting/templates/themes/default/account/login.twig#L56-L64)

**Q: I'm using the ufFormSubmit function but it's still not working. What now?**

A: There are two things to check at this point. Go to the page in your browser where the form should be and view the source. Go to the lines where ufFormSubmit sits and check that the validators and any other server-generated content rendered correctly. At the same time, you'll want to check your browser's console for any javascript errors. If there are other javascript errors on the page you may not be able to use ufFormSubmit even if your implementation is correct. If that's the case, you'll need to resolve those other javascript errors first.

Lastly, you'll want to monitor what happens when you POST your form by using your browser's network monitor. Check the response code and the parameters sent. Is the CSRF token present?


## :eyes: I have a different/unknown error. What do I do?

#### Basic error mitigation:
With time, most errors can be hammered out with a bit of investigation and elbow grease. Here are the steps I take when investigating a random error that I have no idea about:

1. Check the error log, read through the entire stack trace. Where does the error occur? 
2. What changed between when it worked and now?
3. Go through the code, following the course of the stack trace in reverse. Do the values passed to my functions and classes reflect what I expect? Double check the stack trace, you should see much of the calculated values that get passed into functions.

#### There's something else I'm confused about.
Ever heard of GIMF? If not, google it. Seriously though, much of UserFrosting's environment is built on code provided by 3rd party vendors and it's your due diligence to learn from their documentation. So, if you are coming up with no google results for "userfrosting here's my issue" you might just try "here's my issue". You can also double check in the error log for which vendor the error occurs with and include this in your search (eg: "laravel querybuilder error").

Another great place to look is in [UserFrosting's chat history](https://chat.userfrosting.com/channel/support).  You can also look in our old chat room, at `https://gitter.im/userfrosting/UserFrosting`.
