Basecamp API Authentication
===========================

> Speak, friend, and enter.

All Basecamp' products API requests can be authenticated by passing along an OAuth 2 token. API keys are supported as well for some apps.

Basic Auth (Basecamp 2 only)
------------------------------

To hit the ground running with the Basecamp 2's API, just use HTTP Basic authentication with your own login info:

```shell
curl -u username:password -H 'User-Agent: MyApp (yourname@example.com)' https://basecamp.com/999999999/api/v1/projects.json
```
_Never ask a user of your application for their username or password!_

You're free to use your own username & password to access your own account and
to get started with the API. OAuth 2 is a simple protocol, but it's yet another
speed bump to getting an integration off the ground.

Your HTTP client software includes built-in support for HTTP Basic authentication.
Just provide your username and password.

API Key
-------

Basecamp Classic, Highrise, Campfire, and Backpack allow usage of an API token to authenticate:

```shell
curl -u 605b32dd:X https://sample.campfirenow.com/room/1.xml
```

Remember that anyone who has your authentication token can see and change everything you have access to. So you want to guard that as well as you guard your username and password. If you come to fear that it has been compromised, just change your regular password and the authentication token will change too.

OAuth 2
-------

For a full app integration, you wouldn't want to get into the business of asking
customers for their passwords -- or storing them! -- so we offer a simple way to
ask a user for access to their account. You get an API access token back without
ever having to see their password or ask them to copy/paste an API key.

1. [Grab an OAuth 2 library](http://oauth.net/code/).
2. Register your app at [launchpad.37signals.com/integrations](https://launchpad.37signals.com/integrations). You'll be assigned a `client_id` and `client_secret`. You'll need to provide a `redirect_uri`: a URL where we can send a verification code. Just enter a dummy URL like `http://myapp.com/oauth` if you're not ready for this yet.
3. Configure your OAuth 2 library with your `client_id`, `client_secret`, and `redirect_uri`. Tell it to use `https://launchpad.37signals.com/authorization/new` to request authorization and `https://launchpad.37signals.com/authorization/token` to get access tokens.
4. Try making an authorized request to `https://launchpad.37signals.com/authorization.json` to dig in and test it out! Check out the [Get authorization](#get-authorization) endpoint for a description of what this returns.


OAuth 2 from scratch
--------------------

If you're going bare-metal and developing your own OAuth 2 client, you have a bit more work to do.

**TL;DR** request access, receive a verification code, trade it for an access token.

The typical flow for a web app:

1. Your app requests authorization by redirecting your user to Launchpad:

        https://launchpad.37signals.com/authorization/new?type=web_server&client_id=your-client-id&redirect_uri=your-redirect-uri

2. We authenticate their Basecamp ID and ask whether it's ok to give access to your app. [Here's an example of what this screen looks like](https://launchpad.37signals.com/authorization/new?type=web_server&client_id=0bf18204f5a28003bf7b9abb7e1db5e649d86ef4&redirect_uri=moist%3A%2F%2Foauth)

3. We redirect the user back to your app with a time-limited verification code.

4. Your app makes a backchannel request to trade the verification code for an access token. We authenticate your app and issue an access token:

        POST https://launchpad.37signals.com/authorization/token?type=web_server&client_id=your-client-id&redirect_uri=your-redirect-uri&client_secret=your-client-secret&code=verification-code

5. Your app uses the token to authorize API requests to any of the Basecamp ID's accounts. Set the Authorization request header:

        Authorization: Bearer YOUR_OAUTH_TOKEN

6. To get info about the Basecamp ID you authorized and the accounts you have access to, make an authorized request to `https://launchpad.37signals.com/authorization.json` (or `/authorization.xml`). (See

Implementation notes:

* Start by reading the [draft spec](https://tools.ietf.org/html/draft-ietf-oauth-v2-05)
* We implement draft 5.
* We support the `web_server` and `user_agent` flows, not the `client_credentials` or `device` flows.
* We return more verbose errors than what's given in the spec to help with client development.
* We issue refresh tokens. Use them to request a new access token when it expires (2 week lifetime, currently).

      POST https://launchpad.37signals.com/authorization/token?type=refresh&refresh_token=your-current-refresh-token&client_id=your-client-id&client_secret=your-client-secret


Get authorization
-----------------

* `GET https://launchpad.37signals.com/authorization.json`
* `GET https://launchpad.37signals.com/authorization.xml`

This endpoint requires the `Authorization` header and returns the following:

* A `expires_at` timestamp for when this token will expire, and you'll need to fetch a new one to authenticate requests
* An `identity`, which is **NOT** used for determining who this user is within a specific application. The `id` field should **NOT** be used for submitting data within any application's API. This field can be used to get a user's name and email address quickly, and the `id` field could be used for caching on a cross-application basis if needed.
* A list of `accounts` that this user has access to.

This endpoint should be first request made after you've obtained a user's authorization token via OAuth. You can pick which account to use for a given product, and then base where to make requests to from the chosen account's `href` field.

```json
{
  "expires_at": "2012-03-22T16:56:48-05:00",
  "identity": {
    "id": 9999999,
    "first_name": "Jason",
    "last_name": "Fried",
    "email_address": "jason@basecamp.com",
  },
  "accounts": [
    {
      "product": "bc3",
      "id": 99999999,
      "name": "Honcho Design",
      "href": "https://3.basecampapi.com/99999999",
      "app_href": "https://3.basecamp.com/99999999"
    },
    {
      "product": "bcx",
      "id": 88888888,
      "name": "Wayne Enterprises, Ltd.",
      "href": "https://basecamp.com/88888888/api/v1",
      "app_href": "https://basecamp.com/88888888"
    },
    {
      "product": "campfire",
      "id": 44444444,
      "name": "Acme Shipping Co.",
      "href": "https://acme4444444.campfirenow.com",
      "app_href": "https://acme4444444.campfirenow.com"
    }
  ]
}
```

### On authenticating users via OAuth

**We don't recommend using the OAuth API to authenticate users in third party services**. For example, to implement a button *"Login with Basecamp"* similar to others like *"Login with Google"* or *"Login with GitHub"*. The reason is that we don't verify email addresses. As a result, an attacker could gain access to these services using email addresses they don't own.

Who am I?
---------

So you've picked the product, have the `href` to request to, and now what? Depending on what your API integration needs, you may need to know who the current user is within that product. For example, if you were working with the [Basecamp API](https://github.com/Basecamp/bcx-api) this is how you would obtain the `id` field to [submit a todo](https://github.com/Basecamp/bcx-api/blob/master/sections/todos.md#create-todo) assigned to the current user.

Here are links to the endpoints in our products that you'll need to hit:

* [Basecamp 3: Get person](https://github.com/basecamp/bc3-api/blob/master/sections/people.md#get-person)
* [Basecamp 2: Get person](https://github.com/basecamp/bcx-api/blob/master/sections/people.md#get-person)
* [Basecamp Classic: Current person](https://github.com/basecamp/basecamp-classic-api/blob/master/sections/people.md#current-person)
* [Highrise: Get myself](https://github.com/basecamp/highrise-api/blob/master/sections/users.md#get-myself)
* [Campfire: Get self](https://github.com/basecamp/campfire-api/blob/master/sections/users.md#get-self)
* [Backpack: Current user](https://github.com/basecamp/backpack-api/blob/master/sections/users.md#current-user)
