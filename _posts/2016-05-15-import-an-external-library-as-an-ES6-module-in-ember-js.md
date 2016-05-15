---
category: posts
layout: single
title: "Import an external library as an ES6 module in your Ember app"  
---

Consuming 3rd party libraries from the browser's global `window` scope is nowadays considered as outrageous, egregious, preposterous.

Why is that so? It goes against all good practices of code reuse. Even your JavaScript linter starts complaining. That’s when we’re 98% positive we’re doing something wrong.

The 1001th rule of building ambitious web applications states as follows:

> “As your application grows, you’re gonna need to bring in that obscure JavaScript library from Bower that nobody cared to module-ize or addon-ize before”.

Sure, we can put the external library's code into our vendor folder (or read it from `bower_components`) and include it in our Ember build like so:

```javascript
//ember-cli-build.js
var EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  (...),
  app.import('vendor/chartist/chartist.js'); // or bower_components

  return app.toTree();
};
```

...but then we’re still left with the stinkin’ `Chartist` global (in this case). BTW, [Chartist.js](https://gionkunz.github.io/chartist-js/) is a library for drawing cute charts.

Let’s instead make a new Ember add-on, called `ember-chartist-shim`, that will enable any Ember app to consume the Chartist library as an ES6 module.

What’s a shim? It’s usually a wrapper that brings a new API to an older environment. In this case, the newer stuff is an ES6 module.

What that means is that we’ll be able to install the shim with `ember addon ember-chartist-shim` terminal command and easily import the library wherever in our application (and not be forced to use globals), like so:

```javascript
import chartist from ‘chartist'
```


Let's get to work.

Create your add-on and step into it:

```
ember addon ember-chartist-shim
cd ember-chartist-shim
```

Create a blueprint that fetches the library from Bower after the add-on gets installed:

```
ember g blueprint ember-chartist-shim
```

```javascript
// blueprints/ember-chartist-shim/index.js

/*jshint node:true*/
module.exports = {
  normalizeEntityName: function() {
    // allows to run ember -g ember-chartist-shim and not blow up
    // because ember cli normally expects the format
    // ember generate <entityName> <blueprint>
  },
  afterInstall: function(options) {
    return this.addBowerPackageToProject('chartist');
  }
};
```

Tell the add-on what to include in your target Ember application:

```javascript
// index.js

/* jshint node: true */
'use strict';

module.exports = {
  name: 'chartist',
  included: function included(app) {
    this._super.included(app);
    app.import(app.bowerDirectory + '/chartist/dist/chartist.css');
    app.import(app.bowerDirectory + '/chartist/dist/chartist.js');
    app.import('vendor/chartist.js', {
      exports: {
        Chartist: ['default']
      }
    });
  }
};
```

... and generate the module (AMD module syntax):

```javascript
// vendor/chartist.js

(function() {
  /* globals define, chartist */

  function generateModule(name, values) {
    define(name, [], function() {
      'use strict';

      return values;
    });
  }

  generateModule('chartist', { 'default': Chartist });
})();
```

That's it. You can now test in the add-on's dummy app if the module import actually works. The absolute minimum you could do is to make a little unit test:

```javascript
// tests/unit/chartist-module-test.js
import { module, test } from 'qunit';
import chartist from 'chartist';

module('chartist as an ES6 module');

test('it works', function(assert) {
  assert.ok(chartist);
});
```

You're now ready to publish your add-on and use it in any Ember application. For more test examples please visit the [ember-chartist-shim](https://github.com/matixmatix/ember-chartist-shim) GitHub repo.

If you want to read more on this topic, read this [shimming best practices](http://discuss.emberjs.com/t/best-practices-shimming-libraries-which-use-global-variables/7922) post.



P.S. Are you fresh in Ember land? I wrote a great book for you: [Front-end revolution with Ember.js](http://emberjs-book.com/). Check it out!
