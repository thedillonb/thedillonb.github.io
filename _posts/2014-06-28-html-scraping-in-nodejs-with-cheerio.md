---
layout: post
published: true
title: HTML Scraping in NodeJS with Cheerio
categories: Programming
tags: 
  - nodejs
  - html scraping
---

As a developer I live for APIs. The ability to take structured information from one source and transform it into another is very exciting. Unfortunately, not all information sources provide a API. Many times, the information needed is jamed into a HTML source which presents a challenge attempting to extract it. Luckily, there's an incredible NodeJS package called [Cheerio](https://github.com/cheeriojs/cheerio) which makes this task pretty simple.

I'm going to demonstrate creating a NodeJS application which will HTML scrape information from GitHub's [Showcase](https://github.com/showcases) Page. This example is taken from the [GitHub-Trending](https://github.com/thedillonb/CodeHub-Trending) project where GitHub's trending and showcase pages are scraped to provide a JSON API which can be viewed [here](http://codehub-trending.herokuapp.com/trending) - this project is used to power a few of my mobile applications.

Here's a few prerequisites:

1. NodeJS - duh
2. The [Request](https://github.com/mikeal/request) package
3. The [Cheerio](https://github.com/cheeriojs/cheerio) package

For this demonstration I'm going to create a single file application which will dump information when executed. This will suffice for an example but if you're looking to integrate with a larger project take the steps proper steps to modulize it.

Run the following commands in the command line

1. `mkdir cheerio-example` - Create a new project directory
2. `cd cheerio-example` - Go into the project directory you just created
3. `npm install request` - Install the Request package
4. `npm install cheerio` - Install the cheerio package
5. `touch app.js` - Create the application file

Great! We've got our dependencies downloaded and the application file created. Now it's time to start populating the _app.js_ file with content:

{% highlight javascript %}
var cheerio = require('cheerio');
var request = require('request');

request({
    method: 'GET',
    url: 'https://github.com/showcases'
}, function(err, response, body) {
    if (err) return console.error(err);
    console.log(body);
});
{% endhighlight %}

If we run this (`node app.js`) it will reach out to GitHub's Showcase page, grab the HTML and print it to the console. Not very exciting but we're actually half way. This is where Cheerio comes in. If you've never used Cheerio before then you're in for a treat. Cheerio takes raw HTML, parses it, and returns a jQuery object to you so you may traverse the DOM.

{% highlight javascript %}
var cheerio = require('cheerio');
var request = require('request');

request({
    method: 'GET',
    url: 'https://github.com/showcases'
}, function(err, response, body) {
    if (err) return console.error(err);

    // Tell Cherrio to load the HTML
    $ = cheerio.load(body);
    $('li.collection-card').each(function() {
            var href = $('a.collection-card-image', this).attr('href');
            if (href.lastIndexOf('/') > 0) {
                console.log($('h3', this).text());
            }
    });
});
{% endhighlight %}

If we run this we'll see each showcase printed to the console like so:

    Icon fonts
    Package managers
    Science
    Machine learning
    Web games
    Emoji
    Projects with great wikis
    Productivity tools
    Policies
    CSS preprocessors
    Video tools
    Clean code linters
    Data visualization
    Projects that power GitHub
    Projects that power GitHub for Windows

That's all there is to it. Cheerio, combined with Request, makes parsing HTML very easy. With just this example, you can begin scraping HTML into structred data which can be used in practical applications - in my case, mobile applications! The iOS application, [CodeHub](http://codehub-app.com/), calls out to [CodeHub-Trending](https://github.com/thedillonb/CodeHub-Trending) which exposes a structured API of data that is scraped from GitHub's Trending and Showcase web pages!