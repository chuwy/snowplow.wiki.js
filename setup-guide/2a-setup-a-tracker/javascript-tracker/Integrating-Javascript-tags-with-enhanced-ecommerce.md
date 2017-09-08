<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**JavaScript Tracker**](Javascript-tracker-setup) > [Integrating JavaScript tags with enhanced ecommerce](Integrating-Javascript-tags-with-enhanced-ecommerce)

**Warning: This functionality depends on Snowplow JavaScript Tracker v2.6.0 or above**

__Please visit__ the [technical documentation](https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker#enhanced-ecommerce) to see example use of the Enhanced Ecommerce functions.

1. [Overview](#overview)
2. [Creating the Data Layer Variable](#variable)
3. [Creating the trigger](#overview)
4. [Writing the JavaScript](#script)
5. [Creating the tag](#tag)

<a name="overview" />

### 1. Overview

This guide will show you how to configure Google Tag Manager to load the Snowplow JavaScript Tracker and send enhanced ecommerce data to Snowplow as well as Google without changing any of your calls to `dataLayer.push`. We will assume that you have already implemented the GTM `dataLayer` for enhanced ecommerce as described in the [Enhanced Ecommerce (UA) Developer Guide][enhancedEcommerceDeveloperGuide].

We also assume that any ecommerce-related call to `dataLayer.push` which does not contain an "event" field is made before Google Tag Manager loads, as described [here](http://www.simoahava.com/analytics/ecommerce-tips-google-tag-manager/#tip1).

If you are sending very large ecommerce events containing lots of impressions, the size of your events may exceed Internet Explorer's maximum querystring size for GET requests. In this case we recommend configuring the tracker to use POST instead as described [here](https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker#post).

<a name="variable" />

### 2. Creating the Data Layer Variable

[[/setup-guide/images/enhanced-ecommerce/enhanced_ecommerce_variable.png]]

In the Variables tab, create a Data Layer Variable. Set the name of this variable to "ecommerce". This variable will hold all ecommerce-related data and will be updated when you call `dataLayer.push` with a JSON containing the key "ecommerce".

<a name="trigger" />

### 3. Creating the trigger

[[/setup-guide/images/enhanced-ecommerce/enhanced_ecommerce_trigger.png]]

The trigger will detect ecommerce data pushed into the data layer and cause the main tag to fire.

In the Triggers tab, Create a new trigger named "Enhanced Ecommerce". In the "Choose Event" section, choose "Custom Event". Set "Fire On" to something like the string `gtm.dom|checkout|checkoutOption|productClick|addToCart|removeFromCart|promotionClick|purchase` and check the "Use regex matching" box.

The regex should consist of "gtm.dom" together with every string which you set as a the value of the "event" key in the enhanced ecommerce objects you push to the data layer, separated by the "|" pipe character.

<a name="script" />

### 4. Writing the JavaScript

Your tag will fire both when the page loads and also every time an ecommerce event is pushed to the data layer.

When the page loads, the tag will load the Snowplow JavaScript Tracker, make the API calls necessary to set up tracking. If the data layer contains ecommerce data, like product impressions, the tag will also send that data to Snowplow.

Whenever ecommerce data is pushed to the data layer, the tag will fire again. It will not attempt to set up tracking again; instead it will send the ecommerce event to Snowplow.

The example script below will be used as the basis for your tag. There are some changes you should make to this script. In the example "SNOWPLOW_NAME_HERE" is used as the name of the Snowplow function. This is the global function used to make API calls to the Snowplow JavaScript Tracker. You should change this string to something unique so that if there is another Snowplow user on the page the namespaces will not collide. Similarly, you should change "MY_COOKIE_NAME" to a unique value. You should change "MY_COLLECTOR" to the URL of your Snowplow collector (minus the http/https scheme), for example "mycloudfrontsubdomain.cloudfront.net".

You can also customize the part of the tag between the comments containing "!!!". The example below creates a tracker instance, sets page pings to fire every 30 seconds, and sends a page view event. See the [[JavaScript Tracker]] page for more information on other tracking methods.

```html
<script type="text/javascript">

  // If this tag fires more than once (eg page view followed by ecommerce action),
  // do not re-initialise the tracker

  if (!window.SNOWPLOW_NAME_HERE) {
    ;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
    p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
    };p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
    n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.8.2/sp.js","SNOWPLOW_NAME_HERE"));

    // !!! Customizable section starts
    // Track page views, enable link clicks, and so on here
    // Structured events and self-describing events go here
    // These methods will not be called if the tag fires due to ecommerce events

    window.SNOWPLOW_NAME_HERE('newTracker', 'co', 'MY_COLLECTOR', {
      appId: 'MY_APP_ID',
      cookieName: 'MY_COOKIE_NAME'
    });

    window.SNOWPLOW_NAME_HERE('enableActivityTracking', 30, 30); // Ping every 30 seconds after 30 seconds
    window.SNOWPLOW_NAME_HERE('trackPageView');

    // !!! Customizable section ends
  }

  // Enhanced ecommerce

  var actions = [
    "click",
    "detail",
    "add",
    "remove",
    "checkout",
    "checkout_option",
    "purchase",
    "refund",
    "promo_click",
    "view"
  ];

  // Get only the latest fired event

  var lastEvent = dataLayer[dataLayer.length-1];
  var lastEcommerce = lastEvent.ecommerce;

  if (lastEcommerce) {                    // If latest event is EE
    sendEnhancedEcommerceEvent(lastEcommerce);
  } else if ({{event}} == 'gtm.js') {    // If EE data is sent in a message (no event property)
    var mergedEcommerce = {{ecommerce}};
    if (mergedEcommerce) {
      sendEnhancedEcommerceEvent(mergedEcommerce);
    }
  }

  function sendEnhancedEcommerceEvent(ecommerce) {
    var currencyCode = ecommerce.currencyCode;
    var relevantActions = [];
    if({{event}} == 'gtm.js') {
        for (var i = 0; i < actions.length; i++) {
            if (ecommerce[actions[i]]) {
                relevantActions.push(actions[i]);
            }
        }
    } else {
        switch ({{event}}) {
            case 'addToCart':
                relevantActions.push('add');
                break;
            case 'removeFromBasket':
                relevantActions.push('remove');
                break;
            case 'productImpression':
                relevantActions.push('view');
                break;
            case 'productClick':
                relevantActions.push('click');
                break;
            case 'promoClicked':
                relevantActions.push('promo_click');
                break;
            case 'checkout':
                relevantActions.push('checkout');
                break;
            case 'checkoutOption':
                relevantActions.push('checkout_option');
                break;
        }
    }

    if (ecommerce.impressions) {
        for (var j = 0; j < ecommerce.impressions.length; j++) {
            var impression = ecommerce.impressions[j];
            SNOWPLOW_NAME_HERE('addEnhancedEcommerceImpressionContext',
                impression.id,
                impression.name,
                impression.list,
                impression.brand,
                impression.category,
                impression.variant,
                impression.position,
                impression.price,
                currencyCode
            );
        }
    }

    if (ecommerce.promoView) {
        for (var l = 0; l < ecommerce.promoView.promotions.length; l++) {
            var promo = ecommerce.promoView.promotions[l];
            SNOWPLOW_NAME_HERE('addEnhancedEcommercePromoContext',
                promo.id,
                promo.name,
                promo.creative,
                promo.position,
                currencyCode
            );
        }
    }

    if (relevantActions.length === 0) {
        SNOWPLOW_NAME_HERE('trackEnhancedEcommerceAction', 'view');
    } else {
        for (var m = 0; m < relevantActions.length; m++) {
            var relevantAction = relevantActions[m];
            if (ecommerce[relevantAction].products) {
                for (var k = 0; k <
                    ecommerce[relevantAction].products.length; k++) {
                    var product = ecommerce[relevantAction].products[k];
                    SNOWPLOW_NAME_HERE('addEnhancedEcommerceProductContext',
                        product.id,
                        product.name,
                        product.list,
                        product.brand,
                        product.category,
                        product.variant,
                        product.price,
                        product.quantity,
                        product.coupon,
                        product.position,
                        currencyCode
                    );
                }
            }
            if (ecommerce[relevantAction].actionField) {
                var actionObject = ecommerce[relevantAction].actionField;
                SNOWPLOW_NAME_HERE('addEnhancedEcommerceActionContext',
                    actionObject.id,
                    actionObject.affiliation,
                    actionObject.revenue,
                    actionObject.tax,
                    actionObject.shipping,
                    actionObject.coupon,
                    actionObject.list,
                    actionObject.step,
                    actionObject.option,
                    currencyCode
                );
            }
            SNOWPLOW_NAME_HERE('trackEnhancedEcommerceAction', relevantAction);
        }
    }
}

  // End of enhanced ecommerce

</script>
```

<a name="tag" />

### 5. Creating the tag

[[/setup-guide/images/enhanced-ecommerce/enhanced_ecommerce_tag.png]]

In the Tags tab, create a new tag. Name it something like "Enhanced Ecommerce Pageview". In the "Choose Product" section, select "Custom HTML Tag". Copy and paste the JavaScript you wrote in the previous section into the textbox.

From the Fire On section, choose "More", then select the "Enhanced Ecommerce" trigger you created earlier and save.

[enhancedEcommerceDeveloperGuide]: https://developers.google.com/tag-manager/enhanced-ecommerce
