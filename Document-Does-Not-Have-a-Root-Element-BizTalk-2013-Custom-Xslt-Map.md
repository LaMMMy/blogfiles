<!-- {Title:"BizTalk 2013 Map with Custom XSLT -- Document Does Not Have a Root Element",PublishedOn:"2014-02-27 15:09",Intro:"Upgrading from BizTalk 2010 to 2013 caused the maps that were using a custom XSLT to always result in a 'Document Does Not Have a Root Element' error and a blank transformed XML Document. When using the xsl:strip-space elements='*'"} -->

# Introduction #

Ever since starting work with BizTalk using a custom XSLT file for doing mapping, we've been relying on the handy element at the root of our XSLT:

`<xsl:strip-space elements="*"/>`

This made it so we didn't get any extra whitespace in elements where it didn't belong. Because if there were extra line-feed/carriage-return/tab characters, BizTalk would choke on the errors, because would read those elements as actually having a value and throw an error for it having invalid data.

# The Cause#

Well it turns out that some of the underlying libraries that BizTalk uses were changed for 2013, most importantly the following:

> The Mapper uses the XSLCompiledTransform class. Previous BizTalk Server versions used the XslTransform class, which is obsolete

As outlined on the [What's New in BizTalk 2013](http://msdn.microsoft.com/en-us/library/jj248703.aspx) page on MSDN.

Well I decided to try mucking around with the test app, I found it all came down to this line at the top of the XSL file:

`<xsl:strip-space elements="*"/>`

I ended up getting the error 

>Whitespace cannot be stripped from documents that have already been loaded. Provide the input as an XmlReader instead

So, I did what it says, in my test app and it worked! However, I couldn't do so (or didn't know how to specify) in BizTalk.

# The Fix #

The fix as it turns out was to stop being lazy basically. Since we can't dig into BizTalk (we aren't using custom pipelines unfortunately) it came down to using a whole bunch of `[normalize-space(.)]` selectors around basically anything that selects text in the XSLT file. It's not at all ideal, but relatively easy and got me in the habit of doing it now.

I know I also enabled the `legacy Whitespace behavior` in my Host --> Settings dialog box. The combination of these 2 things did the trick.

If I get the chance to do this app over again, it will be done using custom pipelines, instead of orchestrations and I believe this will allow us to avoid this situation.

If anyone else has better suggestions, please let me know via twitter.