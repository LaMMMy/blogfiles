<!-- {Title:"BizTalk Upgrade to 2013 - Missing assembly SSOClient",PublishedOn:"2013-11-08 15:09",Intro:"After upgrading to BizTalk 2013 from BizTalk 2010 the SSOClient assembly was not found."} -->

After trying to upgrade a BizTalk 2010 installation to 2013 the first thing I went to do is test the 
WCF service that is attached to my main Receive Port. I was presented with a strange error in the Event Log

> Could not load file or assembly Could not load file or assembly 'Microsoft.BizTalk.Interop.SSOClient, Version-7.0.2300.0,
> Culture=neutral, PublicKeyToken=31b876ad766e45' or one of its dependencies. The system cannot find the file specified.

So, opened up the web browser and pointed it to the Metadata endpoint for the WCF service and received the same error.

After doing so, I opened up the BizTalk Server Configuration tool and it looked like Enterprise SSO was all there and configured, so I didn't know what was going on...on to Google. After much searching I was able to find one lone blog post off in the abyss complaining of the same error in the same situation and it told about having to re-install the Enterprise Single Sign-On separately.

So I ended up running a check on the GAC:
    gacutil -l Microsoft.BizTalk.Interop.SSOClient
	
Sure enough it was reporting back as version 5.xxx

So I went back to the BizTalk installation media and performed a Modify on the installation and sure enough, under "Additional Software" the "Enterprise Single Sign-On" checkboxes were unchecked. 

![ESSOUnchecked](/blog/img/ESSO_Modify.jpg "SSO_Modify")

So I completed the installation and checked for the error again...no change.

I then decided to look further into the installation files and found them:

> <path to install files>\BizTalk Server\Platform\SSO64\Setup.exe

After running the Setup.exe, I checked the browser, checked the GAC and all was well...then on to the next error.