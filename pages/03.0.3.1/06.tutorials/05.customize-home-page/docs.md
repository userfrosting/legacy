---
title: Customizing the home page
---

The default script has a login button that takes your guests (non logged in visitors) to a login page.

In some home page designs the user/password fields need to be in the home page. if you need to do that then you also need to have the validators available inside the javascript that you should also be rendering in the home page.

One way to do it is to adjust `AccountController.php` when you render the default page with something like this:

```
public function pageHome()
{
    
      $schema = new \Fortress\RequestSchema($this->_app->config('schema.path') . "/forms/login.json");
     $this->_app->jsValidator->setSchema($schema);

    $this->_app->render('home.twig',  [
       'validators' => $this->_app->jsValidator->rules()
    ]);
}
```
