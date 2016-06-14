---
layout: post
title:  "The ultimate security RSS feed compilation"
date:   2016-06-14 15:16:05 +0430
categories: 
---

If you are, as I am, a big fan of RSS feeds and interested in being up to date with the latest security news, you have come to the right place. 

I love RSS. I have [newsbeuter](www.newsbeuter.org) on my Arch Linux and the open source [Flym](https://github.com/FredJul/Flym) on my Android. I have all my favorite RSS feeds in both applications, so I am in the first line to know about a new blog post or a new discovered exploit.

However, compiling my list of good security blogs took some time. I have been reading security blogs for some time now, and separated the superficial non-interesting ones from the high-quality ones. Some provide the latest news with a high frequency, others are less frequent, but very interesting blog sites.

My primary criteria for choosing the security blogs to follow:

- Should discuss security topics
- Must have published at least 3 blogs in the last 6 months and at least 1 blog in the last month
- Should publish high quality posts

If you are also reading your RSS feeds with newsbeuter, just add the following lines to your .newsbeuter/urls file:

```
https://www.us-cert.gov/ncas/current-activity.xml
http://krebsonsecurity.com/feed/
https://www.schneier.com/blog/atom.xml
http://www.darkreading.com/rss_simple.asp
http://www.cio.com/index.rss
http://feeds.pcmag.com/Rss.aspx/SectionArticles?sectionId=25442
https://www.wired.com/category/security/feed/
http://www.forbes.com/sites/firewall/feed/
http://www.techrepublic.com/rssfeeds/topic/security/
http://www.zdnet.com/blog/rss.xml
http://news.hitb.org/rss.xml
https://threatpost.com/feed/
http://www.itsecurityguru.org/feed/
http://itsecurity.co.uk/feed/
https://dankaminsky.com/feed/
http://emergentchaos.com/feed
http://www.flyingpenguin.com/?feed=rss2
http://feeds.feedburner.com/inftoint
https://www.grahamcluley.com/feed/
https://blog.elearnsecurity.com/feed
http://www.hackingarticles.in/feed/
http://holisticinfosec.blogspot.com/feeds/posts/default
http://jeffsoh.blogspot.com/feeds/posts/default
http://feeds.feedburner.com/MattFlynnsIdentityManagementBlog
http://feeds.feedburner.com/blogspot/CqwP?format=xml
https://nakedsecurity.sophos.com/feed/
http://infosecisland.com/rss.html
http://robert.penz.name/feed/
https://blog.trailofbits.com/feed/
http://blog.uncommonsensesecurity.com/feeds/posts/default
https://www.veracode.com/blog/feed
https://labsblog.f-secure.com/feed/
http://feeds.danielmiessler.com/danielmiessler
https://feeds.feedblitz.com/thesecurityledger
https://zeltser.com/feed/
```
You can also download the url file from [here](/downloads/urls). Just place this file in your ~/.newsbeuter folder. Watch out that you will lose your other feeds if you overwrite a currently existing url file.

For Flym, you can either add each RSS feed one by one (if you have a lot of time to kill), or create an OPML file which contains all the RSS feeds and import it in Flym. For this create a file called *the-ultimate-security-rss-feed-compilation.opml* with this content:

``` xml
<?xml version="1.0"?>
<opml version="1.0">
  <head>
    <title>newsbeuter - Exported Feeds</title>
  </head>
  <body>
    <outline type="rss" xmlUrl="https://www.us-cert.gov/ncas/current-activity.xml" htmlUrl="https://www.us-cert.gov/ncas/current-activity.xml" title="US-CERT Current Activity"/>
    <outline type="rss" xmlUrl="http://krebsonsecurity.com/feed/" htmlUrl="http://krebsonsecurity.com" title="Krebs on Security"/>
    <outline type="rss" xmlUrl="https://www.schneier.com/blog/atom.xml" htmlUrl="https://www.schneier.com/blog/" title="Schneier on Security"/>
    <outline type="rss" xmlUrl="http://www.darkreading.com/rss_simple.asp" htmlUrl="http://www.darkreading.com" title="Dark Reading: "/>
    <outline type="rss" xmlUrl="http://www.cio.com/index.rss" htmlUrl="http://www.cio.com" title="CIO"/>
    <outline type="rss" xmlUrl="http://feeds.pcmag.com/Rss.aspx/SectionArticles?sectionId=25442" htmlUrl="http://www.pcmag.com" title="PCMag.com Latest Articles"/>
    <outline type="rss" xmlUrl="https://www.wired.com/category/security/feed/" htmlUrl="https://www.wired.com" title="WIRED"/>
    <outline type="rss" xmlUrl="http://www.forbes.com/sites/firewall/feed/" htmlUrl="http://www.forbes.com/sites/firewall/" title="The Firewall - the world of security - Forbes"/>
    <outline type="rss" xmlUrl="http://www.techrepublic.com/rssfeeds/topic/security/" htmlUrl="http://www.techrepublic.com/rssfeeds/topic/security/" title="Security on TechRepublic"/>
    <outline type="rss" xmlUrl="http://www.zdnet.com/blog/rss.xml" htmlUrl="http://www.zdnet.com/" title="Latest blogs for ZDNet"/>
    <outline type="rss" xmlUrl="http://news.hitb.org/rss.xml" htmlUrl="http://news.hitb.org/" title="HITBSecNews - Keeping Knowledge Free for Over a Decade"/>
    <outline type="rss" xmlUrl="https://threatpost.com/feed/" htmlUrl="https://threatpost.com" title="Threatpost | The first stop for security news"/>
    <outline type="rss" xmlUrl="http://www.itsecurityguru.org/feed/" htmlUrl="http://www.itsecurityguru.org" title="IT SECURITY GURU"/>
    <outline type="rss" xmlUrl="http://itsecurity.co.uk/feed/" htmlUrl="http://itsecurity.co.uk" title="ITsecurity"/>
    <outline type="rss" xmlUrl="https://dankaminsky.com/feed/" htmlUrl="https://dankaminsky.com" title="Dan Kaminsky's Blog"/>
    <outline type="rss" xmlUrl="http://emergentchaos.com/feed" htmlUrl="http://emergentchaos.com" title="Emergent Chaos"/>
    <outline type="rss" xmlUrl="http://www.flyingpenguin.com/?feed=rss2" htmlUrl="http://www.flyingpenguin.com" title="flyingpenguin"/>
    <outline type="rss" xmlUrl="http://feeds.feedburner.com/inftoint" htmlUrl="https://www.elie.net/blog/" title="Elie on Internet Security and Performance"/>
    <outline type="rss" xmlUrl="https://www.grahamcluley.com/feed/" htmlUrl="https://www.grahamcluley.com" title="Graham Cluley"/>
    <outline type="rss" xmlUrl="https://blog.elearnsecurity.com/feed" htmlUrl="https://blog.elearnsecurity.com" title="eLearnSecurity Blog"/>
    <outline type="rss" xmlUrl="http://www.hackingarticles.in/feed/" htmlUrl="http://www.hackingarticles.in" title="Hacking Articles"/>
    <outline type="rss" xmlUrl="http://holisticinfosec.blogspot.com/feeds/posts/default" htmlUrl="http://holisticinfosec.blogspot.com/" title="HolisticInfoSec"/>
    <outline type="rss" xmlUrl="http://jeffsoh.blogspot.com/feeds/posts/default" htmlUrl="http://jeffsoh.blogspot.com/" title="JeffSoh on NetSec"/>
    <outline type="rss" xmlUrl="http://feeds.feedburner.com/MattFlynnsIdentityManagementBlog" htmlUrl="http://360tek.blogspot.com/" title="Matt Flynn's Security and Identity Blog"/>
    <outline type="rss" xmlUrl="http://feeds.feedburner.com/blogspot/CqwP?format=xml" htmlUrl="http://marcoramilli.blogspot.com/" title="Marco Ramilli's Blog"/>
    <outline type="rss" xmlUrl="https://nakedsecurity.sophos.com/feed/" htmlUrl="https://nakedsecurity.sophos.com" title="Naked Security"/>
    <outline type="rss" xmlUrl="http://infosecisland.com/rss.html" htmlUrl="https://infosecisland.infosecisland.com" title="Infosec Island Latest Articles"/>
    <outline type="rss" xmlUrl="http://robert.penz.name/feed/" htmlUrl="http://robert.penz.name" title="Robert Penz Blog"/>
    <outline type="rss" xmlUrl="https://blog.trailofbits.com/feed/" htmlUrl="https://blog.trailofbits.com" title="Trail of Bits Blog"/>
    <outline type="rss" xmlUrl="http://blog.uncommonsensesecurity.com/feeds/posts/default" htmlUrl="http://blog.uncommonsensesecurity.com/" title="Uncommon Sense Security"/>
    <outline type="rss" xmlUrl="https://www.veracode.com/blog/feed" htmlUrl="https://www.veracode.com/blog" title=""/>
    <outline type="rss" xmlUrl="https://labsblog.f-secure.com/feed/" htmlUrl="https://labsblog.f-secure.com" title="News from the Lab"/>
    <outline type="rss" xmlUrl="http://feeds.danielmiessler.com/danielmiessler" htmlUrl="https://danielmiessler.com/" title="Daniel Miessler : infosec | technology | humans"/>
    <outline type="rss" xmlUrl="https://feeds.feedblitz.com/thesecurityledger" htmlUrl="https://securityledger.com" title="The Security Ledger"/>
    <outline type="rss" xmlUrl="https://zeltser.com/feed/" htmlUrl="https://zeltser.com" title="Lenny Zeltser"/>
  </body>
</opml>
```

You can also simply download the OPML file [here](/downloads/the-ultimate-security-rss-feed-compilation.opml).

Enjoy being up to date!
