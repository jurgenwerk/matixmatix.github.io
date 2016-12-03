---
category: posts
layout: single
mainTopic: 'ember'
title: "Continuous integration for Ember.js apps with Codeship and Divshot"  
---

[DEPRECATED] I guess pretty much everyone agrees there is no better thing in life than to see your app get deployed automatically after you push some code to your repository.

Since Ember build is a fully static web assets bundle, it's possible to serve it via a CDN so you can take some load off your app servers. While the pros and cons of serving your JavaScript app either via CDN or your server are heavily debatable, let's spare this essay for another time.

For my continuous delivery system I chose [Codeship](http://codeship.com), and [Divshot](http://divshot.com) for CDN. Once you set up your account on both platforms (repository push hook on Codeship, Divshot token), all it's left to do is to write a simple custom deploy script on Codeship - this case demonstrates deploy to staging environment:

```bash
npm install -g bower
npm install -g divshot-cli
npm i && bower i
ember build --environment=staging
ember test //if you have tests
divshot push staging --token $DIVSHOT_TOKEN
```

Same script could most probably also be used in other popular continuous integration platforms, such as CircleCI and others.
