---
title: Automated accessibility testing with Travis CI
description: Using Pa11y CI to make accessibility tests a key part of your Node.js application's development
date: 2017-03-08 16:45
layout: post
theme: soft
type: Article
topic: Accessibility
crossPost:
  site: Cruft.io
  url: http://cruft.io/posts/automated-accessibility-testing-node-travis-ci-pa11y/
---

<div class="gutters" markdown="1">
Too often accessibility testing is left as a manual process, and so inevitably it is done only sporadically (or worse; never).  To help address this shortfall, [Springer Nature developed and released Pa11y](http://cruft.io/posts/accessibility-testing-with-pa11y/) which uses HTML CodeSniffer and PhantomJS to raise common accessibility issues in an automated manner.

Since then Pa11y has continued to evolve to suit the needs of the developer community.  This week with the first stable release of the brand new [Pa11y CI], it's easier than ever to leverage Pa11y and make accessibility a core part of your development and deployment processes.

---
**Please note:**  
Pa11y is not designed to be a complete replacement for regular accessibility testing. It's not a magic bullet, and won't catch everything that you could. Instead think of it as being complementary to your existing workflow (which should include testing with actual users).

For the purposes of this article I'll assume you're running a Node.js application and are already [up and running with Travis CI](https://docs.travis-ci.com/user/getting-started/).

---

## Getting started with Pa11y CI

First, we want to make sure that accessibility tests are a first-class citizen just like your unit and integration tests, so let's add [Pa11y CI] to your dev dependencies list in your `package.json`;

{% highlight diff %}
{
  "name": "my-fabulous-app",
  "version": "1.0.0",
  "dependencies": {},
  "devDependencies": {
    "jshint": "^2",
+   "pa11y-ci": "~1.0",
    "your-other-dependencies": "~3.5"
  },
  "scripts": {
    "start": "node app",
    "test": "make test",
+   "test-pa11y": "./node_modules/.bin/pa11y-ci"
  }
}
{% endhighlight %}

We've also added a new npm script `test-pa11y` which will allow you to run the tests locally, as well as during Travis's build process.


### Basic Pa11y configuration

At its simplest you can run Pa11y CI with no options.  In this case, Pa11y CI will look for a `.pa11yci` file in the root directory of your application, where [you can provide it with a list of urls](https://github.com/pa11y/ci#configuration) to check for accessibility problems.

It's also possible to [specify a sitemap to run tests against](https://github.com/pa11y/ci#sitemaps).

To get up and running, let's assume a very simple `.pa11yci` configuration like the following;

{% highlight json %}
{
  "defaults": {
    "timeout": 1000
  },
  "urls": [
    "http://localhost:5400/home",
    "http://localhost:5400/archive"
  ]
}
{% endhighlight %}

### Testing the code that's changing, not what is on live

Since we ideally want to be catching errors **before** they go live, it's important to be able to run Pa11y against the code that you've actually changed.  To be able to do this, Travis must be able to spin up a version of your application to test against.

I'm going to assume you can do so using `npm start`, but if your start script is named something else you'll have to edit that below.

In your `.travis.yml` config we need to create a new build script to run the Pa11y tests:

{% highlight yaml %}
# Build script
script: 
  - npm run start & sleep 5; npm run test-pa11y;
{% endhighlight %}

There's some complexity on that line, so let's break that down.  The build script will now;

1. `npm run start` - start your application
1. `&` - direct that process to [run in the background](http://bashitout.com/2013/05/18/Ampersands-on-the-command-line.html)
1. `sleep 5;` - wait a little while for it to finish starting
1. `npm run test-pa11y` - and finally run the Pa11y test script we defined earlier

### Testing multiple things in parallel

Rather than running your accessibility tests multiple times in your test matrix, I find it useful to split them out and run them in a parallel build process (in the same way that I treat my linting test).

Below you can see an environment variable `MODE` being set in a much abridged `.travis.yml` file:

{% highlight yaml %}
# Build matrix
language: node_js
matrix:
  include:

    # Lint only in Node.js 0.12
    - node_js: '0.12'
      env: MODE=lint

    # Pa11y check only in Node.js 4.x
    - node_js: '4'
      env: MODE=pa11y
      (etc...)

    # Run tests in Node.js 4.x
    - node_js: '4'
      (etc...)

    # Run tests in Node.js 6.x
    - node_js: '6'
      (etc...)

# Build script
script:
  - 'if [ $MODE == "lint" ]; then make lint; fi'
  - 'if [ $MODE == "pa11y" ]; then make start & sleep 5; make test-pa11y; fi'
  - 'if [ ! $MODE ]; then make test; fi'
{% endhighlight %}

You can then see how I use this `$MODE` variable to either run just the linting command, just the pa11y tests, or the other tests instead - all in parallel!

![Screenshot of Travis showing 4 parallel build jobs][screenshot-builds]
</div>

[Pa11y CI]: https://github.com/pa11y/ci
[screenshot-builds]: /resources/images/automated-accessibility-testing-node-travis-ci-pa11y/builds.png
