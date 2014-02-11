<!-- {Title:"BizTalk Server - How to Restore Master Secret",PublishedOn:"2013-12-17 14:09",Intro:"Cannot perform encryption or decryption because the secret is not available from the master secret server."} -->

One Saturday night I was feeling particularly proactive and decided to deploy a small change to an XSLT file that is being used by our BizTalk implementation. It was nothing major, I'd been running it in the development and test environments for a couple days to make sure it kept running smoothly, so I felt it was ready to go live.

I proceeded to do the same process I've done a million times by down...stop the BizTalk application, import --> MSI, choose my file and import it. I did this just fine, but when it came time to restart the BizTalk application that I was updating I got the following error popping up and being logged in the Event Viewer:

![MasterSecret](/blog/img/MasterSecret.jpg "Master_Secret")

Also, the applications were not starting up...insta-panic...at close to midnight on a weekend no less. This was not good.

Luckily I did a quick google search and found the answer (also luckily we had our master secret backed up and password in our store).

To restore it, run the following on your Master Secret Server (usually whatever SQL Server the BizTalk Server databases reside):

-      In a command prompt, goto C:\Program Files\Common Files\Enterprise Single Sign-On\ (this is the default location of the master secret backup if you don't store it elsewhere)
-     Enter "ssoConfig -restoresecret <ssobackupfile>.bak"
-     Enter the password that was set on BizTalk installation. (you're screwed without this)

After that, palms a-sweatin', heart a-thumpin' I went back to the BizTalk runtime server and restarted the application that I had deployed and lo and behold, it started just fine.

There was even a nice little message in the Event Viewer saying: 

> Got the current secret from the master secret server.
>  Secret Server Name: MyServer
>  MSID: 8f26122c-4830-4216-bba5-e49dca3ba13b

Followed by:

>Recovered from failure to get master secrets. Secret Server Name: ServerName

Talk about a relief...