<!-- {Title:"WCF Service using Client Certificates throws: HTTP Error 413.0 - Request Entity Too Large Error",PublishedOn:"2014-02-11 11:00",Intro:"After enabling client certificates on a WCF web service, clients are receiving and err stating 'The page was not displayed because the request entity is too large'"} -->

After switching on Client Certificates for a WCF service I've been working on, I decided to go and test a bunch of different messages. I've been using the same few "developer" test messages, you know, the ones that work every time.

Well, upon sending one of the larger messages, I suddenly received the following error:


## HTTP Error 413.0 - Request Entity Too Large ##
**The page was not displayed because the request entity is too large.**

It then gave some helpful hints (not being sarcastic here)

**Most Likely Causes**

- The Web server is refusing to service the request because the request entity is too large.
- The Web server cannot service the request because it is trying to negotiate a client certificate but the request entity is too large.
- The request URL or the physical mapping to the URL (i.e., the physical file system path to the URL's content) is too long.

**Things you can try:**

- Verify that the request is valid.
- If using client certificates, try:
- Increasing system.webServer/serverRuntime@uploadReadAheadSize
- Configure your SSL endpoint to negotiate client certificates as part of the initial SSL handshake. (netsh http add sslcert ... clientcertnegotiation=enable)

I decided to try the the first suggestion. I went to `C:\Windows\System32\inetsrv` and opened up the `applicationHost.config` file. Like the error suggests, I looked into how to add this updated value. I found a section already defined for the service I was trying to target:

    <location path="Default Web Site/MyService">
        <system.webServer>
            <security>
                <access sslFlags="Ssl, SslNegotiateCert, SslRequireCert" />
            </security>
        </system.webServer>
    </location>

So, I then added the element `<serverRuntime uploadReadAheadSize="2147483647" />` and altogether it looks like this:

    <location path="Default Web Site/MyService">
        <system.webServer>
            <security>
                <access sslFlags="Ssl, SslNegotiateCert, SslRequireCert" />
            </security>
			<serverRuntime uploadReadAheadSize="2147483647" />
        </system.webServer>
    </location>
 
I went and restarted IIS, tried the large message again and still received the error. After firing up Google to search for more help, I ended finding [a post](http://forums.iis.net/post/1952504.aspx) in [a thread](http://forums.iis.net/t/1169257.aspx) on the IIS forums which mentions using the `appcmd.exe` program to enter the setting:

`appcmd.exe set config -section:system.webserver/serverruntime /uploadreadaheadsize:2147483647 /commit:apphost`

After issuing the command, everything worked. If you want to issue the command at a site-level vs. machine-level, try the following:

`appcmd.exe set config "Default Web Site/MyService" /section:system.webserver/serverruntime /uploadreadaheadsize:2147483647 /commit:APPHOST`

That should get everything up and running...it did for me.

Any comments or suggestions, please let me know via Twitter.