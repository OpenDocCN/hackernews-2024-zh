<!--yml

category: 未分类

date: 2024-05-27 14:59:21

-->

# Don't use Google Analytics

> 来源：[https://jak2k.schwanenberg.name/post/google-analytics/](https://jak2k.schwanenberg.name/post/google-analytics/)

It's a bad idea to use Google Analytics on your website and it is even illegal. In this post I'll explain why and what you can use instead.

## Why you shouldn't use Google Analytics

### It's bad for your users

Google Analytics forces you to add a disturbing consent banner to your website. It also blocks the main thread and slows down your website. Google also violates the users privacy extremely by collecting way more data than necessary.

### It's illegal in Europe

Google Analytics tracks without consent and this is illegal in Europe. You can get fined for using Google Analytics without consent, as it violates the GDPR. [[1](https://www.wired.com/story/google-analytics-europe-austria-privacy-shield/), [2](https://www.androidpolice.com/google-analytics-gdpr-eu-violation/)]

### It gives Google too much power

Google already has a lot of power and you shouldn't give them even more. They already know a lot about you and your users, because they track on every website. The tracking script could also manipulate your website and you wouldn't even notice.

## What you can use instead

### Matomo

A popular, mature and open source analytics tool.

[Website](https://matomo.org/)

#### Pros

+   Free and open source

+   Self-hostable

+   GDPR compliant

+   Customizable & custom dashboards

+   Extensible with plugins (including your own)

+   No data limits

+   Exportable data

#### Cons

+   A bit old-fashioned UI

+   Many plugins are paid (but you can write your own)

#### Used by

### Plausible

A simple and easy to use analytics tool.

[Website](https://plausible.io/)

#### Pros

+   Free and open source

+   Self-hostable

+   GDPR compliant

+   Modern UI

+   Simple & easy to use

+   Option for public stats

#### Cons

+   Very minimalistic

+   No custom dashboards

#### Used by

### Posthog

A very powerful analytics tool.

[Website](https://posthog.com/)

#### Pros

+   Free and open source

+   Self-hostable

+   GDPR compliant

+   Modern UI

+   Very powerful

+   Customizable & custom dashboards

#### Cons

+   Complex

+   Limited in official hosted version

+   Dashboard doesn't work on mobile (& tablet)

#### Used by

### umami

Simple, fast and privacy-focused alternative to Google Analytics.

[Website](https://umami.is/)

#### Pros

+   Self-hostable (free forever)

+   GDPR compliant

+   Modern UI

+   Open source

+   Simple & easy to use

#### Cons

+   No custom dashboards

+   Requires some technical knowledge to self-host

#### Used by

## What I use

I use a matomo instance hosted by a German hosting provider. I get a weekly report via email and I can see the stats in the matomo dashboard. I also use the Google Search Console to see how my website performs in Google Search.

Thanks for reading! If you have any questions or comments, comment below. I'd love to hear from you! You may also fix errors or suggest changes in the [GitHub repo](https://github.com/Jak2k/website).

[Jak2k](https://jak2k.schwanenberg.name)
