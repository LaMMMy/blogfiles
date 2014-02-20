<!-- {Title:"Overriding Group Policy to Allow Google Chrome Updates",PublishedOn:"2014-02-19 11:09",Intro:"If you work in an enterprise environment where Chrome updates are disabled through a group policy, this may be your way to get updates going again."} -->

We all know an Enterprise/Corporate workplace can't just go letting people install whatever the please on PCs throughout the company, that makes sense. I even understand wanting to be in control of updates on software, even in the case of the Google Chrome web browser. However, we all know the quick pace at which Chrome gets updated as well anyone who has worked in a corporate environment knows the pace at which said companies get around to pushing out updates. They don't play well together.

Such is the case at my workplace. As a developer we have more freedoms on our development machines, the main one being local admins on our machines. This means we can install whatever we want, yusssss!

However, it doesn't mean we are immune to Windows Group policies and domain rules, we are still a part of the Domain. The one rule that has bit me lately is the admins taking over control of Google Chrome updates.

Then somewhere along the line, users out in the wild needed to be able to run some web-based application that only worked in Chrome and not in the IE that is a part of the default corporate PC image. This makes sense because at the time, the web browser out there was Internet Explorer 8. It was determined that this new app was tested and worked on Chrome version 31.0.16 something, so the next thing that was done is a Group Policy created, preventing the updating of Google Chrome at all.

I didn't really notice for a while, until after Google decided to move where the Chrome Apps were displayed (used to be from the new tab page, but now is a button on the bookmarks bar). So I tried to update from work and had no luck. This was no good, since in the world of web apps (especially publicly exposed ones) you need to be able to test with the version people are actually using. And no, the application development team couldn't have an exception to the rule!

**The Workaround**

Luckily, since we have local admin on our machines, we were able to get around this with a little help from Google Chrome forums and [Google group policy templates ](http://www.chromium.org/administrators/policy-templates).

- Open up `gpedit.msc`
- right-click *Administrative Templates*
- choose *Add/Remove Templates* and in the resulting dialog click the *Add* button
- navigate to the [admx](http://dl.google.com/update2/enterprise/GoogleUpdate.adm) that was downloaded and open it

After the template is added there should be a new folder added to the list. It should look something like this:

![snapin](/blog/img/goog_admx.jpg "gpedit")

We're interested in the *Preferences* and the *Applications* section

**Preferences**

The preferences section is not as interesting as the rest of it. Here we just set the period of time after which Chrome will auto-check for updates.

![snapin](/blog/img/gpedit_auto_update.jpg "gpedit_auto_update")

**Applications**

Here's the good stuff. Click on the following folders:

- Google Chrome
- Google Chrome Binaries
- Google Chrome Frame

and while they are highlighted in the right-hand pane for each one, double-click on *Allow installation* and make sure the *Enabled* radio button is selected.

![snapin](/blog/img/allow.jpg "allow")

then double-click on the *Update policy override* and make sure the enabled radio is chosen here as well. Then choose your desired Policy in the "Options" section down in the bottom. I chose *Always allow updates (recommended)* which I believe is the default.

![snapin](/blog/img/updateoverride.jpg "updateoverride")

Apparently this "should" be all you have to do. But sometimes, as was what happened in my case, the policy doesn't take. I had to do some digging and discovered [some help on the Google Support site](https://support.google.com/a/answer/1385049) 

Under the link on that page that says *Check that your settings are correct* I had to delete the registry entries it says should NOT be there. Which are under `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Update`

and called:


`Update{8A69D345-D564-463C-AFF1-A69D9E530F96}`
`Update{8BA986DA-5100-405E-AA35-86F34A02ACBF}`

Once those are deleted, you should be able to now go and update Chrome using the usual route under the Chrome Menu and *About Chrome*

![snapin](/blog/img/about.jpg "about")

Enjoy your new found freedom! Remember though, if you don't have local admin privileges, gpedit will not even be available.