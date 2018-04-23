# Neos Matomo Package

[![Version](https://poser.pugx.org/flowpack/neos-matomo/v/stable)](https://packagist.org/packages/flowpack/neos-matomo)
[![license](https://poser.pugx.org/flowpack/neos-matomo/license)](https://packagist.org/packages/flowpack/neos-matomo)

**Track visits of your Neos site with Matomo (Piwik) Open Analytics Platform!**

This package integrates the `Matomo Open Analytics Platform` into Neos and is also compatible with older `Piwik` installations.

**Features**
 + adds a Backend Module to your Neos instance which helps checking your configuration
 + adds a tab to the Property Inspector, which shows time, device, OS and browser related statistics collected by Matomo
 + adds a customizeable content element which allows visitors to opt-out of tracking in the frontend

Inspired by the packages [neos/neos-googleanalytics](https://github.com/neos/neos-googleanalytics) and [khuppenbauer/MapSeven.Piwik](https://github.com/khuppenbauer/MapSeven.Piwik).

Initially this package was called `portachtzig/neos-piwik`. Thanks to the creators @koernchen02 and @sarahRehbock for all their work! 

- - -
![matomo-logo](Documentation/Images/matomo.svg)   

> Matomo - Open Analytics Platform
> https://matomo.org/

- - -


## Requirements

+ **cURL php extension** for api calls
+ **Neos CMS** >= 3.3
+ **Matomo** >= 3.3
+ A **Matomo instance** that is reachable via **https**
+ A Neos Backend User with the Role **Neos.Neos:Administrator**

### Compatibility with older versions

The version 2 of this package is still compatible with Piwik instances
but might at some point not work anymore when Matomo gets a bigger update.

Always check that your tracking software is up-to-date and use the latest
releases of this package if possible.

- - -

## Installation

Run the following command

    $ composer require flowpack/neos-matomo

### Updating from `neos-piwik` to `neos-matomo`

Instead of requiring `portachtzig/neos-piwik` you should now require `flowpack/neos-matomo` in your `composer.json`.

Also if you use or override `Portachtzig.Neos.Piwik:StatsTabMixin` you'll have to change the naming  
in your own NodeType configurations to `Flowpack.Neos.Matomo:StatsTabMixin`. 

In your configuration file you have to change the path

    Portachtzig:
      Neos:
        Piwik:

to
    
    Flowpack:
      Neos:
        Matomo:
        
Also we added caching for the api requests. Therefore by default the statistics will be kept up to 5 minutes before 
a refresh occurs.

## Configuration
After the package has been installed, there will be an additional Backend Module in Neos, called "Matomo".
Depending on your current FLOW_CONTEXT you might want to flush the cache.

![matomo-logo](Documentation/Images/matomo-config.png)

To connect Neos with your Matomo installation you just have to enter some properties 
in your `Settings.yaml` to track your user's statistics. You can have the configuration in your site package
or add it during deployment.
You can also use the included backend module then to verify your configuration.

+ **host**
Enter your Matomo installations hostname without protocol. 

+ **host**
You should always have your Matomo installation configured with https, but you can change it for testing purposes.

+ **token_auth**
You have to enter a valid auth token of a Matomo user who has the `view` permissions for the configured site.

+ **idSite**
You have to enter the id of the site you configured in Matomo.

+ **apiTimeout**
You can change the default timeout of 10 seconds after which the backend will cancel requests to your
Matomo installation.

### This is an example of how it can look:

    Flowpack:
      Neos:
        Matomo:
          host: 'tracking.example.org'
          protocol: 'https'
          token_auth: 'abcdefg1234567890'
          idSite: 1
          apiTimeout: 10 
          cacheLifetimeByPeriod:
            year: 86400
            day: 3600

## Additional features

### Tracking Opt-Out content element

This package provides a small configurable iframe content element for Neos which allows users
to manually opt out of the tracking if "Do-Not-Track" is not enabled in their browser.
The content of the iframe is loaded from the configured tracking host.

You can adjust all settings that Matomo offers via their API.      

### API request caching

By default the requests to the Matomo API will be cached depending on the period of the stat that is being checked.
Some information is shown based on a period of a year. Here we use a default timeout of 1 day.
Other stats are for one to several days. Here the preset for the timeout is 1 hour.

The timeouts by period can be overriden in your `Settings.yaml`. See the example above.

You can override the cache backend settings of this package in your own `Caches.yaml`:

    FlowpackNeosMatomo_ApiCache:
      backend: Neos\Cache\Backend\FileBackend
      backendOptions:
        defaultLifetime: 300
        
And if you don't want caching at all, just use the backend `Neos\Cache\Backend\NullBackend` instead of the 
configured `FileBackend`. 

### Accessing the Matomo API

You can access the Matomo API through this package by injecting the `Flowpack\Neos\Matomo\Service\Reporting` class.

You can then use the public method `callAPI($methodName, $arguments = [], $useCache = true)` to call any API method 
of Matomo and you will get the json decoded response as array.
Some methods will need specific user permissions, if your tokens user doesn't have them a error message will be in the
array.

### Multi site compatibility

You can override the site id (and the other settings) via fusion like this:

    prototype(Flowpack.Neos.Matomo:TrackingCode) {
        idSite = Neos.Fusion:Case {
            myOtherSite {
                condition = ${site == 'myOtherSite'}
                renderer = 2
            }
    
            myDefaultSite {
                condition = ${true}
                renderer = 1
            }
        }
    }
    
Or read it from a sites property like this:

    prototype(Flowpack.Neos.Matomo:TrackingCode) {
        idSite = ${q(site).property('trackingId') ? q(site).property('trackingId') : this.settings.idSite}
    }
    
Then you even have a fallback to your settings if the field is left empty. 
This is useful when having several microsites which are managed from the backend.

## License

Neos Matomo Package is released under the GPL v3 (or later) license.

## Screenshots 

![visist per day / last week](Documentation/Images/demo-package-matomo.png)

Backend view with selected Matomo Panel in the Property Inspector

![visist per day / last week](Documentation/Images/visits_per_day_month.png)

visits per day in the last month 

![visist per day / last week](Documentation/Images/visits_per_day_week.png)

visits per day in the last week 

![hits all time](Documentation/Images/visits_hits_all_time.png)

all time visits and page views (hits) of the selected page

![visist per day / last week](Documentation/Images/visits_per_browser.png)

visits per browser (all time)

![visist per os ](Documentation/Images/visits_per_os.png)

visits per os (all time)

![visist per os ](Documentation/Images/visits_per_device_cat.png)

visits per device category (all time)

