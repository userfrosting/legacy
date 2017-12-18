# Troubleshooting
The following is a list of commonly encountered problems when setting up UserFrosting.  If you have found a solution to a problem that you think others might also encounter, it would be greatly appreciated if you can write a brief explanation of the problem and the solution here.  Please provide relevant details about your platform (OS, version of PHP, etc) and link to any relevant issues.

### Safari - Getting authentication to work with iframes & cross domain
#### Problem

When using Userfrost though a Iframe on a Cross Domain Setup, For Example Website A.com Hosts an Iframe which is Website B.com

Using Safari, When a User Logs in you get a 404 Error. This is because as default Safari Blocks 3rd Party Cookies. If you set 3rd Party Cookies to 'NEVER' then Safari will be able to generate the session token which allows the user to log into Userfrost.

Unless the User has had access to the second domain (b.com) then Safari will block the cookie from being generated. If you manually navigate to B.com then back to A.com you will be able to log in. This is because Safari now Trusts the domain in question. 

#### Solution

Get away from an Iframe on a Cross Domain. This can be done by hosting Userfrost on the Same Site EG: Server/Domain

Setup a Pop Up which will 'OnLoad' : B.com then close immediately - Please note Safari also by default Blocks Pop ups. If you set it to Never Block Pop ups, it will still ask the user weather to allow the pop up to be executed. Not ideal, i know!

Use some Java to talk to the domain B.com in the background. Basically i tried this with some queries, if the user can get some data or feedback from b.com then Safari will trust the domain which in turn will allow the user to log in. Although this sounds the best solution, If the User Bookmarks the page; This practice doesn't work to well. 

**Final Solution**
From hours or searching the Web i found the only future proof answer is to get rid of the Iframe. Safari really doesnt like Iframes which Track. Unless you can ensure your users are happy with having there Cookies set to 'Never'
 
#### Relevant issues and links

http://stackoverflow.com/questions/408582/setting-cross-domain-cookies-in-safari

http://stackoverflow.com/questions/12950541/cross-domain-cookies-in-iframe-safari

### MySQL shuts down unexpectedly after installing UserFrosting
#### Problem

In at least one development platform, MySQL has been observed to [shut down unexpectedly](https://github.com/alexweissman/UserFrosting/issues/247) after installing UF.  

#### Solution
See https://stackoverflow.com/questions/18022809/xampp-mysql-shutdown-unexpectedly.