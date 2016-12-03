---
category: posts
layout: single
mainTopic: 'ember'
title: "Transforming JSON payload to include object root with RESTSerializer in Ember.js"  
---

If you find yourself using Ember Data combined with a backend that doesn’t conform to JSON API specs, you just might be in a bit of a trouble. As always, Ember comes with a few assumptions.

In this case, about how your JSON responses from the HTTP server should look like. Sometimes you don’t have the luxury to be able to change the format of how your backend is responding or already have API consumers depending on current response format, meaning you have to deal with this on the client side.

In my case the issue was the server endpoints were returning simple arrays with objects, like this:

```
[
  {
    "id": 1,
    "name": “Kobosil"
  },
  {
    "id": 2,
    "name": “Efdemin"
  }
]
```

Most commonly used adapter in Ember.js to communicate with HTTP server by transmitting JSON is RESTAdapter. This adapter expects the JSON payload should be an object that contains an object root type element. While we could roll out our very own customized adapter to support this case, let us rather just patch (extend) RESTSerializer a bit and keep it’s handy functionalities.  

So in order for our Ember application to properly serialize this response, the response should look like this:

```javascript
{
  "artists":
    [
      {
        "id": 1,
        "name": “Kobosil"
      },
      {
        "id": 2,
        "name": “Efdemin",
      }
    ]
}
```

Since we decided we don’t want to touch the way our server responds, let’s modify the response object to add the type root element right before Ember Data internals start serializing things into Ember objects:

```javascript
//app/serializers/application.js:

import DS from 'ember-data';

export default DS.RESTSerializer.extend({
  extractSingle: function extractSingle(store, primaryType, payload, recordId) {
      var newPayload, typeKey;
      typeKey = primaryType.typeKey;
      newPayload = {};
      if (payload[typeKey] && typeof payload[typeKey] === 'object') {
        newPayload = payload;
      } else {
        newPayload[typeKey] = payload;
      }
      return this._super(store, primaryType, newPayload, recordId);
    },
    extractArray: function extractArray(store, primaryType, payload) {
      var newPayload, pluralTypeKey;
      pluralTypeKey = Ember.Inflector.inflector.pluralize(primaryType.typeKey);
      newPayload = {};
      if (payload[pluralTypeKey] && typeof payload[pluralTypeKey] === 'object') {
        newPayload = payload;
      } else {
        newPayload[pluralTypeKey] = Array.isArray(payload) ? payload : [payload];
      }
      return this._super(store, primaryType, newPayload);
    }
});
```

With this code, we've robustly extended RESTSerializer to be able to add root objects to responses when they're missing.
