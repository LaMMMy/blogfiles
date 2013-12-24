<!-- {Title:"WCF Base being shown incorrectly when behind TMG.", PublishedOn:"2013-12-23 15:29",Intro:"WCF Service Base Address is being reported incorrectly when behind a Threat Management Gateway."} -->

Recently, I discovered a little gem when dealing with an error reported by the consumer of one of our WCF services. It was running just fine for the longest time, then we moved it behind a TMG, with a new externally facing base address. This external https: URL is mapped to an internal https: URL which points to an internal machine name. The SSL certificate was created for the External URL, which is housed on the TMG server.

Even after this change, for about a month it was going well and I could see data moving in and out of it from a few clients as well as from the test applications I run against it.

That was until today, when one of the clients was reporting an SSL error when trying to access the service:

>ERROR: Error invoking web service [ClientTransportException: HTTP transport error: javax.net.ssl.SSLHandshakeException: java.security.cert.CertificateException: No subject alternative DNS name matching <myinternalservername.mydomain.com> found.]

Two things lead to this happening.

1. The client is using a Mirth service to call our service
2. The client had recently cached a version of our WCF's wsdl (post-move to behind the TMG)


I must note as well, this particular client has a domain trust, so it actually COULD reach our internal-only DNS entry (unlike any other clients). Which is why, instead of a 404 or similar error, they actually received an SSL error, because the certificate for a host other than what was the internal server name.

This prompted me to visit the WCF services help page and sure enough the references in the WSDL were pointing to the internal machine name DNS entry, which of course could not be reached by clients other than myself or this one special client. This is when the problem of the incorrect base address in the WSDL was discovered.

The way to tell if this happening to your WCF service is, while at the .svc file for the service (in your browser) look at the url that is published beside the

> svcutil.exe

If the the base address matches the URL to your service, then that basically means all is well and the correct base address is being published. However in my case, it was something like this:

> svcutil.exe http://myinternalservername/myservice/service.svc?wsdl

instead of 

> svcutil.exe http://myexternalurl/service.svc?wsdl

So now I say this was the perfect storm because it turns out the Mirth service that calls the WCF, actually reads the service endpoint address from one of the elements inside the WSDL file that is published. This is unlike anything I've done, where you just pull down the WSDL once and use it to build proxy client objects if necessary and then just POST the resulting serialized objects to whatever endpoint you want to specify. BUT, since their Mirth service was reading the endpoint address from the cached WSDL file, it was trying to do an SSL handshake with our **internal** server DNS entry, for which there is no SSL cert...thus the error described above.

If it weren't for this one client, we would have just been going along with our internal server machine name DNS entry published to the public facing internet.

**What did we do to solve it?**

Well after some searching I found out about a configuration element called `useRequestHeadersForMetadataAddress` on the MSDN site. Duh, who wouldn't think to add that to their config, amirite? After that I was able to then search for examples of it's use and found, it's actually quite simple to use. Just add the following to your Service's behavior configuration.

		  <useRequestHeadersForMetadataAddress>
            <defaultPorts>
              <add scheme="http" port="80" />
              <add scheme="https" port="443" />
            </defaultPorts>
          </useRequestHeadersForMetadataAddress>

In fact, I discovered that it will still even work if you just add the following line to the behaviour:

    <useRequestHeadersForMetadataAddress />

So, under your system.serviceModel it would look like this:

    <behaviors>
      <serviceBehaviors>
        <behavior name="ServiceBehaviorConfiguration">
          <serviceDebug httpHelpPageEnabled="true" httpsHelpPageEnabled="true" includeExceptionDetailInFaults="true" />
          <serviceMetadata httpGetEnabled="true" httpsGetEnabled="true" />
		  <serviceSecurityAudit auditLogLocation="Application" serviceAuthorizationAuditLevel="Failure" messageAuthenticationAuditLevel="Failure" suppressAuditFailure="false" />
		  <useRequestHeadersForMetadataAddress>
            <defaultPorts>
              <add scheme="http" port="80" />
              <add scheme="https" port="443" />
            </defaultPorts>
          </useRequestHeadersForMetadataAddress>  
        </behavior>
      </serviceBehaviors>
    </behaviors>

I'm hoping this will help someone in the future, as I know it's one of those simple, somewhat hidden "features" that never come to light until you're in a bind. If nothing else, at least now, I'll know where to look when it happens to me next time.