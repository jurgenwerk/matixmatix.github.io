---
category: posts
layout: single
mainTopic: 'ember'
title: "How to reset Ember applications"  
---

Usually when the user logs out of your Ember application, we want to clear the state, wipe out every trace of user activity, and unload the store in order to reach a blank state and make the application ready to start a fresh session.

Most of the time we are (totally rightfully) tempted to find the one and only **Ember way** for each possible action in our application. So, you go surfin' in the Ember's [API docs](http://emberjs.com/api/classes/Ember.Application.html) and find the `Application#reset` method. It says it re-creates the application's container. Sounds good?

**No. Do not use it!** (except perhaps in tests).

Why is that so?

The `reset` method *can not* ensure all possible states will be wiped out. Due to the nature of JavaScript, there's a plethora of ways us programmers can pile up various state and values all over the place in a way where Ember's container mechanism is not aware of it.

Here's one example. Ever found guilty of doing this?

```javascript
export default Ember.Controller.extend({
  name: '', // don't do this
  actions: {
    saveName(name) {
      this.set('name', name);
    }
  }
});
```

This piece of code puts the `name` in the class prototype and `Application#reset` will not be able to reset it. BTW, for obvious reasons, **never** declare default object values in a way depicted in the upper example, but rather set them in the object's `init()` hook.

What should we use then?

### The good ol' page refresh.

To be 100% sure all memory is wiped away and the grounds are freshly prepared, just reload the page (usually in the session invalidation step). `window.location = '/'` or something similar should do just fine. There's no shame in doing it. Things are cached. Don't worry.

P.S. Are you fresh in Ember land and want to learn more tricks like these? Check out [Front-end revolution with Ember.js](http://emberjs-book.com/)!
