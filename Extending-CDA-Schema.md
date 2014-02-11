<!-- {Title:"Extending the Clinical Document Architecture Schema",PublishedOn:"2013-12-09 08:34",Intro:"Is the CDA schema missing some piece that makes it a no-go for your organization? Then just extend it!."} -->

##Extending The CDA Schema##

I've been working on a project for the past year or so involving HL7 Version 3 and Clinical Document Architecture schemas and messages. I've come to learn that in the process of being defined, both the HL7v3 and CDA schemas have become very fat. They reek of being designed by committee, or rather several sub-committees which combine to make one giant super committee. There are some structures with more than enough elements and attributes to account for anything you could possibly want, any message that can be conveyed, could be conveyed. Except of course, the one you're trying to convey.

What we were trying to convey was the status of a `<serviceEvent>` that is sent across the wire. Sometimes  there may be microbiology test batteries ordered which multiple tests, or time based results related to them. This means the lab will send out "preliminary" results, sometimes before the full battery of tests are done. For example, a certain test may be observed at different points in time and the results sent out before the test is truly "complete". This is done in case there are extreme results in the first time period, then care action can be taken without having to wait for the entire time to pass (think incubating certain specimens). 

On first glance, we thought we were safe, because there are so many other elements with a `statusCode` attribute, surely `<serviceEvent>` does as well. It doesn't.

So, what did we do? We extended the CDA schema.

How did we do it? First we made a separate XSD file like so:

    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns="urn:my-namespace.ca" xmlns:hl7="urn:hl7-org:v3" targetNamespace="urn:my-namespace.ca" elementFormDefault="qualified" attributeFormDefault="unqualified">
	    <xs:import namespace="urn:hl7-org:v3" schemaLocation="datatypes.xsd"/>
	     <xs:element name="statusCode" type="hl7:CS">
		    <xs:annotation>
			    <xs:documentation>This element in an extension to CDA R2. The purpose is to indicate whether the report is preliminary or final</xs:documentation>
		    </xs:annotation>
	    </xs:element>
    </xs:schema> 

Then in the `POCD_MT000040.xsd` we had to add an include the newly defined namespace:

    <xs:schema xmlns:mif="urn:hl7-org:v3/mif" 
        xmlns="urn:hl7-org:v3"
        xmlns:myns="urn:my-namespace.ca" 
        xmlns:xs="http://www.w3.org/2001/XMLSchema" 
        targetNamespace="urn:hl7-org:v3" 
        elementFormDefault="qualified">

This is the only place we had to actually modify the `CDA.xsd` file. Fortunately, it doesn't actually "change" the schema itself.  

Take note of the new namespace prefix **xmlns:myns="urn:my-namespace.ca"**

So then in the XML of the CDAs that are being submitted, if a `<serviceEvent>` needs a statusCode, it can be added like so:

    <documentationOf typeCode="DOC">
        <serviceEvent classCode="ACT" moodCode="EVN">
          <myns:statusCode xmlns:myns="urn:myns" code="active" />
        </serviceEvent>
    </documentationOf>

Of course, this extension must be optional, or else it would be pretty much the same as modifying the CDA schema and if we're going to be doing that, why not just go wild and "fix" all the things that would make it easier for our project...we could make our own "standard" just like everyone else ;)