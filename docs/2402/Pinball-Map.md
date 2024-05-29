<!--yml
category: 未分类
date: 2024-05-27 14:32:27
-->

# Pinball Map

> 来源：[https://pinballmap.com/](https://pinballmap.com/)

The Pinball Map website, API, and app make extensive use of many free and/or open source services (as well as paid services). The website uses [Ruby on Rails](https://rubyonrails.org/) along with open source gems. The app uses [React Native](https://reactnative.dev/) and likewise is aided by many open source packages.

[Dokku.](https://dokku.com/) We use Dokku to build and deploy Pinball Map! It is a free service, though we support it with a monthly donation.

[DigitalOcean](https://www.digitalocean.com/) hosts our server and Postgresql database. This is fully paid for via our Patreon supporters.

[Mapbox](https://www.mapbox.com/) is used for map tiles.

[Expo](https://expo.dev/) manages, among many things, the app development and build processes.

[Scout APM](https://scoutapm.com/) tracks performance monitoring for the website and API.

[Sentry](https://sentry.io/) is used for crash reporting on the app!

The geocoding requests are processed by [Google Maps](https://developers.google.com/maps) and [HERE Maps](https://developer.here.com/) and [Nominatim](https://nominatim.org/). We also use the Google Places API for location submissions.