[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 1: setup a Collector**](Setting-up-a-Collector) > [**Clojure collector setup**](setting-up-the-clojure-collector) > [[Set your tracker to point at the Clojure collector endpoint]]

Now that you've set up your Clojure collector, you need to configure your tracker to send event data to it.

Assuming you're using the [JavaScript Tracker][javascript-tracker], you'll need to modify your JavaScript tracking tags to set the correct end point.

If you are using version 2.0.0 or later of the JavaScript Tracker, then just replace `'{{CLOUDFRONT DOMAIN}}.cloudfront.net'` with your Clojure collector's url when creating a new tracker instance. For instance, you might replace

```javascript
snowplow_name_here("newTracker", "tracker1", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});
```

with

```javascript
snowplow_name_here("newTracker", "tracker1", "collector.snplow.com", {
  appId: "cfe23a",
  platform: "mob"
});

If you are using a version earlier than 2.0.0, you will also need to change the method you use from `setCollectorCf` to `setCollectorUrl` as explained below.

The standard tracking tags for a version of the JavaScript Tracker that's earlier than 2.0.0 look as follows:

```html
<!-- Snowplow starts plowing -->
<script type="text/javascript">
window._snaq = window._snaq || [];

window._snaq.push(['setCollectorCf', '{{CLOUDFRONT DOMAIN}}']);
window._snaq.push(['enableLinkTracking']);
window._snaq.push(['trackPageView']);

(function() {
var sp = document.createElement('script'); sp.type = 'text/javascript'; sp.async = true; sp.defer = true;
sp.src = ('https:' == document.location.protocol ? 'https' : 'http') + '://d1fc8wv8zag5ca.cloudfront.net/0.9.1/sp.js';
var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(sp, s);
})();
 </script>
<!-- Snowplow stops plowing -->
```

The line...

```javascript
window._snaq.push(['setCollectorCf', '{{CLOUDFRONT DOMAIN}}']);
```

...is the one that sets the end point, and assumes that you're using the Cloudfront collector rather than the Clojure collector.

To set your Clojure collector as the end point, remove that line and replace it with:

```javascript
window._snaq.push(['setCollectorUrl', '{{COLLECTOR URL}}']);
```

Note that the URL your enter must **exclude** `http://` or `https://`. In our case, the url would be: `collector.snplow.com`.

Also, you need to tell the JavaScript that it doesn't need to send user IDs to the collector - this is because your Clojure collector is now setting user IDs. You do this like so:

```javascript
window._snaq.push(['attachUserId', false]);
```

So, the complete tag will look like this:

```html
<!-- Snowplow starts plowing -->
<script type="text/javascript">
window._snaq = window._snaq || [];

window._snaq.push(['setCollectorUrl', 'collector.snplow.com']);
window._snaq.push(['attachUserId', false]);
window._snaq.push(['enableLinkTracking']);
window._snaq.push(['trackPageView']);

(function() {
var sp = document.createElement('script'); sp.type = 'text/javascript'; sp.async = true; sp.defer = true;
sp.src = ('https:' == document.location.protocol ? 'https' : 'http') + '://d1fc8wv8zag5ca.cloudfront.net/0.9.1/sp.js';
var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(sp, s);
})();
 </script>
<!-- Snowplow stops plowing -->
```

Next: [[update the EmrEtlRunner configuration YAML file]]

[javascript-tracker]: javascript-tracker-setup