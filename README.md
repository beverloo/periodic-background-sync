# Periodic Background Sync

**Written**: 2019-03-26<br/>
**Updated**: 2019-03-27

Periodic Background Sync is a method that enables web applications to periodically synchronize data
in the background, building on the [Background Sync](https://wicg.github.io/BackgroundSync/spec/)
specification that defines a method for one-off synchronization.

The [original explainer](https://github.com/WICG/BackgroundSync/blob/master/explainer.md#periodic-synchronization-in-design)
included a thorough exploration of this capability, but it neither got specified, nor ever shipped
in a browsing engine. This was caused by a combination of low interest from both developers and
other browser engines, and high complexity due to an unclear permission model. We believe the
situation has changed since.

If this proposal stands, we plan to extend the [Background Sync](https://wicg.github.io/BackgroundSync/spec/)
specification to describe this ability as well, and move the document to the standardization track.

## Use cases
  Consider a website that's offline enabled. It uses service workers to provide an almost
  instantaneous loading experience. However, it may have to show stale content and switch it out
  shortly after the user visits the site, once it has downloaded the latest content.
  This app will benefit from updating its state and content when the device has network
  connectivity, *before* the user navigates to its web page. This way, it can delight its users by
  presenting the most up to date content, right away at launch time.

  Here are two main types of updates that are beneficial if done opportunistically:
  1. Updating state:
     This is the data required for the correct functioning of the app. Examples:
     a. Updated search index for a search app.
     b. Synchronized game scores for the user and their friends for a gaming app.
     c. Updated icons after a UI overhaul of a web app.

  2. Updating content:
     Periodic content producers can push content to user devices, to be consumed at a later, more
     convenient time. Examples:
     a. Fresh articles from news sites.
     b. List of badges the user has earned, in a fitness app.
     c. List of new songs from their favorited artists, in a songs app, to be downloaded using
        Background Fetch if the user clicks on "Download".

  Currently, the Push API enables this, but it requires setting up a server that speaks the Web
  Push protocol, and is GDPR compliant. Additional tasks like resource management add complexity to
  this approach. Periodic Background Sync offers a simpler, more accessible solution.

## Why do we care?
  * todo

## Goals
  * todo

## Non-goals
  * todo

# Example code

Please see [WebIDL.md](WebIDL.md) for the proposed WebIDL.

## Requesting a periodic sync
```javascript
// index.html

navigator.serviceWorker.ready.then(registration => {
  registration.periodicSync.register('get-latest-news', {
    // Minimum interval at which the sync may fire.
    minInterval: 24 * 60 * 60 * 1000,
  });
});
````

## Responding to a periodic sync event
```javascript
// service_worker.js

self.addEventListener('periodicsync', event => {
  if (event.tag == 'get-latest-news') {
    event.waitUntil(fetchAndCacheLatestNews());
  }
});
```

# Security and Privacy

TODO

# Design decisions

## Separate interface or extending one-shot Background Sync?
We think that the use cases for periodic background sync are sufficiently different from the use
cases for one-shot background sync to justify having a separate event in the developer's service
worker: synchronizing data in response to a user action versus opportunistically refreshing
content. In addition, there are different timing guarantees: one-shot sync has to run as soon as
possible, where the browser has the final timing decision for periodic sync, as long as it
honors the requested `minInterval` between two consecutive events.

The interfaces mimic regular Background Sync, but substitute `navigator.sync` with
`navigator.periodicSync` for symmetry with the event that will be fired in the service worker.

# References and acknowledgements

  * [Background Sync](https://wicg.github.io/BackgroundSync/spec/)
  * [Push API](https://w3c.github.io/push-api/)

Many thanks to the authors of the original explainer for starting the project, and to @mugdhalakhani
for picking periodic sync up again. Thanks to @jakearchibald and @rayankans for their ideas, input
and discussion.
