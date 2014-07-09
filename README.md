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

## Getting started ##
The first thing I like to do with any project is to get our build pipeline set up and start deploying code to our 'production' environment. To that end we need to look at building the simplest thing possible to validate our testing infrastructure works, our CI envinronment can pick up changes on commit and after a succesful build deploy the changes.

I will assume you have signed up and installed all of the software outlined in the [What you will need section]("What you will need section").

### Setting up our project ###
Open up a terminal window or code to the command line and navigate to the location you want to store your project files. Once there let's create a project folder:

	mkdir weatherly && cd weatherly

And initilise our github repository:

	git init
	Initialized empty Git repository in /Users/gregstewart/Projects/github/weatherly/.git/

Before we go any further let's create a .gitignore file in the root of our project and add the following to it:

	node_modules
	.idea

Let's commit this change quickly: 
	
	git add .gitignore
	git commit -m "Adding folders to the ignore list"

From a folder perspective I like to create a distribution folder and an app folder to hold the source, so let's go ahead and add these folders as well.
	
	mkdir app
	mkdir dist

We'll start by using bower to grab some of our front end dependencies. Let's start by creating a bower.json file by typing: `bower init`, fill in the details as you see fit, but here's what I selected:

	{
  		name: 'weatherly',
  		version: '0.0.0',
  		authors: [
    		'Greg Stewart <gregs@tcias.co.uk>'
  		],
  		description: 'Building a web app guided by tests',
  		moduleType: [
    		'amd'
  		],
  		license: 'MIT',
  		homepage: 'http://www.tcias.co.uk/',
  		private: true,
  		ignore: [
    		'**/.*',
    		'node_modules',
    		'bower_components',
    		'test',
    		'tests'
  		]
	}	

Everybody likes a bit of Bootstrap so let's start with that package (I use the SASS port and quite frankly so should you):

	bower install bootstrap-sass-official --save

The `--save` flag at the end of the command means that the dependecy will be added to our bower.jon file. We are doing this because we do not want to check in any external dependencies into our repository. Instead at build/CI time we'll restore these using bower.

So let's edit our .gitignore file to make sure we don't accidentally commit these files:

	node_modules
	.idea
	bower_components

And let's add this change to our repo:

	git add .gitignore 
	git commit -m "Adding bower_components to the ignore list"

Modernizr is another staple:

	bower install modernizr --save

To round things off let's install HTML5 boilerplate

	bower install html5-boilerplate
	
