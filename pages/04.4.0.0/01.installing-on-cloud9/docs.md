---
title: Installing on Cloud9
---

# Setting up your workspace
The first thing you have to do, is create a workspace for Userfrosting. Press the giant "+" to get started.

![Create new](http://p1.pichost.me/640/79/2044415.png)

Next you'll have to give it a name, and chose the "PHP, Apache & MySQL" template. Do **not** use the clone from git function, as it can often act weird, and not clone properly.

![](http://p1.pichost.me/640/79/2044416.png)

# Installing/Updating things
So, by default, C9 has PHP `v5.5.9`, and Node `v4.6.1`, both of which are not the best for UF.

## PHP
To update PHP, follow [this](http://askubuntu.com/a/760926) handy dandy Stack Overflow guide. ;)
It'll show you how to replace `v5.5.9` with `v7.x`.

## Node
To update Node, use:
```
nvm install 5.5.0
nvm use 5.5.0
nvm alias default v5.5.0
```

## PHP MySQL Package
You'll need to install the PHP MySQL package too, for that use `sudo apt-get install php-mysql`.
