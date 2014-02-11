<!-- {Title:"Setting up a WCF Service and Client to user 2-Layer Authentication (Client Certs and WS UserName Security)",PublishedOn:"2014-01-30 15:09",Intro:"By default WCF doesn't allow client certs AND message level/SOAP authentication. Here's how to do it using customBindings"} -->


# Introduction #

As far I can could find in my research, WCF does not make it easy to (figure out) how to enable both Transport-Layer client certificates and SOAP, message-layer client credentials. It took me a few days of messing around and researching before I was finally able to do it. It turns out it can be with a customBinding with a few small elements added to your config files...it's the fact that WCF hides it rather well that's the tough part.

Hopefully this will unhide it for someone out there.  


# Service Configuration #

Here's how it's setup for the service side of things. I basically just added the `security` element to allow for UserName authentication and the `httpsTransport` for requiring Client Certificates

		<customBinding>
			<binding name="CustomBinding">                    
				<textMessageEncoding messageVersion="Soap11" />
				<security authenticationMode="UserNameOverTransport" />
				<httpsTransport requireClientCertificate="true" />
			</binding>
		</customBinding>

		<services>
			<service name="MyNameSpace.WCFService.Something">
				<endpoint
					address="" binding="customBinding"
					bindingConfiguration="CustomBinding"
					name="CDARequestEndpoint"
					contract="MyNameSpace.WCFService.ISomething"
					bindingNamespace="urn:hl7-org:v3">
				</endpoint>
			</service>
		</services>

# Client Configuration #

On the client side, since I'm working in Visual Studio, all I did was add a service reference to the WCF service configured above and it pulled down all the correct configuration and added to my app.config file (I have a Winforms test app). You could also just manually add this configuration to your config file and it would most likely work.

The only thing I added was the behavior shown below and associated it to the endpoint.

    <system.ServiceModel>
		<bindings>
	    	<customBinding>
	      		<binding name="CustomEndpointBinding">                    
	        		<textMessageEncoding messageVersion="Soap11" />
	        		<security authenticationMode="UserNameOverTransport" />
		    	    <httpsTransport requireClientCertificate="true" />
	      		</binding>
	    	</customBinding>
	    </bindings>
	
	    <behaviors>
	      <endpointBehaviors>
	        <behavior name="ohBehave">
	          <clientCredentials useIdentityConfiguration="false">
	            <clientCertificate findValue="6D0DBF387484B25A16D0E3E53DBB178A366DA954" storeLocation="CurrentUser"
	               x509FindType="FindByThumbprint" />            
	           </clientCredentials>          
	         </behavior>
	       </endpointBehaviors>
	    </behaviors>
	
	
	    <client>
	      <endpoint address="https://myservice.ca/MyService/Contract.svc"
	        binding="customBinding" bindingConfiguration="CustomEndpointBinding" behaviorConfiguration="ohBehave"
	        contract="MyService.Contract" name="CustomEndpointBinding" />      
	    </client>
    </system.serviceModel>

If you don't want to add the client certifcate using a behavior in the config, you can do it through code like so:

    MyService.proxyClient client = new MyService.proxyClient("CustomEndpointBinding");
   	client.ClientCredentials.ClientCertificate.SetCertificate(StoreLocation.CurrentUser, StoreName.My, 	X509FindType.FindByThumbprint, "6D0DBF387484B25A16D0E3E53DBB178A366DA954");


When it comes to the WCF Client Credentials, those will always have to be added programatically as far as I know

	 client.ClientCredentials.UserName.UserName = clinicAdAcount;
     client.ClientCredentials.UserName.Password = clinicPassword;

You can apparently create a [behaviorExtension](http://stackoverflow.com/questions/3725294/set-wcf-clientcredentials-in-app-config/7350369#7350369), but we'll leave that for another day.

Now, just be sure to set IIS to "Require SSL" and "Client Certificates" set to "Require"

![IIS_SSL](/blog/img/IIS_SSL.jpg "IIS_SSL")

Armed with this info, your WCF service should now require 2-layer authentication. 

Please feel free to tweet @ me with any questions, critiques or comments.