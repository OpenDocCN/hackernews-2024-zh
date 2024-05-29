<!--yml
category: 未分类
date: 2024-05-27 14:56:40
-->

# 10 Second Teleportation | Upollo Blog

> 来源：[https://upollo.ai/blog/10-second-teleportation](https://upollo.ai/blog/10-second-teleportation)

## The scenario

We started seeing some strange devices show up for some of our customers' users. These devices would access the same page the user just had around 10 seconds after they did, from somewhere quite different. 

## It’s obviously a VPN!

We looked at where these requests were coming from places such as AWS in the central US, data centers in California and similar locations around the world. This could be someone switching on a VPN we thought. 

But wait, these are different devices, they have none of the same cookies. If this were a VPN it would be the same device.

## It’s obviously a Virtual Desktop!

Seeing a machine hosted in a data center and with all the users we saw coming from these locations working for large institutions we thought maybe it is just a remote cloud desktop. 

We would expect a remote cloud desktop to be most likely Windows, to be comparatively speced to a real world desktop and to have similar behaviour to real desktop as well. 

What we found were user agents purporting to be from a range of devices including mobile devices, all only ever loading a single page without any existing state like cookies. TLS fingerprints and other factors told us that these were consistently using an old version of Chrome. 

So far the device doesn’t look like any legitimate user we had ever seen. It looked like automation of some kind as most users run a fairly up to date version of Chrome and don’t lie about their user agents. 

The behavior itself is also strange, how did it load these pages which were often behind an authwall without ever logging in or having auth cookies? That seems very odd. 

## It’s obviously some kind of security system!

We had pieced together that this was:

*   An automated system 
*   That somehow had the page content from a user
*   Would render and execute all scripts on that page as if it was that user
*   Would not have any of the cookies or other context a user would have
*   Worked on a range of devices
*   Worked on HTTPS traffic 
*   Only showed up randomly for users from large institutions 

This looks like a security system that randomly scans pages by grabbing the page contents, sending it to a render queue and then processing it. This is interesting as these pages can contain PII, financial details or other sensitive information and they are being processed on versions of Chrome seemingly tens of versions out of date, seemingly on a shared instance. 

We got [nerd sniped](https://xkcd.com/356/) at this point and wanted to know what this system was. It didn’t have any user agent data as that was always forged to match the users. It often ran out of AWS or other data centers and it was a number of organizations. 

Finally we saw it start popping up from a data center that wasn’t a public one, Palo Alto Networks. 

In the end after reading product page after product page, we couldn’t work out exactly what product it was. We are keen to find out so if you know,  drop us a line and let us know!

These are now safely excluded from showing up for our users and is just one of the many things Upollo does to ensure it is giving accurate and actionable information to our customers. 

We look forward to sharing more fun things from the internet soon. 

#### Join the Wave

Ready to revolutionize how you recognize opportunities? Sign up for the waitlist below and be among the first to experience its transformative power when it launches.

Thanks! We'll let you know when you're off the waitlist.

Oops! Something went wrong while submitting the form.

###### About the Author

Cayden Meyer

Founder & CEO

On a mission to help millions of businesses understand their users and grow faster!