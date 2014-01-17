<!-- {Title:"Configuring BizTalk WCF-BasicHttp Send Ports to use Client Certificates",PublishedOn:"2014-01-17 15:09",Intro:"Wanting to add Transport-Level client certificate validation and Message-Level user authentication? Here's how."} -->

Wow, where does the time go? I'm still writing 2013 on all my Dogecoin trans...no, bad joke.

But honestly, how is it over half-way through January and I haven't written anything at all? Well, mostly it's because I've been busy trying to figure out to get client certificates to work with our BizTalk services. This was something that, all along, we just assumed would be a "flick-a-switch" type implementation. You'd think after so many disappointments, I'd have learned by now, there is no such thing in the world of BizTalk.

So, at first I assumed it would all be handled by IIS and BizTalk would just chug along in the background. This was in part due to my naivety with BizTalk and even more than naive level of understanding of SSL and certificates in general (I mean looking back, did I really expect some kind of magically certificate sender to step-in and pass 
along the cert?)

So anyway, enough rambling

# The Problem #
**TransportWithMessageCredential:** Now, I'm not saying there is a problem with this Security Mode, but from the start, my colleagues and I **both** saw it and instantly assumed:
 
> oh, that must mean you can just provide both, for now let's just use the message credentials part until we get the client certs issued.

So with that being said, the biggest problem was the fact that I lacked knowledge, but really, take a look at this dialog box, with all it's options I swear it was TRYING to trick me:

![BasicHttpProps](/blog/img/WCF_BasicHttp_props.jpg "BasicHttp")

You have Message security, Client Certificate, User Name credentials, it's all there! For the longest time, I just went on with development, using just message security because that's all we had. When you choose message, here's what it looks like:

![MessageProps](/blog/img/WCF_BasicHttp_message.jpg "Message")

It greys out the cert related things, and just leaves message related credentials. 

So now, we finally had our test client certs, I got the IIS server set up to trust all the right issuers, got the IIS Server cert setup, checked "Require" on the Client Certificates section in IIS configuration. Then, I opened up our BizTalk solution to the same WCF-BasicHttp properties page again and switched to TransportWithMessageCredential, thinking all I would have to do is supply a client cert...wrong:

![TransMessageProps](/blog/img/WCF_BasicHttp_transmessage.jpg "TransMessage")

It's hard to tell, but I did change the security mode in that screenshot...but nothing else changed. So basically you can't supply a cert here...

# The Solution #

Now that I can start to understand what is actually going on in my situation, it makes more sense why the security page does not allow an SSL client cert AND message credentials. It's because this particular tab applies only to message security. The *Transport* part of *TransportWithMessageCredential* is just saying, there will be Transport-Layer security of some sort, but it's not being handled by BizTalk, however, BizTalk will be expecting it. This tab has nothing to do with the HTTP/IIS layer of security.

You want to change those, you need to edit the identity of the endpoint. That's on the General tab. Duh.

So, to make the BizTalk WCF-BasicHttp adapter send port attach a client cert to the HTTP request, you go to:

> General --> Endpoint Identity Box --> Edit button

That opens up the following dialog:

![endpoint_ID](/blog/img/endpoint_ID.jpg "endpoint_ID")

**(General) Section:** This is the section that deals with Service you are trying to send to and it's identity

**Certificate Reference:** This is the section that allows you to input the client certificate

**NOTE:** if you fill out the Certificate Reference section, you also MUST fill out one of the textboxes in the (General) section, without it, the send port will fail the SSL/TLS security negotiation for the service's domain...even though the Address (URI) is specified in the Endpoint Address box on the *WCF-BasicHttp Transport Properties*. It seems that once you mess with the Endpoint Identity, you need to mess with it thoroughly. 

Once that identity dialog is completed, you click OK and then you'll notice some extra text on the *WCF-BasicHttp Transport* Properties dialog:

![endpoint_ID2](/blog/img/endpoint_ID2.jpg "endpoint_ID2")

I'm not sure where this configuration XML is being stored/read, but it matches what needs to be entered in a normal WCF Service web.config when defining an endpoint and BizTalk must be overriding the default web.config that it generates for the WCF service.

I just want to add that I could find little to no documentation on what the Endpoint Identity Editor fields actually meant. I know after thinking and playing for a while it was obvious, but my Google-fu failed me on this on.

# Conclusion #

Before playing with Client Certificates (or any for that matter) and BizTalk, make sure you read up a little bit and prepare to be frustrated for the first run. Oh and remember -- the message-layer can't do everything.
