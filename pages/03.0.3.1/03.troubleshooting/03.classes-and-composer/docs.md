---
title: Classes and Composer
---

## I created a new class, but UserFrosting doesn't seem to recognize it.

You need to [install Composer](http://getcomposer.org), PHP's dependency manager. In addition to automatically loading external packages, Composer also autoloads the class files for your project. When you add a new class, you must run `composer update` in the `userfrosting` directory (the same directory that contains `composer.json`). Make sure that you have installed Composer [globally](https://getcomposer.org/doc/00-intro.md#globally).

## Composer gives me errors when I run `composer update`.

When this happens, you might get an error message complaining about a `.git` directory missing in `vendor`. To solve this problem, simply delete the `vendor/alexweissman` directory and re-run `composer update`.
