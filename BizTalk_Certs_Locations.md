<!-- {Title:"Importing Client Certificates and Trusted Roots into the Certificate store for use by BizTalk",PublishedOn:"2014-01-30 15:09",Intro:"How to get Certs into the correct places for BizTalk to use. Not as simple as I originally hoped."} -->

# Introduction #

I've been posting what feels like way too much about BizTalk and client certificates lately, but I've come to realize, it's most of what I've been consumed with lately. Here's a note on how I've finally managed to refine the process of getting client certificates into the store in the correct place and how to get BizTalk reading them in its ports. The process is quite simple, it was all the hiccups I encountered that really slowed me down.

# The Certificate Store #

BizTalk hosts run under a service account that is provided during initial configuration, so when thinking about certificates in relation to BizTalk, we're thinking about this service account's store. There is a simple way to get them in there however and it is as simple as running the MMC as that user from the command-line

    c:\>runas /user:Mydomain\myUser mmc    
    Enter the password for Mydomain\myUser:

after entering the password, you are presented with the usual MMC console ready for the certificates Snap-In to be added.

![snapin](/blog/img/snapin.jpg "certs snapin")
 
When you highlight *Certificates* and click Add, there is no dialog to choose what you'd like to add the snap-in for like there usually is, it just automatically adds the *Current User*. Once this opens up, it's as simple as importing your Client Certificate to the *Personal* store. After this, take a look at the certificate's certification path. Once you know the path, make sure the Root CA (top of the path) is in the *Trusted Root Certification Authorities* and any intermediate authorities (the ones below the Root CA) are in the *Intermediate Certification Authorities*.

In the ideal situation, you should now be able to go on to using the Client Certificate in your BizTalk application send ports. To see how to set up a Send Port for using client certificates, see my post [where it's outlined here in detail.](http://www.bensoniam.com/blog/BizTalk_SendPort_ClientCerts)

# My Issue #

So, the most critical step to getting Client Certificates into a usable state for BizTalk is most definitely importing the certificates and trusted roots into the store that BizTalk can read and use. This simple act is what gave me the most trouble and run around on my dev box. 

I hit my first roadblock when I went to try the runas command from the command-line:

    Enter the password for :Mydomain\myUser:
    Attempting to start mmc as user ":Mydomain\myUser" ...
    RUNAS ERROR: Unable to run - mmc
    740: The requested operation requires elevation.

Hmm, OK. So after researching, I found this has to do with User Account Control and that when running in command-line it can't actually pop-up with the User Account Control approval box. So I turned to `gpedit.msc` to try to tweak some settings

![gpedit](/blog/img/gpedit.jpg "gpedit User Account Control")

To me that looks pretty wide open, but I'm no domain admin (feel free to tweet me, with advice). None of the changes helped avoid the `740: The requested operation requires elevation.` error, even after logging off/on, etc.

So next, I decided to try the UI route. I opened Start menu, searched for mmc.exe, then `Shift-Right-Click --> Run as Different user`. So it then popped up the following

![diffuser](/blog/img/diffuser.jpg "Different User Dialog")

I entered the username password of the service my BizTalk Send Port runs under and clicked OK. Suddenly another dialog was presented:

![uac_admin](/blog/img/uac_admin.jpg "Different User Admin Dialog")

So, I followed the instructions and entered an administrator password and MMC opened. I added the "Current User" certificates snap-in, same as above. The first thing I noticed however, was that the certs all looked the same as when just open MMC as admin...fishy. Regardless and went on adding my client certificates as outlined above, thinking it was for the service account...wrong! BizTalk could not find the certificate despite it "being there".

>Cannot find the X.509 certificate using the following search criteria: StoreName 'My', StoreLocation 'LocalMachine', FindType 'FindByThumbprint', FindValue '99 a2 fd 91 db 1e 91 c5 86 4e 4c bf 8a df 8c 66 60 e2 56 9c'.

So, I hit up Google and everything came down to the same solution, you have to add the client cert to the service account's Current User store, which I thought I already had. I tried adding and re-adding and trying to hit different stores to no avail. Finally I decided to just spin up a new VM and try the same steps as above. I went straight to the command-line route and it worked immediately. I added the certificates snap-in and immediately noticed a difference: there was no dialog to choose *My user account, Service account or Computer account* like there was when I ran it with `Shift-Right-Click --> Run as Different user` on my initial dev box. It just automatically added the *Current User* like in my instructions above.

Usually it looks like this and you have to choose (because usually you are running as an Administrator account):

![adminsnap](/blog/img/adminsnap.jpg "certs snapin when admin")

**If you get the dialog above, it probably means you're not running as the service account you chose**

This prompted me to go back and try again. This time I did the command-line and it still gave me the *740* error. So I tried the `Shift-Right-Click --> Run as Different user` route but this time instead of entering the administrator password for the 2nd time I chose the `User another account` option and RE-ENTERED the service account information. MMC opened!

This time when I added the snap-in for certificates, it only added the *Current User* automatically, I imported the Client Certificate, tried BizTalk again and Hallelujah it all worked!

# Conclusion #

Don't trust anything in Windows :) This was Server 2008 R2, apparently older versions don't have this problem, since there was no User Account Control.

Please feel free to tweet @ me with any questions, critiques or comments.