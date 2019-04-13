---
category: posts
layout: single
mainTopic: 'programming'
title: "User impersonation in SaaS with Ember.js and Ruby on Rails"
---

When developing SaaS, it's very convenient, and arguably even urgent, to be able to log in as your users and see exactly what they see. This is especially important when debugging an issue or enabling your support team to check for bugs without looking into error logs.

I'm going to show you how you can do it using the following technologies:

- Ember.js (with [Ember Simple Auth](https://ember-simple-auth.com/) add-on)
- Ruby on Rails (with [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) gem for OAuth2)

## How should this work?

When the client is exchanging username and password for an access token, we call this OAuth 2.0 Password Grant flow.

If we wanted to impersonate a user, we need to create a new token by bypassing the part where password is used and ask Doorkeeper to issue a new token for the provided user, directly.

Then, when we have the token, we need a way to pass it to the Ember app that uses the Ember Simple Auth add-on.

## Creating a new authenticator

We take the existing `oauth2` authorizer and extend its `authenticate` method. Originally it accepts username and password, and gets the token by issuing a request to your OAuth endpoint.

We modify this so we can pass `tokenData` directly, which logs us in:

```javascript
// app/authenticators/oauth2-custom-inject.js

import OAuth2 from "./oauth2";
import { isEmpty } from "@ember/utils";
import RSVP from "rsvp";

export default OAuth2.extend({
  // tokenData is { access_token: 'asd', refreshToken: 'asdf', ... }
  authenticate(tokenData) {
    return new RSVP.Promise(resolve => {
      const expiresAt = this._absolutizeExpirationTime(tokenData["expires_in"]);
      this._scheduleAccessTokenRefresh(
        tokenData["expires_in"],
        expiresAt,
        tokenData["refresh_token"]
      );

      if (!isEmpty(expiresAt)) {
        tokenData = Object.assign(tokenData, { expires_at: expiresAt });
      }

      resolve(tokenData);
    });
  }
});
```

We also need to add the accompanying authorizer:

```javascript
// app/authorizers/oauth2-custom-inject.js

import OAuth2 from "./oauth2";
export default OAuth2.extend();
```

Now, where should we call this new custom authenticator? One way is to do it in the `application` route, where we receive the token data in a form of a query parameter when the app loads.

```javascript
// app/routes/application.js

async beforeModel(transition) {
  const uit = transition.queryParams.userImpersonationToken;
  if (uit) {
    await this.tryLoginWithProvidedToken(uit);
  }

  return this._loadCurrentUser();
},
async tryLoginWithProvidedToken(uit) {
  return new RSVP.Promise(async resolve => {
    const tokenObj = JSON.parse(decodeURIComponent(uit));
    await this.session.authenticate(
      "authenticator:oauth2-custom-inject",
      tokenObj
    );
    resolve();
  });
}
```

This enables us to send the token data to our Ember app via `userImpersonationToken` query parameter.

## Generating a token for any user

Use the following code for generating a URL that gets you logged in as any user.

```ruby
user = User.find(user_id)
access_token = Doorkeeper::AccessToken.create!(resource_owner_id: user.id, scopes: 'all', expires_in: Doorkeeper.configuration.access_token_expires_in,
use_refresh_token: Doorkeeper.configuration.refresh_token_enabled?)

token_response = Doorkeeper::OAuth::TokenResponse.new(access_token).body.to_json

user_impersonation_url = "#{Rails.configuration.web_app_url}/?userImpersonationToken=#{token_response}"
```

A good place to use this code and redirect to the `user_impersonation_url` is in your admin app, for example Rails Active Admin.

## Things to watch out for

1. This keeps you logged in with another user when you reopen the browser. You might want to find a way to nullify the session after you are done impersonating.

2. If you have some kind of user behaviour tracking set up (e.g. Mixpanel), you might want to disable it while you are impersonating.





