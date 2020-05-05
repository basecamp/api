Basecamp APIs
=============

Looking to integrate with a Basecamp product? We've got everything you need here. From sample code to detailed API documentation, you'll be up and running in no time. (We just encourage you to have [realistic expectations](#realistic-expectations) regarding the continued evolution of these APIs).


Where do I start?
-----------------

Want to get started with API integration? Here's a quick check list:

1. [Register your app](https://launchpad.37signals.com/integrations/new) or [update your already registered app](http://launchpad.37signals.com/integrations) to access more products.
2. Review our [Terms of Service](https://basecamp.com/about/policies/terms), in particular the **Services Adaptations and API Terms** section and our [Use Restrictions](https://basecamp.com/about/policies/abuse).
3. Review the [brand guidelines](#brand-guidelines) for rules on naming and marketing your app.
4. Read up on how to [authenticate your app](#authentication) with our products.
5. Peruse the [API docs](#products) for the product you need to work with.
6. Have a question? Post it on [StackOverflow](http://stackoverflow.com/questions/ask) tagged with the product you're working on (for example, `basecamp`) or [open up a support ticket](https://basecamp.com/support).

API Documentation
-----------------

Need to look up the API documentation for a product? There's a repo for each:

* [Basecamp Classic API](https://github.com/basecamp/basecamp-classic-api)
* [Basecamp 2 API](https://github.com/basecamp/bcx-api)
* [Basecamp 3 API](https://github.com/basecamp/bc3-api)

Authentication
--------------

Logging your users into our products is supported through OAuth 2. Several apps support login via API tokens and HTTP Basic Authentication as well.

We have a [detailed guide](https://github.com/basecamp/api/blob/master/sections/authentication.md) for authenticating your users with our apps.


Brand Guidelines
----------------

Have questions on how to name and publicize your app? Our [brand guidelines](https://github.com/basecamp/api/blob/master/sections/brand_guidelines.md) have some basic answers for you. If you have a more detailed question, just [open up a support ticket](http://help.basecamp.com/tickets/new) and we'll sort it out!


Official logo
-------------

Need our logo for your marketing page? You can download the [Basecamp official logo](https://github.com/basecamp/api/tree/master/logos) in the repo.


Realistic expectations
----------------------

The Basecamp APIs aren't intended to be complete models of everything you can do in all our applications. The truth is that less than half a percent of our end-users take advantage of applications that integrate with Basecamp or write their own integrations. Such low usage means that continuing to update the API to stay in sync with all new features is not a high priority at Basecamp. When we do work for that less than that one half of a percent of users, we're not helping that other 99.5%.

We still do make upgrades and improvements from time to time, and if you find a bug in the APIs we do have exposed, please [open a support ticket](https://basecamp.com/support). Also, feel free to fork these docs and send a pull request with improvements!
