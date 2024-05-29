<!--yml
category: 未分类
date: 2024-05-29 13:25:04
-->

# Avoid bot detectors with our new /unblock API for Browserless

> 来源：[https://www.browserless.io/blog/2024/02/26/unblock-api/?apcid=00620de59ffc742367908900&utm_campaign=unblock-api-announcement&utm_content=unblock-api-announcement&utm_medium=email&utm_source=ortto](https://www.browserless.io/blog/2024/02/26/unblock-api/?apcid=00620de59ffc742367908900&utm_campaign=unblock-api-announcement&utm_content=unblock-api-announcement&utm_medium=email&utm_source=ortto)

The cat-and-mouse between bots and WAFs requires increasingly sophisticates solutions.

Traditional methods often rely on "mocking" JavaScript APIs or setting Chrome flags, but these are becoming less reliable due to advances in protections from Cloudflare. Even plugins like `puppeteer-extra-plugin-stealth` fails to mask fundamental aspects of how Puppeteer effects a browser.

It's why we've taken a new deeper approach to humanizing traffic.

## When was the last time you loaded a browser at 800x600 pixels?

That's what Puppeteer does. Every time you run a script with Puppeteer, it creates a 800x600px browsers.

Even if you use `Page.setViewport()`, it will only change resize the browser once it's been instantiated at the default size. This behavior is just one of the many dead giveaways that the traffic is coming from a bot, not a human.

To hide fingerprints like these, our /unblock API runs modifies behavior at the CDP layer.

For this browser size example, the API creates the browser at a more typical 1920x1080 pixels. Only once that's done is Puppeteer allowed to connect to it. By feeding Chrome modified instructions with the Chrome DevTools Protocol, we're able to tidy away all the breadcrumbs like these which bot detectors pick up on.

## Launching Chrome like a human would

The aim of this /unblock API is to launch Chrome with settings that are what you'd expect on a normal computer. Bot detectors can't start filtering out traffic using a 1920x1080 browser, as that would start blocking authentic traffic.

While there will still be some amount of cat and mouse as we find increasingly subtle fingerprints, we expect that once changed that each one will permanently help with avoiding bot blockers.

## Ask and you shall receive (a site)

Whether you're looking to connect Puppeteer to an "unblocked" browser, or you're simply after cookies, our JSON-based API is designed to cater to a wide range of needs. This flexibility is crucial for developers with different setups.

The experience using the Unblock API is as follows:

1.  You provide a URL of the website that's difficult to access.
2.  We'll instantiate the browser leaving as few breadcrumbs as possible and bypass bot detection depending on mechanisms identified on the site.
3.  Once the site is accessed, the API will return the data of that site.
4.  Unless cookies content or screenshot are set to false in the POST’ed payload, it’ll generate those.
5.  If browserWSEndpoint is set to true, it’ll generate a custom one-time link for puppeteer or any other CDP-like library to reconnect to.
6.  It’ll return these values above in a JSON response.

You shouldn't need to setup stealth libraries, test chrome flags combinations, or implement bot-solving technologies. However, you might still want to use our usual [bypassing bot detection mechanisms](https://docs.browserless.io/Recipes/bypass-bot-detection) such as residential proxies.

## Staying ahead of bot detection mechanisms

Looking ahead, we commit to continuously improving. This means we'll constantly update and fixes for new detection mechanisms. We're always open to knowing what sites you're having a problem bypassing because we can use those as samples of sites we need to analyze and setup bypassing mechanisms on.

Some sites enforce Captchas 100% even on our real human computers, which means we're also working on supporting Captcha auto-solving soon.

## Try the /unblock API yourself

To use the API you'll need to use our new V2 service, which is currently only available for [Starter and Scale plans](https://www.browserless.io/pricing) and will be soon rolled out to clients on Enterprise offerings with dedicated fleets. That means the code becomes:

```
 curl --request POST \
  --url 'https://production-sfo.browserless.io/unblock?timeout=300000&token=YOUR-API-TOKEN' \
  --header 'content-type: application/json' \
  --data '{
  "url": "https://example.com",
  "browserWSEndpoint":true
}' 

```

Or use production-lon.browserless.io for the London server.

The API is now publicly available for our users. Sign up for a 7-day trial of browserless to test it with your existing Puppeteer scripts.

[7-Day Browserless Trial](https://www.browserless.io/pricing)

*Note: as this process is resource intensive, it will use extra Units, and is not available for our legacy free tier users.*