You may have noticed that I decided not to add this package to our bower.json file, simply because we'll copy the files we need into our app folder:

	mv bower_components/html5-boilerplate/css app/
	mv bower_components/html5-boilerplate/img app/
	mv bower_components/html5-boilerplate/*.html app/
	mv bower_components/html5-boilerplate/*.png app/ 
	mv bower_components/html5-boilerplate/*.xml app/
	mv bower_components/html5-boilerplate/*.ico app/
	mv bower_components/html5-boilerplate/*.txt app/

In your favourite editor open up the app/index.html file and add the following:

	<!DOCTYPE html>
	<!--[if lt IE 7]>      <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
	<!--[if IE 7]>         <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
	<!--[if IE 8]>         <html class="no-js lt-ie9"> <![endif]-->
	<!--[if gt IE 8]><!--> <html class="no-js"> <!--<![endif]-->
    	<head>
        	<meta charset="utf-8">
        	<meta http-equiv="X-UA-Compatible" content="IE=edge">
        	<title>Weatherly - Forecast for London</title>
        	<meta name="description" content="">
    	    <meta name="viewport" content="width=device-width, initial-scale=1">

	        <!-- Place favicon.ico and apple-touch-icon.png in the root directory -->

        	<link rel="stylesheet" href="css/main.css">
    	</head>
    	<body>
        	<!--[if lt IE 7]>
            	<p class="browsehappy">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
        	<![endif]-->

        	<!-- Add your site or application content here -->
        	<div class="container">
            	<div class="header">
                	<ul class="nav nav-pills pull-right">
                    	<li class="active"><a href="#">Home</a></li>
                    	<li><a href="#">About</a></li>
                   		<li><a href="#">Contact</a></li>
                	</ul>
                	<h3 class="text-muted">test</h3>
            	</div>

            	<div class="jumbotron">
                	<h1>London Right Now</h1>
                	<p class="temperature">14 degrees</p>
                	<p>Mostly cloudy - feels like 14 degrees</p>
            	</div>

            	<div class="row marketing">
                	<div class="col-lg-6">
                    	<h4>NEXT HOUR</h4>
                    	<p>Mostly cloudy for the hour.</p>

                    	<h4>NEXT 24 HOURS</h4>
                   		<p>Mostly cloudy until tomorrow afternoon.</p>
                	</div>
            	</div>

            	<div class="footer">
                	<p><span class="glyphicon glyphicon-heart"></span> from Weatherly</p>
            	</div>

        	</div>
        	<p></p>

        	<script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        	<script>window.jQuery || document.write('<script src="js/vendor/jquery-1.10.2.min.js"><\/script>')</script>

        	<!-- Google Analytics: change UA-XXXXX-X to be your site's ID. -->
        	<script>
            	(function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
                   	 function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
                	e=o.createElement(i);r=o.getElementsByTagName(i)[0];
                	e.src='//www.google-analytics.com/analytics.js';
                	r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
            	ga('create','UA-XXXXX-X');ga('send','pageview');
        	</script>
    	</body>
	</html>

If you open up the file in your browser you should see something like this hopefully:

![Rendered HTML](manuscript\Screenshot 2014-06-18 23.19.42.png)

Not exactly something to write home about, but it's enough for us to get started setting up our little server, writing a functional test and deploying something to our Heroku instance.

We'll make this a lot prettier later on in the book when we deal with setting up Grunt to build our JavaScript and CSS assets.

The last thing we'll do is commit all of our changes to our local repos:

	git add .
	git commit -m "Added Bootstrap/Modernizr to bower.json, moved the skeleton of the HTML5 boilerplate to the app folder and created a base index page for our weather forecat app."

At this stage it's a good idea to also push the changes to our remote repository. If you have followed the [What you will need section]("What you will need section"), you will hopefully have created a Github account. If not go ahead and to that now. Then create a repository called weatherly, here's what I entered:

![Creating your weatherly repository](manuscript\Screenshot 2014-06-18 23.35.50.png)

To push our changes to the remote repository, you will need to tell your local repo where it is:

	git remote add origin https://github.com/<account_name>/weatherly.git

Now you can push your changes:

	git push -u origin master
	
###Recap###
Before we move on let's just quickly recap what we have done so far:

* created our app folder structure
* initialised our git repo
* created a Git ignore file
* used bower to manage some of our front end dependencies:
	* Bootstrap
	* Modernizr
	* HTML5 boilerplate
* created a very basic index.html page
* pushed all of the changes to our remote git repository

## Writing our first functional test ##
I order to write our first functional test we needed a test page, which we built in the previous section. Now let's set up a simple Node.js webserver.

### Web server: Express ###
A good practice to follow while working with Git is to create a branch for each feature that you are working on, so let's go ahead and create a new branch for this item of work.
	
	git checkout -b web-server
	
Make sure you are in the root of our project and not in the `app/` folder

Since we'll be using Node.js we can use NPM to manage the dependencies. These dependencies are stored in a folder called node_modules. Since we don't want to check any node modules/packages into our repository we added that folder to our `.gitignore` file in when we set up the project. If we don't add those packages to our repository you may be wondering how our CI and Heroku instance will now how to run the app. To that end we'll use a handy file called `package.json`. When we run NPM we can not only install dependencies, we can also add them to our `package.json` file and our target envinronments can read this file and install these packages for us.

Typing `npm init` give us a way to create our package.json, here's what I answered when prompted:

	This utility will walk you through creating a package.json file.
	It only covers the most common items, and tries to guess sane defaults.

	See `npm help json` for definitive documentation on these fields
	and exactly what they do.

	Use `npm install <pkg> --save` afterwards to install a package and
	save it as a dependency in the package.json file.

	Press ^C at any time to quit.
	name: (weatherly) 
	version: (0.0.0) 
	description: Building a web app guided by tests
	entry point: (index.js) 
	test command: grunt test
	git repository: (https://github.com/gregstewart/weatherly.git) 
	keywords: 
	author: Greg Stewart
	license: (ISC) MIT
	About to write to /Users/gregstewart/Projects/github/weatherly/package.json:

	{
  		"name": "weatherly",
  		"version": "0.0.0",
  		"description": "Building a web app guided by tests",
  		"main": "index.js",
  		"scripts": {
    		"test": "grunt test"
  		},
  		"repository": {
    		"type": "git",
    		"url": "https://github.com/gregstewart/weatherly.git"
  		},
  		"author": "Greg Stewart",
  		"license": "MIT",
  		"bugs": {
    		"url": "https://github.com/gregstewart/weatherly/issues"
  		},
	  	"homepage": "https://github.com/gregstewart/weatherly"
	}
	
	Is this ok? (yes) yes

As you can see it autocompleted a bunch of information for you, such as the project name, version number and Git details. Let's add that file to our repo before going any further:

	git add package.json
	git commit -m "Created package.json file"

Now let's go ahead and install a web server module. We'll just use [express](http://expressjs.com/).

	npm install express --save
	
By specifying `--save` the dependecy was added to our `package.json` file, if you open it up you should see the following toward the end of the file:

	"dependencies": {
   		"express": "^4.4.5"
  	}

Next create a new file called `server.js` in the root of our project and add the following content:

	var express = require('express');
	var app = express();

	app.use(express.static(__dirname + '/app'));

	var server = app.listen(3000, function() {
	  console.log('Listening on port %d', server.address().port);
	});

And to start our server type:

	npm start

If you now open your browser and hit `http://localhost:3000` you should once again see: 

![Rendered HTML hosted by our Connect server](Screenshot 2014-06-18 23.19.42.png)

The process that runs our server is not daemonised and will continue to run until we close the console or type `^C`. Go ahead and kill the server and add those changes to our repository, merge these changes back into master and finally push to origin:

	git add server.js
	git add package.json
	git commit -m "Installed Connect and created a very basic web server for our app"
	git checkout master
	git merge web-server
	git push
	
### Cucumber, WebDriver and Selenium ###
For our functional tests I have chosen [Cucumber.js](https://github.com/cucumber/cucumber-js) and [WebDriver.js](http://webdriver.io/) with [Selenium](http://docs.seleniumhq.org/). I chose this combination because I believe this will give you greater felxibility in the long wrong, especially if you plan on using different languagesin your toolchain. You can find Ruby, Java and .Net versions of Cucumber, WebDriver and Selenium.

Once again we'll create a dedicated branch for this work:

	git checkout -b functional-test

####Selenium###

> Selenium uses Java, so you will need to make sure you have it installed.

We could install the binaries manually, but since I plan using Grunt to automate tasks around starting and stopping the server, we might as well use [grunt-selenium-webdriver] (https://www.npmjs.org/package/grunt-selenium-webdriver) module as this includes everything that we need, including the jar file for the Selenium Server. 

	`npm install grunt-selenium-webdriver --save-dev`
	
We use the `--save-dev` flag to indicate that we want to add this dependency to our package.json file, however only for development purposes. With that done let's create a Grunt task to start the server (you can find more information on Grunt and tasks over at [the official Grunt.js website](http://gruntjs.com/getting-started)). The first thing we'll need is a `Gruntfile.js`, so add one to the root of your project and edit it to contain the following:

	module.exports = function(grunt) {
  		grunt.initConfig({
  		});

  		grunt.loadNpmTasks('grunt-selenium-webdriver');

  		grunt.registerTask('e2e', [
    		'selenium_start',
    		'selenium_stop'
  		]);
	};

Save the changes and at the command line type: `grunt e2e` and you should see something like this: 

	Running "selenium_start" task
	seleniumrc webdriver ready on 127.0.0.1:4444

	Running "selenium_stop" task

	Done, without errors.
	
This told grunt to execite a task called `e2e` and confirms that the selenium server started properly at the following address `127.0.0.1:4444` and then was shutdown again (apparently it is not necessary to shutdown the server with a stop task).

#### Using Grunt to start and stop the server####
Let's also add a step to stop and start our web server when we are running our frunctional tests. To that end we'll install another grunt module:

	npm install grunt-express-server --save-dev

And we'll edit our Grunt file so that it looks for our `server.js` and we can control the starting and stopping of our server:

	module.exports = function(grunt) {
  		grunt.initConfig({
    		express: {
      			test: {
        			options: {
          				script: './server.js'
        			}
      			}
    		}
  		});

  		grunt.loadNpmTasks('grunt-express-server');
  		grunt.loadNpmTasks('grunt-selenium-webdriver');

  		grunt.registerTask('e2e', [
    		'selenium_start',
    		'express:test',
    		'selenium_stop',
    		'express:test:stop'
  		]);
	};
	
If you now run `grunt e2e`, you should see the following output:

	Running "selenium_start" task
	seleniumrc webdriver ready on 127.0.0.1:4444

	Running "express:test" (express) task
	Starting background Express server
	Listening on port 3000

	Running "selenium_stop" task

	Running "express:test:stop" (express) task

####WebDriver####
The next thing we need to do is install [WebDriver.js](http://webdriver.io/) and we are then nearly ready to write our first feature test:

	npm install webdriverjs --save-dev


#### Cucumber ####
The final piece of the puzzle is [Cucumber.js](https://github.com/cucumber/cucumber-js):
	
	npm install cucumber --save-dev

### Our first test ###

Features are written using the [Gherkin syntax](https://github.com/cucumber/cucumber/wiki/Gherkin), and this is what our first feature looks like:

	Feature: Using our awesome weather app
		As a user of weatherly
 		I should be able to see the weather information for my location
	
		Scenario: Viewing the homepage
    		Given I am on the home page
    		When I view the main content area
    		Then I should see the temperature for my location

I like to store these and the associated code in a functional-tests directory. So go ahead and create that folder under the root of our app. Then create a features folder and save the above feature contents to a file called `using-weatherly.feature`.

If we were to run our cucumber tests now using `cucumber.js functional-tests/features/using-weatherly.feature` we would see the following output:

	UUU

	1 scenario (1 undefined)
	3 steps (3 undefined)

	You can implement step definitions for undefined steps with these snippets:

	this.Given(/^I am on the home page$/, function (callback) {
  		// express the regexp above with the code you wish you had
  		callback.pending();
	});

	this.When(/^I view the main content area$/, function (callback) {
  		// express the regexp above with the code you wish you had
  		callback.pending();
	});

	this.Then(/^I should see the temperature for my location$/, function (callback) {
  		// express the regexp above with the code you wish you had
  		callback.pending();
	});

This is extremely useful output. While it's clear that the code to execute the steps in the feature are undefined, the output actually gives snippets to create our step definitions. So let's go ahead and create our step definition. Inside of our functional test folder, create a `steps` folder and add a file called `using-weather-steps.js` with the following content:

	var UsingWeatherlyStepDefinitions = function () {

    	this.Given(/^I am on the home page$/, function (callback) {
      		// express the regexp above with the code you wish you had
      		callback.pending();
    	});

    	this.When(/^I view the main content area$/, function (callback) {
      		// express the regexp above with the code you wish you had
      		callback.pending();
    	});
    
    	this.Then(/^I should see the temperature for my location$/, function (callback) {
      		// express the regexp above with the code you wish you had
      		callback.pending();
    	});       
	};

	module.exports = UsingWeatherlyStepDefinitions;
	
Let's try and execute our feature test again with `cucumber.js functional-tests/features/using-weatherly.feature --require functional-tests/steps/using-weatherly-step-definitions.js` and now we should see:

	P--

	1 scenario (1 pending)
	3 steps (1 pending, 2 skipped)

Time to flesh out the steps to do some work and check for elements on the page while the tests are running. We'll make use of [Chai.js](http://chaijs.com/) as our assertion library, so let's go ahead and install this module:

	npm install chai --save-dev

The first bit of code we'll add to our tests is a [World object](https://github.com/cucumber/cucumber-js#world), which willl initialise our browser (read WebDriver) and add a few helper methods (`visit` and `hasText`). As our browser we are using phantomjs, but if you would like to see the test running simply replace `browserName: 'phantomjs'` with say `browserName: 'firefox'`.

> Note that other browsers such as Chrome and IE require special drivers which you can download from the [Selenium website](http://docs.seleniumhq.org/)

Here's our world object (`world.js`), which we save into a folder called support under `functional-tests`:

	var webdriverjs = require('webdriverjs');
	var expect = require('chai').expect;
	var assert = require('chai').assert;

	var client = webdriverjs.remote({ desiredCapabilities: {browserName: 'phantomjs'}, logLevel: 'silent' });

	client.addCommand('hasText', function (selector, text, callback) {
  		this.getText(selector, function (error, result) {
    		expect(result).to.have.string(text);
    		callback();
  		});
	});

	client.init();


	var World = function World(callback) {
  		this.browser = client;

  		this.visit = function(url, callback) {
    		this.browser.url(url, callback);
  		};

  		callback(); // tell Cucumber we're finished and to use 'this' as the world instance
	};

	exports.World = World;

Please note that if when running the tests you come across the message shown below (logging in verbose mode here), this simply (indeed simply...) means that webdriverjs cannot find phatomjs in your PATH.

	====================================================================================
	Selenium 2.0/webdriver protocol bindings implementation with helper commands in nodejs.
	For a complete list of commands, visit http://webdriver.io/docs.html.

	====================================================================================

	[09:51:45]:  ERROR      Couldn't get a session ID - undefined
	Fatal error: [init()] <=
	An unknown server-side error occurred while processing the command.

Now let's re-visit our `using-weatherly-step-definitions.js` and replace the contents with the following code:

	var UsingWeatherlyStepDefinitions = function () {
    	this.World = require("../support/world.js").World;

	    this.Given(/^I am on the home page$/, function (callback) {
      		this.visit('http://localhost:3000/', callback);
    	});

    	this.When(/^I view the main content area$/, function (callback) {
      		this.browser.hasText('.jumbotron h1', 'London Right Now', callback);
    	});

    	this.Then(/^I should see the temperature for my location$/, function (callback) {
      		this.browser.hasText('p.temperature', '14 degrees', callback);
    	});
	};

	module.exports = UsingWeatherlyStepDefinitions;

The first step opens the site, and then we assert that the header element displays `London Right Now` and that the element with our temperature shows `14 degrees`

If we were to once again try and execute our feature test, we would get an error telling us that it can't connect to the selenium server. So let's wrap all of this into our e2e grunt task. Let's start by adding another module to our setup:
	
	npm install grunt-cucumber --save-dev

And let's edit our `Gruntfile.js` to look like this now:

	module.exports = function(grunt) {
  		grunt.initConfig({
    		express: {
      			test: {
        			options: {
          				script: './server.js'
        			}
      			}
    		},
    		cucumberjs: {
      			src: 'functional-tests/features/',
      			options: {
        			steps: 'functional-tests/steps/'
      			}
    		}
  		});

  		grunt.loadNpmTasks('grunt-express-server');
  		grunt.loadNpmTasks('grunt-selenium-webdriver');
  		grunt.loadNpmTasks('grunt-cucumber');

  		grunt.registerTask('e2e', [
    		'selenium_start',
    		'express:test',
    		'cucumberjs',
    		'selenium_stop',
    		'express:test:stop'
  		]);
	};

Now type `grunt e2e` and you should see the following output:

	Running "selenium_start" task
	seleniumrc webdriver ready on 127.0.0.1:4444

	Running "express:test" (express) task
	Starting background Express server
	Listening on port 3000

	Running "cucumberjs:src" (cucumberjs) task
	...

	1 scenario (1 passed)
	3 steps (3 passed)

	Running "selenium_stop" task

	Running "express:test:stop" (express) task
	Stopping Express server

	Done, without errors.
	
With that done we can commit our changes to our repository:

	git add .
	git commit -m "Scenario: Viewing the homepage, created and implemented"
	git checkout master
	git merge functional-test
	git push

###Recap###
To sum things up in this section we created a set of grunt tasks that:

* start our selenium server
* start our express server that hosts our page
* execute the features and steps we defined with cucumberjs
* output the result to the console
* closes down the services after finishing the tests

We also wrote a feature test that:

* open a browser
* check the contents for a header
* check for an element that holds the current temperature

#Continuous delivery#
In the previous part we wrote our first functional test or feature test and automated the running using a set of Grunt tasks. Now we will put these tasks to good use and have our Continuous Integration server run the test with each commit to our remote repository. There are two parts two Continuous Delivery: Continuous Integration and Continuous Deployment. These two best practices were best defined [in the blog post over at Treehouse](http://blog.teamtreehouse.com/use-continuous-integration-continuous-deployment), do read the article, but here's the tl;rd:

>**Continuous Integration** is the practice of testing each change done to your codebase automatically and as early as possible. But this paves the way for the more important process: Continuous Deployment.

>**Continuous Deployment** follows your tests to push your changes to either a staging or production system. This makes sure a version of your code is always accessible.

As always before starting we'll create a dedicated branch for our work:

	git checkout -b ci

##Setting up our ci environment using Codeship##
In the what you will need section I suggested signing up for a few services, if you haven't by now created an account with either [Github](https://github.com/) and [Codeship](https://www.codeship.io/) now is the time! Also if you haven't already now is the time to connect your Githuib account with Codeship. You can do this by looking under your account settings for connected services:

![Link your Github account to Codeship](Screenshot 2014-07-06 20.14.42.png) 

To get started we need to create a new project:

![Create a new project](Screenshot 2014-07-06 20.17.15.png) 

This starts starts a three step process:

1. Connect to your source code provider
2. Choose your repository 
3. Setup test commands

The first step is easy, choose the Github option, then for step two choose the `weatherly` repository from the list.

>If you hadn't already signed up for Github and hadn't pushed your changes to it, then the repository won't be showing up in the list. Link your local repository and [push all changes up](https://help.github.com/articles/pushing-to-a-remote) before continuing.

Not it's time to set up the third step, set up out test commands. From the drop down labelled `Select your technology to prepopulate basic commands` choose `node.js`. 

Next we need to tackle the section: `Modify your Setup Commands`. The instructions tell us that it can use the Node.js version specified in our `package.json` file, given that we have added this information let's go ahead and do that now. If you are unsure of the version of Node.js simply type

	node --version

In my case the output was **0.10.28**, below is my package.json file, look for the block labelled with **engines**:

	{
  		"name": "weatherly",
  		"version": "0.0.0",
  		"description": "Building a web app guided by tests",
  		"main": "index.js",
  		"engines" : {
    		"node" : "~0.10.28"
  		},
  		"scripts": {
    		"test": "grunt test"
  		},
  		"repository": {
    		"type": "git",
    		"url": "https://github.com/gregstewart/weatherly.git"
  		},
  		"author": "Greg Stewart",
  		"license": "MIT",
  		"bugs": {
    		"url": "https://github.com/gregstewart/weatherly/issues"
  		},
  		"homepage": "https://github.com/gregstewart/weatherly",
  		"dependencies": {
    		"express": "^4.4.5"
  		},
  		"devDependencies": {
    		"chai": "^1.9.1",
    		"cucumber": "^0.4.0",
    		"grunt": "^0.4.5",
    		"grunt-cucumber": "^0.2.3",
    		"grunt-express-server": "^0.4.17",
    		"grunt-selenium-webdriver": "^0.2.420",
    		"webdriverjs": "^1.7.1"
  		}
	}

With that added we can edit the set up commands to look as follows:

	npm install
	npm install grunt-cli

Now let's edit the `Modify your Test Commands` section. In the previous chapter we created a set of tasks to run our tests and wrapped them in a grunt command `grunt e2e`. Let's add this command to our configuration:

	grunt e2e

That's hit the big save button. Right now we are ready to push some changes to our repository. Luckily we have a configuration change ready to push!

	git add package.json
	git commit -m "Added node version to the configuration for CI"
	git checkout master
	git merge ci
	git push
	
And with that go over to your codeship dashboard and if it all went well, then you should see something like this:

![First CI run!](Screenshot 2014-07-06 20.46.36.png) 
	
