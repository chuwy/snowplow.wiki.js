<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > Java Tracker

This page refers to version 0.1.0 of the Snowplow Java Tracker.

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
- 3. [Adding extra data](#add-data)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`trackScreenview()`](#screen-view)
  - 4.3 [`trackPageview()`](#page-view)
  - 4.4 [`trackEcommercetransaction()`](#ecommerce-transaction)
  - 4.5 [`trackStructuredevent()`](#struct-event)
  - 4.6 [`trackUnstructuredevent()`](#unstruct-event)
- 5 [Contracts](#contracts)
- 6 [Logging](#logging)

<a name="overview" />
## 1. Overview

The [Snowplow Java Tracker](https://github.com/snowplow/snowplow-java-tracker) allows you to track Snowplow events from your Java-based desktop and server apps, servlets and games. It supports JDK6+.

The tracker should be straightforward to use if you are comfortable with Java development; its API is modelled after Snowplow's [[Python Tracker]] so any prior experience with that tracker is helpful but not necessary.

[Back to top](#top)

<a name="init" />
## 2 Initialization

Assuming you have completed the [[Java Tracker Setup]] for your Java project, you are now ready to initialize the Java Tracker.

<a name="importing" />
### 2.1 Importing the module

Import the Java Tracker's classes into your Java code like so:

```java
import com.snowplowanalytics.snowplow.tracker.*;
```

That's it - you are now ready to initialize a TrackerC instance. 

[Back to top](#top)

<a name="create-tracker" />
### 2.2 Creating a Tracker

To instantiate a tracker in your code (can be global or local to the process being tracked) simply instantiate the `Tracker` interface with one of the following:

```java
TrackerC(String collector_URI, String namespace)

TrackerC(String collector_uri, String namespace, String app_id, String context_vendor, boolean base64_encode, boolean enable_contracts)
```

For example:

```java
Tracker t1 = new TrackerC("d3rkrsqld9gmqf.cloudfront.net", "Snowplow Java Tracker Test", "testing_app", "com.snowplow", true, true);
```

TODO: clarify each of the arguments.


## Comments

<a name="add-data" />
## 3. Adding extra data

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

The TrackerC class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`setUserId`](#set-platform)
* [`setPlatform`](#set-user-id)
* [`setScreenResolution`](#set-screen-resolution)
* [`setViewport`](#set-viewport)
* [`setColorDepth`](#set-color-depth)
* [`setTimezone`](#set-timezone)
* [`setLanguage`](#set-lang)

Here are some examples:

```java
t1.setUserID("Kevin Gleason"); 
t1.setLanguage("en-gb");
t1.setPlatform("cnsl");
t1.setScreenResolution(1260, 1080);
```

Current supported platforms include "pc", "tv", "mob", "cnsl", and "iot".

TODO: break each of these into its own sub-section, as in the Python Tracker docs.

<a name="events" />
## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Java Tracker at a glance:

| **Function**                                                        | **Description**                                        |
|--------------------------------------------------------------------:|:-------------------------------------------------------|
| [`trackScreenview()`](#screen-view)                               | Track the user viewing a screen within the application |
| [`trackPageview()`](#page-view)                                   | Track and record views of web pages.                   |
| [`trackEcommercetransaction()`](#ecommerce-transaction)           | Track an ecommerce transaction and its items           |
| [`trackStructuredevent()`](#struct-event)                             | Track a Snowplow custom structured event               |
| [`trackUnstructuredevent()`](#unstruct-event)                         | Track a Snowplow custom unstructured event             |

<a name="common" />
### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track_XXX()`, where `XXX` is the name of the event to track.

<a name="custom-contexts" />
### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a Java String in JSON format. dictionary object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```java
t1.trackPageview(String page_url, String page_title, String referrer, String context)
```

The `context` argument should consist of a `String` containing a JSON array of one or more contexts. The format of each individual context element is the same as for an [unstructured event](#unstruct-event).

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```json
{ 
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2.1.1",
  "data": {
    "movie_name": "Solaris", 
    "poster_country": "JP", 
    "poster_year": "1978"
  }
}
```

Note that even if there is only one custom context attached to the event, it still needs to be placed in an array.

Providing the contexts as a Java object (i.e. not as a `String` in JSON format) is not yet supported.

<a name="tstamp-arg" />
### 4.1.2 Optional timestamp argument

Not yet implemented.

<a name="return-value" />
### 4.1.3 Tracker method return values

To be confirmed.

<a name="screen-view" />
### 4.2 Track screen views with `trackScreenview()`

**Warning:** this feature is implemented in the Java tracker, but it is **not** currently supported in the Enrichment, Storage or Analytics stages in the Snowplow data pipeline. As a result, if you use this feature, you will log screen views to your collector logs, but these will not be parsed and loaded into e.g. Redshift to analyse. (Adding this capability is coming soon to Snowplow.)

Use `trackScreenview()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | TBC           | TBC                     |
| `id`         | Unique identifier for this screen   | TBC           | TBC                     |
| `context`    | Custom context for the event        | TBC           | TBC                     |

Example:

```java
t1.trackScreenview("HUD > Save Game", "screen23", null);
```

[Back to top](#top)

<a name="page-view" />
### 4.3 Track pageviews with `trackPageview()`

If you are using Java servlet technology or similar to serve webpages to a browser, you can use `trackPageview()` to track a user viewing a page within your app.

Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `page_url`   | The URL of the page                 | TBC           | TBC                     |
| `page_title` | The title of the page               | TBC           | TBC                     |
| `referrer`   | The address which linked to the page| TBC           | TBC                     |
| `context`    | Custom context for the event        | TBC           | TBC                     |

Example:

```java
t1.trackPageview("www.example.com", "example", "www.referrer.com", null);
```

[Back to top](#top)

<a name="ecommerce-transaction" />
### 4.4 Track ecommerce transactions with `trackEcommercetransaction()`

Use `trackEcommercetransaction()` to track an ecommerce transaction.

Arguments:

| **Argument**  | **Description**                      | **Required?** | **Validation**           |
|--------------:|:-------------------------------------|:--------------|:-------------------------|
| `order_id`    | ID of the eCommerce transaction      | TBC           | Non-empty string         |
| `total_value` | Total transaction value              | TBC           | Int or Float             |
| `affiliation` | Transaction affiliation              | TBC           | String                   |
| `tax_value`   | Transaction tax value                | TBC           | Int or Float             |
| `shipping`    | Delivery cost charged                | TBC           | Int or Float             |
| `city`        | Delivery address city                | TBC           | String                   |
| `state`       | Delivery address state               | TBC           | String                   |
| `country`     | Delivery address country             | TBC           | String                   | 
| `currency`    | Transaction currency                 | TBC           | String                   |
| `items`       | Items in the transaction             | TBC           | List                     |
| `context`     | Custom context for the event         | TBC           | List                     |

The `items` argument is a `List` of individual `Map<String, String>` elements representing the items in the e-commerce transaction. Note that `trackEcommercetransaction` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `items` `List`. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

These are the fields that can appear as elements in each `Map` element of the transaction item `List`:

| **Field**  | **Description**                     | **Required?** | **Validation**           |
|-----------:|:------------------------------------|:--------------|:-------------------------|
| `ti_id`    | Order ID                            | TBC           | TBC                      |
| `sku`      | Item SKU                            | TBC           | TBC                      |
| `price`    | Item price                          | TBC           | TBC                      |
| `quantity` | Item quantity                       | TBC           | TBC                      |
| `name`     | Item name                           | TBC           | TBC                      |
| `category` | Item category                       | TBC           | TBC                      |

Example of tracking a transaction containing two items:

```java
// Example to come, in the meantime here is the type signature:
t1.trackEcommercetransaction(String order_id, Double total_value, String affiliation, Double tax_value,Double shipping, String city, String state, String country, String currency, List<Map<String, String>> items, String context);
```

[Back to top](#top)

<a name="struct-event" />
### 4.5 Track structured events with `trackStructuredevent()`

Use `trackStructuredevent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**           |
|-------------:|:---------------------------------------------------------------  |:--------------|:-------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | Non-empty string         |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | Non-empty string         |
| `label`      | A string to provide additional dimensions to the event data      | No            | String                   |
| `property`   | A string describing the object or the action performed on it     | No            | String                   |
| `value`      | A value to provide numerical data about the event                | No            | Int or Float             |
| `context`    | Custom context for the event                                     | No            | List                     |

Example:

```java
t1.trackStructuredevent("shop", "add-to-basket", null, "pcs", 2, null);
```

[Back to top](#top)

<a name="unstruct-event" />
### 4.6 Track unstructured events with `trackUnstructuredevent()`

**Warning:** this feature is implemented in the Java Tracker, but it is **not** currently supported in the Enrichment, Storage or Analytics stages in the Snowplow data pipeline. As a result, if you use this feature, you will log unstructured events to your collector logs, but these will not be parsed and loaded into e.g. Redshift to analyse. (Adding this capability is coming imminently.)

Use `trackUnstructuredevent()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                                           | **Required?** | **Validation**          |
|---------------:|:----------------------------------------------------------|:--------------|:------------------------|
| `eventVendor`  | The vendor who authored the event. Deprecated; do not use | TBC           | TBC                     |
| `eventName`    | The name of the event. Deprecated; do not use             | TBC           | TBC                     |
| `dictInfo`     | The properties of the event                               | TBC           | TBC                     |
| `context`      | Custom context for the event                              | TBC           | TBC                     |

The `dictInfo` must be either a `String` or a `Map<String, Object>`.

If you supply a `String`, make sure that it is a valid JSON object containing two fields: `schema` and `data`. `data` is itself a JSON object containing the properties of the unstructured event. `schema` identifies the JSON schema against which `data` should be validated.

Example:

```java
// Example to come, in the meantime here is the type signature:
t1.trackUnstructuredevent(String eventVendor, String eventName, String dictInfo, String context);
```

If you supply a `Map<String, Object>`, make sure that this top-level contains your `schema` and `data` keys, and then store your `data` properties as a child `Map<String, Object>`.

Example:

```java
// Example to come, in the meantime here is the type signature:
t1.trackUnstructuredevent(String eventVendor, String eventName, Map<String, Object> dictInfo, String context);
```

For more on JSON schema, see the [blog post] [self-describing-jsons].

<a name="contracts" />
## 5. Contracts

Not yet documented.

<a name="logging" />
## 6. Logging

Not yet implemented.

[documentation]: https://gleasonk.github.io/Saggezza/JavaDoc/index.html

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/