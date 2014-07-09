#Building Modern Web Apps Guided By Tests#

# Introduction #
The idea for this book started with a series of blog posts I wrote some time ago exploring building a [Backbone.js](http://backbonejs.org/) app using tests. I initially wanted to consolidate these blog posts into a book, but once I started jotting down what I wanted to put into this series, I decided to expand the scope. 

In the Front end and JavaScript world we have come a long way since the heady days of table based layouts sprinkled with Macromedia roll over scripts to make web pages interactive. The lines between back end and front end application code has almost blurred completely, you could argue it no longer exists when you consider Isomorpic apps. Our tooling has changed substantially as well: at our dispsal we have package managers (for both back and front end), build tools, unit test frameworks and deployment tools.  

## What we will be building ##
Over the coming pages and posts we will explore how to build a modern JavaScript web app guided by tests, using a toolset that will allow you to deploy with each commit to the cloud. We'll be building a small JavaScript weather app using [forecast.io](http://forecast.io)'s API. The app itself will use the browser's Geolocation API to figure out where your user is and retrieve a weekly weather forecast for that location.

## How we will build it ##
Plain and simple, we'll build this app guided by tests and using a continuous delivery model, by having it deployed using [Codeship's CD service](https://www.codeship.io/) to a [Heroku](https://www.heroku.com/) instance. 

The focus of this book is really about setting you on the right path to delivering quality software reliably and continuously. By the end you should have the confidence to push every commit to 'production'.

We will be making extensive use of the following tools:
* Node.js
* Grunt
* Karma
* Jasmine
* Cucumber
* Selenium
   
## What you will need ##
There are few pre-requisits you will need to get this app built. You will need to sign up for some services, grab a code/etxt editor, set up a version control system and finally get your Node.js environment configured. 

### Services you wil need to sign up for ###
As I mentioned for our weather forecast API, we'll be using [forecast.io](https://developer.forecast.io/), so you might want to [go and sign up for a developer account](https://developer.forecast.io/register) as you will need a key to access the API.

You should also sign up for a [Github](https://github.com/) or [Bitbucket](https://bitbucket.org/) account if you don't already have one, we'll need this for our CI service.

So that we can reliably deploy our app, we'll make use of [Codeship's](https://www.codeship.io/) hosted Continuous Integration service. Sign up for the [free service] (https://www.codeship.io/pricing) to get started.

To host our app we'll make use of [Heroku](https://www.heroku.com/) cloud computing service. They also [offer a free service](https://www.heroku.com/pricing) to help you get started.

That should cover the things you need to sign up for.

###Code editor###
You will need a decent IDE (I recommend [WebStorm](http://www.jetbrains.com/webstorm/)) or Text Editor ([Sublime](http://www.sublimetext.com/) is very popular with many of my co-workers)

###Version control: Git###
I recommend using Version Control for every project, regardless of size or complexity. If don't already have [Git](http://git-scm.com/) installed you should do so. There are many ways to install the necessary binaries and the [Git website has all the necessary links](http://git-scm.com/downloads). If you are on a Mac though, then I would recommend using [Homebrew](http://brew.sh/) to install the binaries.

If you are new to Git then I recommend taking the [Git Immersion guided tour](http://gitimmersion.com/).

###Node.js and NPM###
We'll be making extensive use of JavaScript throughout the book, so you will need to install [Node.js and NPM](http://nodejs.org/). Once again if you are on a Mac though, then I would recommend using [Homebrew](http://brew.sh/) to install the binaries.

NPM will allow us to resolve all of the dependencies we need in order to achieve our goal of building and delivering a web app guided by tests. 

###Bower###
[Bower](http://bower.io/) is handy tool to manage your front end library dependencies, so I recommed installing it as well. 

###Grunt###
[Grunt](http://gruntjs.com/) will be our build and automation tool. If you haven't used [Grunt](http://gruntjs.com/)  before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. 

With all that installed and configured, it's time to get started!
