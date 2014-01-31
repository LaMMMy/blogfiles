<!-- {Title:"Configuring BizTalk WCF-BasicHttp Ports to use Client Certificates AND Message-Level Authentication",PublishedOn:"2014-01-17 15:09",Intro:"Wanting to add Transport-Level client certificate validation and Message-Level user authentication? Here's how."} -->

**Jan 30, 2014: I've since updated the following to be correct**

I previously thought I had it working and posted some incorrect steps. It turns out that setting two-factor Client Cert/WS-Security authentication is not possible using a BasicHttp binding/adapter. So, I've updated the Solution section. 

Wow, where does the time go? I'm still writing 2013 on all my Dogecoin trans...no, bad joke.

But honestly, how is it over half-way through January and I haven't written anything at all? Well, mostly it's because I've been busy trying to figure out to get client certificates to work with our BizTalk services. This was something that, all along, we just assumed would be a "flick-a-switch" type implementation. You'd think after so many disappointments, I'd have learned by now, there is no such thing in the world of BizTalk.

So, at first I assumed it would all be handled by IIS and BizTalk would just chug along in the background. This was in part due to my naivety with BizTalk and my more-than-naive level of understanding of SSL and certificates in general (I mean looking back, did I really expect some kind of magically certificate sender to step-in and pass 
along the cert?)


# The Problem #
**TransportWithMessageCredential:** Now, I'm not saying there is a problem with this Security Mode, but from the start, my colleagues and I **both** saw it and instantly assumed:
 
> oh, that must mean you can just provide both transport(client cert) and message security on this single dialog, yay! For now let's just use the message credentials part until we get the client certs issued.

So with that being said, the biggest problem was the fact that I lacked knowledge, but really, take a look at this dialog box, with all it's options I swear it was TRYING to trick me:

![BasicHttpProps](/blog/img/WCF_BasicHttp_props.jpg "BasicHttp")

You have Message security, Client Certificate, User Name credentials, it's all there! For the longest time, I just went on with development, using just message security because that's all we had. When you choose message, here's what it looks like:

![MessageProps](/blog/img/WCF_BasicHttp_message.jpg "Message")

It greys out the cert related things, and just leaves message related credentials. Simple, just enter your username and password which the WCF is expecting and Bob's your uncle...client certs must be even easier! 

So now, we finally had our test client certs, I got the IIS server set up to trust all the right issuers, got the IIS Server cert setup, checked "Require" on the Client Certificates section in IIS configuration. I could reach the mex endpoint that's put up by the WCF service in a browser and choose the client cert from the service on which our BizTalk Send Port resides.

Then, I opened up our BizTalk solution to the same WCF-BasicHttp properties page again and switched to TransportWithMessageCredential, thinking all I would have to do is supply a client cert using the that handy looking browse button...Wrong!

![TransMessageProps](/blog/img/WCF_BasicHttp_transmessage.jpg "TransMessage")

It's hard to tell, but I did change the security mode in the screenshot above...but nothing else changed. So basically you can't supply a cert here, the handy dandy browse button turns out to be only for when you choose Transport security with a "Transport client credential type:" of Certificate.

# The Solution #

Now that I can start to understand what is actually going on in my situation, it makes a bit more sense why the security page does not allow an SSL client cert AND message credentials. It's because this particular tab applies only to message security. The *Transport* part of *TransportWithMessageCredential* is just saying, there will be Transport-Layer security of some sort, but it's not being handled by BizTalk, however, BizTalk will be expecting it. This tab has nothing to do with the HTTP/IIS layer of security.

You want to change the actual Transport-Layer/HTTP endpoint settings, you need to create new Send and Receive Ports that use the **WCF-Custom** adapter combined with a customBinding endpoint binding.

# The Receive Port #

