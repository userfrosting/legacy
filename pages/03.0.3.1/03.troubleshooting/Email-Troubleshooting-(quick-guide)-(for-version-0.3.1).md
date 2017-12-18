Firstly, it has to be noted that most mail issues are not related to userfrosting. But as people keep coming to the chat with (g)mail problems, I decided to give a quick insight into how to solve the most apparent mail issues. For a more comprehensive troubleshooting guide see [Troubleshooting PHPMailer Problems](https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting).

The first step of troubleshooting your mail issues is to check your `php_error_log`. If you are using lampp it is most likely located at `/opt/lampp/logs/php_error_log`. Here are some of the error messages and what you might want to check

##Gmail and `SMTP Error: Could not authenticate.`

First make sure that your settings in `/userfrosting/config-userfrosting.php` are correct. See [Google Apps SMTP settings](https://support.google.com/a/answer/176600?hl=en). Your settings should then for example look like that:

```
            'smtp'  => [
                'host' => 'smtp.gmail.com',
                'port' => 587,
                'auth' => true,
                'secure' => 'tls',
                'user' => 'YOURMAILADRESS@gmail.com',
                'pass' => 'YOURPASSWORD'
            ],
```
*Side note:* [PHPMailer](https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting) on the use of SSL on port 465:
>Don't use SSL on port 465, it's been deprecated since 1998 [...]

Once you got your settings right, you need to take care of some things, specific to gmail.

Firstly, you need to allow less secure apps. Which can be done [here](https://myaccount.google.com/security?pli=1&nlr=1) on the very bottom of the page.

Secondly, you have to [allow access to your Google account ](https://accounts.google.com/DisplayUnlockCaptcha). Follow the instructions given when you follow the link and then immediately try to send an email. 

If you choose to use the gmail account also in production, you might need to [allow access to your Google account ](https://accounts.google.com/DisplayUnlockCaptcha) again as you might be logging in from a different location and possitively an IP address that is known to google.

######Note: I don't know googles terms of service exactly and may not be hold accountable if following this instructions is in viaolation with said TOS.

##`SMTP server error: MAIL FROM command failed` 
####`Detail: syntax error (#5.5.4)`

Details of SMTP are beyond the scope of this quick guide. If you get this error, the first thing you want to do, is to make sure your `admin_email` in the database is set to the mail address you use for sending emails. You can do this for example by running following query (given that you use the `uf_` prefix):

```
UPDATE `uf_configuration` SET `value` = 'YOURMAILADRESS@gmail.com' WHERE `uf_configuration`.`name` = "admin_email";
```

If you check your `admin_email` in the UF settings or in phpmyadmin, it might appear to be ok. However, **blank spaces after the mail address can ruin your day.**

It is sometimes said [[1](http://stackoverflow.com/questions/4421866/cakephp-smtp-emails-syntax-error)][[2](http://stackoverflow.com/questions/4421866/cakephp-smtp-emails-syntax-error)] that this error is due to the format of the mail address. While this might very well be true in some cases, I experienced no problems with the so proclaimed wrong format .

##`SMTP Error: Could not connect to SMTP host.`

This error can have various reasons. If you are sure that your host settings in `/userfrosting/config-userfrosting.php` are correct you should refer to [Troubleshooting PHPMailer Problems](https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting).