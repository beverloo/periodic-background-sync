# Periodic Background Sync

**Written**: 2019-03-26<br/>
**Updated**: 2019-04-17

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
  * Native apps currently have the ability to offer fresh content to users, even when they're 
  offline. This API will enable web apps to do so too.
  * Websites can already push notifications to update content, but the timing of those notifications
  is decided by the developer. This API enables the browser to decide on the timing, so it can be
  be more respectful of user's intents like 'Do Not Disturb', and prevent notification fatigue.
  In addition, this control on timing allows the browser to optimize resource usage and prevent
  resource abuse. For instance, the browser may decide how often each website should be allowed to
  run their tasks. It may also choose to only run periodic tasks when the device is sufficiently
  charged, or has a certain connectivity.

## Goals
* Enable a web app to run tasks periodically on network connectivity.

## Non-goals
* Triggering events at a specific time is an explicit non-goal. A more generalized alarms API can
enable that.
* Multiple periodic tasks per origin, with varying frequency. The browser decides the cadence of
  periodic sync tasks for each origin. An origin can thus register multiple periodic tasks, but the
  frequency decided by the browser for the tasks can be the same.

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
```

## Responding to a periodic sync event
```javascript
// service_worker.js

self.addEventListener('periodicsync', event => {
  if (event.tag == 'get-latest-news') {
    event.waitUntil(fetchAndCacheLatestNews());
  }
});
```

## Checking if a periodic sync task with a given tag is registered
```javascript
// index.html

navigator.serviceWorker.ready.then(registration => {
  registration.periodicSync.getTags().then(tags => {
    if (tags.includes('get-latest-news'))
      skipDownloadingLatestNewsOnPageLoad();
  });  
});
```

## Removing a periodic sync when the user signs out
```javascript
// index.html

navigator.serviceWorker.ready.then(registration => {
  registration.periodicSync.unregister('get-latest-news');
});
```

# Security and Privacy

One-shot Background Sync allows the web page’s logic to live a little longer (service worker
execution time) after the page has been closed by the user. Periodic Background Sync extends it to
potentially run forever, at regular periods, for a few minutes at a time. Here are some privacy
concerns:

* There might be no notification to the user, depending on the implementation.
* A periodic sync can be enabled while the user is connected to one network, and the sync event
can be fired later when they're connected to another network. This can cause inadvertent leakage 
of browsing history on an unintended network. This concern applies to other background tasks such as
one-shot Background Sync and Background Fetch as well.
* Location tracking: The user’s IP address can be revealed to the website every time the periodic
task is run. This risk is also present with any tasks run in a service worker, but Periodic Sync
allows persistent tracking. Note that this risk is also present with push messages,
but they require explicit user permission.

To mitigate these privacy concerns, the user agent can limit the number of times periodic tasks are
run for a site if the user isn't currently visiting the site.

An additional `periodic-background-sync` permission will be exposed through the Permissions API to
allow developers to query whether the API can be used.

# Design decisions

## Separate interface or extending one-shot Background Sync?
We think that the use cases for periodic background sync are sufficiently different from the use
cases for one-shot Background Sync to justify having a separate event in the developer's service
worker: synchronizing data in response to a user action versus opportunistically refreshing
content. In addition, there are different timing guarantees: one-shot Background Sync has to run as
soon as possible, where the browser has the final timing decision for periodic sync, as long as it
honors the requested `minInterval` between two consecutive events.

The interfaces mimic regular Background Sync, but substitute `navigator.sync` with
`navigator.periodicSync` for symmetry with the event that will be fired in the service worker.

# References and acknowledgements

  * [Background Sync](https://wicg.github.io/BackgroundSync/spec/)
  * [Push API](https://w3c.github.io/push-api/)

Many thanks to the authors of the original explainer for starting the project, and to @mugdhalakhani
for picking periodic sync up again. Thanks to @jakearchibald and @rayankans for their ideas, input
and discussion.
