---
category: posts
layout: single
mainTopic: 'ember'
title: "Metadata via HTTP headers in Ember.js"  
---

In your ambitious web app, there's a very high probability you'll want to pass some metadata from your store along with the actual data, e.g. total count of all records when using pagination, various descriptions and whatnot.

By default, Ember Data's JSON deserializer looks for a meta key in received response. But wait! What if your API delivers juicy metadata via HTTP response headers? We need to extend the adapter for specific record type, override handleResponse, intercept stuff we need from headers and manually declare them as metadata with the meta key. Simple stuff. Say we're loading some bonsais:

```javascript
// app/adapters/bonsai.js
export default DS.RESTAdapter.extend({
 handleResponse: function(status, headers, payload) {
   let meta = {
     total: headers["X-Total-Count"],
     pages: headers["X-Page-Count"]
   };
   payload.meta = meta;
   return this._super(status, headers, payload);
  }
});
```

Adapter then automatically picks up things we've set as meta. This allows us to access the metadata like so:


```
{% raw %}{{bonsais.meta.total}} {% endraw %} // in templates
this.get('bonsais.meta.total') // everywhere else
```

Happy computering!
