<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Lua tracker**](Lua-tracker-setup)

## Contents

- 1. [Overview](#overview)  
- 2. [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)  
  - 2.2 [Dependencies](#dependencies)  
  - 2.3 [Documented environments](#documented-envs)  
- 3. [Setup](#setup)
  - 3.1 [LuaRocks](#luarocks)
  - 3.2 [Debian/Ubuntu](#deb-ubuntu)
  - 3.3 [Other environments](#other-envs)

<a name="overview" />
## 1. Overview

The [Snowplow Lua Tracker](https://github.com/snowplow/snowplow-lua-tracker) lets you add analytics to your [Lua] [lua]-based applications, Lua web servers/frameworks, or from the Lua scripting layer within your games or apps.

The tracker should be relatively straightforward to setup if you are familiar with Lua development.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />
## 2. Integration options

### 2.1 Tracker compatibility

As a lightweight, easily-embeddable scripting language, Lua is available in a huge number of different computing environments and platforms, from [World of Warcraft] [wow] through [OpenResty] [openresty] to [Adobe Lightroom] [lightroom].

To make the Snowplow Lua Tracker work out-of-the-box with as many different Lua programs as possible, we have tried to:

1. Minimize external dependencies
2. Provide setup instructions for the most popular Lua environments

[Back to top](#top)

<a name="dependencies" />
### 2.2 Dependencies

To make the Snowplow Lua Tracker work with as many different Lua programs as possible, we have tried to keep external dependencies to a minimum. There is only one external dependency currently:

* [LuaSocket] [luasocket] - network support for the Lua language. Required to send events to the Snowplow collector

All other dependencies are bundled with the Lua Tracker itself - see the [lib] [tracker-lib] for details.

[Back to top](#top)

<a name="documented-envs" />
### 2.3 Documented environments

Below we provide setup instructions for the following popular Lua environments:

1. **LuaRocks** is a popular deployment and management system for Lua modules
2. **Debian/Ubuntu** for developers working directly with Lua in a Linux environment
3. **Game environments** for game and app developers working with embedded Lua

We expect that the Snowplow Lua Tracker should work in other Lua environments too - please [let us know](Talk-to-us) how you get on with setting it up elsewhere.

[Back to top](#top)

<a name="setup" />
## 3. Setup

<a name="luarocks" />
### 3.1 LuaRocks

_Instructions to come._

<a name="deb-ubuntu" />
### 3.2 Debian/Ubuntu

If your Lua app is running on a Linux box, you will need to install Lua and LuaSocket. On Ubuntu:

    $ sudo aptitude install lua liblua5.1-socket2

On Debian:

    $ sudo apt-get install lua liblua5.1-socket2

Then download the Snowplow Lua Tracker like so:

    $ git clone git@github.com:snowplow/snowplow-lua-tracker.git

Copy the `/src/snowplow` folder into your own Lua app's library folder, something like:

    $ cp -r snowplow-lua-tracker/src/snowplow my-app/lib/

Now add the Snowplow Lua Tracker into your Lua app's `package.path`, like so:

```lua
package.path = './lib/?.lua;' .. './lib/snowplow/?.lua;' .. package.path
```

Done? Now read the [Lua Tracker API](Lua-Tracker) to start tracking events.

<a name="ios" />
### 3.3 Game environments

For any other Lua environment, you will first need to address the external dependency on LuaSocket. Here are instructions for some popular games and game SDKs which use Lua:

* [Corona SDK] [corona-sdk] - LuaSocket is bundled. See the [API documentation] [corona-sdk-socket] for details
* [cocos2d-x] [cocos2d-x] - the cocos2d-x forum has [instructions] [cocos2d-x-socket] for integrating LuaSocket
* [LÖVE] [love] - LuaSocket is bundled according to this [LÖVE tutorial] [love-socket]
* [World of Warcraft] [wow] - LuaSocket is bundled according to the [Lua Users Wiki] [wow-socket] 


LuaSocket taken care of? Next, copy the `/src/snowplow` folder into your own Lua app's library folder, something like:

    $ cp -r snowplow-lua-tracker/src/snowplow my-app/lib/

Now add the Snowplow Lua Tracker into your Lua app's `package.path`, like so:

```lua
package.path = './lib/?.lua;' .. './lib/snowplow/?.lua;' .. package.path
```

That's it! Now read the [Lua Tracker API](Lua-Tracker) to start tracking events.

[lua]: http://www.lua.org/

[wow]: http://www.wowwiki.com/Lua
[openresty]: http://openresty.org/
[lightroom]: http://www.adobe.com/devnet/photoshoplightroom.html

[luasocket]: http://w3.impa.br/~diego/software/luasocket/

[corona-sdk]: http://www.coronalabs.com/products/corona-sdk/
[corona-sdk-socket]: http://docs.coronalabs.com/api/library/socket/index.html

[cocos2d-x]: http://www.cocos2d-x.org/
[cocos2d-x-socket]: http://www.cocos2d-x.org/boards/11/topics/6348

[wow-socket]: http://lua-users.org/wiki/WorldOfWarcraft

[love]: https://love2d.org/
[love-socket]: https://love2d.org/wiki/Tutorial:Networking_with_UDP

[tracker-lib]: https://github.com/snowplow/snowplow-lua-tracker/tree/master/src/snowplow/lib