1. First off we need to update the Receive Location to use the `WCF-CustomIsolated` Adapter. This signifies that this WCF service will be using a Custom binding that is also being hosted on Isolated Host, such as IIS.
- Edit the Receive Location in the the BizTalk Admin console, choose `WCF-CustomIsolated` for the Type and click Configure.
- Set the `textMessageEncoding` --> `messageVersion` attribute to "Soap11" because Default value is Soap 1.2 (unless you want to use Soap 1.2).
- remove the `httpTransport` binding element, because if you **don't** do this, the `httpsTransport` element (which is required in order to get this all to work) can't be added
- add the security element. At this point, it should look like so (order of elements matters)

- ![ReceiveCustom](/blog/img/ReceiveCustom.jpg "ReceiveCustom")

- the `security` binding element has an attribute called `authenticationMode` which should be switched to `UserNameOverTransport`. Despite the name, this is what allows the UserName credentials to be sent along with the Message (in the SOAP envelope `security` element). Everything else in `security` is left with the defaults

- ![binding_Security](/blog/img/binding_Security.jpg "binding_Security")

- the `httpsTransport` element has an attribute named `requireClientCertificate` this should be set to "true" everything else is left with the defaults.

- ![binding_httpstransport](/blog/img/binding_httpstransport.jpg "binding_httpstransport")

- then you can add whatever behaviors are necessary for your service. For me, I needed to allow `useRequestHeadersForMetadata` because I'm behind a TMG, as well as allowing https Get for mex data.

- ![behave](/blog/img/behave.jpg "behave")

- after that, if this is being added to a WCF service which already exists, you have to go into the the web storage directory (wwwroot/myService) and edit the .svc file in order to handle the WCF-CustomIsolated adapter.
- change `Factory="Microsoft.BizTalk.Adapter.Wcf.Runtime.BasicHttpWebServiceHostFactory...` to `Factory="Microsoft.BizTalk.Adapter.Wcf.Runtime.CustomWebServiceHostFactory...`
- now if you're not submitting to this from another BizTalk application, you're done. You can now call this WCF service from an external application/service just by hitting the service URL and attaching a Client Cert/Message UserName credentials

# The Send Port #

This section is only necessary if you not only need to use a Receive Port as outlined above, but ALSO need to call that WCF service from another BizTalk Application/Send Port. I'll outline in detail how to get the client cert attached and have the Send Port sending to the Receive port (well, it's WCF service).

For the most part, the configuration is nearly the same as for the Receive Port, except for a few different steps that are needed in order to get the Client Certificate and Message Credentials flowing.

1. The send port uses the WCF-Custom adapter
2. On the General tab choose WCF-Custom and enter your SOAP Action Header. Click Configure
3. On the Binding tab, just repeat steps 1 - 10 from the Receive port instructions above.
4. The Behavior tab is where we enter the actual client certificate
5. Right-click on the Behavior root element and choose "Add extension"
6. Choose the `clientCredentials` extension
7. After the extension element is add, expand `clientCredentials` and select `ClientCertificate`
8. ![clientCred](/blog/img/clientCred.jpg "clientCred")
9. In the right side pane is where you enter your parameters for finding the client cert in the store. As shown above I'm checking the "CurrentUser" and finding using the thumbprint of the cert (`FindByThumbprint` which is in the System.Security.Cryptography.X509Certificates.X509FindType [System.Security.Cryptography.X509Certificates.X509FindType enumeration](http://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509findtype(v=vs.110).aspx))...the thumbprint can be entered with or without spaces and it still works.
10. The "CurrentUser" referenced in the step above is actually the service account under which your Send Port is running. So you have to add the certificate to that user's store [which is outlined here in more detail.](http://www.bensoniam.com/blog/BizTalk_Certs_Location)
11. Now move on to the **Credentials Tab**. There is were you choose "Do not user Single Sign-On" and enter the credentials for the Soap Header.

Once that is all done, and your certificates are all in place, everything should work and the 2 BizTalk applications should be able to talk to each other with 2-level authentication.

# Conclusion #

Before playing with Client Certificates (or any for that matter) and BizTalk, make sure you read up a little bit and prepare to be frustrated for the first run. Oh and remember -- the message-layer can't do everything.

If you want to do this for a Non-BizTalk WCF service check out [my write up](http://bensoniam.com) on that for the details.

Please feel free to tweet @ me with any questions, critiques or comments.