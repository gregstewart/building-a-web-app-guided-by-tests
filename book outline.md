#Book outline#

## Introduction ##
The idea for this book started with a series of blog posts I wrote some time ago exploring building a [Backbone.js](http://backbonejs.org/) app using tests. I initially wanted to consolidate these blog posts into a book, but upon reflection decided to expand the scope.

### What we will be building ###
As you go through this book, we'll be builing a small JavaScript weather app using [forecast.io](http://forecast.io) API. The app itself will use the browser's Geolocation API to figure out where your user is and retrieve a weekly weather forecast for that location.

### How we will build it ###
Plain and simple, we'll build this app guided by tests using a continuous delivery model, by having it deployed using [Codeship's CD service](https://www.codeship.io/) to a [Heroku](https://www.heroku.com/) instance. 

The focus of this book is really about setting you on the right path to delivering quality software reliably and continuously. By the end you should have the confidence to push every commit to 'production'.

We will be making extensive use of the following tools:
* Node.js
* Grunt
* Yo
* Karma
* Jasmine
* Cucumber
* Selenium
 
  
### What you will need ###
There are few pre-requisits you will need to get this app built.
#### Services you wil need to sign up for ####
As I mentioned for our weather forecast API, we'll be using [forecast.io](https://developer.forecast.io/), so you might want to [go and sign up for a developer account](https://developer.forecast.io/register) as you will need a key to access the API.

So that we can reliably deploy our app, we'll make use of [Codeship's](https://www.codeship.io/) hosted CD service. Sign up for the [free service] (https://www.codeship.io/pricing) to get started.

To host our app we'll make use of [Heroku](https://www.heroku.com/) cloud computing service. They also [offer a free service](https://www.heroku.com/pricing) to help you get started.

You should also sign up for a [Github](https://github.com/) or [Bitbucket](https://bitbucket.org/) account if you don't already have one, we'll need this for our CD service.

That should cover the things you need to sign up for.

####Code editor####
You will need a decent IDE (I recommend [WebStorm](http://www.jetbrains.com/webstorm/)) or Text Editor ([Sublime](http://www.sublimetext.com/) is very popular with colleagues)

#### Version control: Git ####
I recommend using Version Control for every project, regardless of size or complexity. If don't already have [Git](http://git-scm.com/) installed you should do so. There are many ways to install the necessary binaries and the [Git website has all the necessary links](http://git-scm.com/downloads). If you are on a Mac though, then I would recommend using [Homebrew](http://brew.sh/) to install the binaries.

If you are new to Git then I recommend taking the [Git Immersion guided tour](http://gitimmersion.com/).

#### Node.js and NPM ####
We'll be making extensive use of JavaScript throughout the book, so you will need to install [Node.js and NPM](http://nodejs.org/). Once again if you are on a Mac though, then I would recommend using [Homebrew](http://brew.sh/) to install the binaries.

NPM will allow us to resolve all of the dependencies we need in order to achieve our goal of building and delivering a web app guided by tests. It's time to get started!

## Getting started ##
The first thing I like to do with any project is to get our build pipeline set up and start deploying code to our 'production' environment. To that end we need to look at building the simplest thing possible to validate our testing infrastructure works, our CI envinronment can pick up changes on commit and after a succesful build deploy the changes.

I will assume you have signed up and installed all of the software outlined in the [What you will need section]("What you will need section").

### Setting up our project ###
Open up a terminal window or code to the command line and navigate to the location you want to store your project files. Once there let's create a project folder:

	mkdir weatherly && cd weatherly

And initilise our github repository:

	git init
	Initialized empty Git repository in /Users/gregstewart/Projects/github/weatherly/.git/

With that all done let's go ahead and install [Yeoman](http://yeoman.io/) as it gives us a quick way to get started with our new app. Type the following: `npm install -g yo`

Next we'll install the webapp generator for Yeoman: `npm install -g generator-webapp`. Once everything is installed let's scaffold our app: `yo webapp` ([see appendices for the full output](Yeoman output) - please note that I opted to inlcude SASS, Modernizr and Bootstrap). 

To verify that everything has installed properly go ahead and type `grunt`, if everything is happy you should see a bunch of output that ends with something like this:

	Execution Time (2014-06-14 13:15:27 UTC)
	jshint:all          98ms  ▇▇ 1%
	concurrent:test     1.3s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 14%
	mocha:all           1.5s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 16%
	concurrent:dist     2.7s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 28%
	autoprefixer:dist  300ms  ▇▇▇▇ 3%
	cssmin:generated   108ms  ▇▇ 1%
	uglify:generated      2s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 21%
	modernizr:dist      1.2s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 13%
	Total 9.4s

Typing grunt `grunt serve` should start our server at [http://localhost:9000](http://localhost:9000) and open up the browser.

![App running using the local server](Screenshot 2014-06-14 14.18.44.png)

We are nearly done, let's commit this to our local repository. Typing `git status` should show you all changes currently pending:

	# On branch master
	#
	# Initial commit
	#
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	.editorconfig
	#	.gitattributes
	#	.gitignore
	#	.jshintrc
	#	Gruntfile.js
	#	app/
	#	bower.json
	#	package.json
	#	test/
	nothing added to commit but untracked files present (use "git add" to track)

Let's stage all of the untracked files: `git add .` and now let's commit these changes `git commit -m "Initial commit, Yeoman base set up"`


### writing our first functional test ###
Thanks to Yeoman we don't need to write our very own functional test this is backed in by the initial installation. If you type `grunt test`, you should see the following output:

	Running "test" task

	Running "clean:server" (clean) task
	Cleaning .tmp...OK

	Running "concurrent:test" (concurrent) task
    
    	Running "copy:styles" (copy) task
    
    
    	Done, without errors.
    
    
    	Execution Time (2014-06-14 13:25:27 UTC)
    	loading tasks  4ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 33%
    	copy:styles    7ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 58%
    	Total 12ms
    
	Running "autoprefixer:dist" (autoprefixer) task

	Running "connect:test" (connect) task
	Started connect web server on http://localhost:9001

	Running "mocha:all" (mocha) task
	Testing: http://localhost:9001/index.html

	  ․
	
	  1 passing (2ms)

	>> 1 passed! (0.00s)

	Done, without errors.


	Execution Time (2014-06-14 13:25:26 UTC)
	concurrent:test  1.3s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 46%
	mocha:all        1.5s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 52%
	Total 2.9s

Let's review the tests. 

If you are opening the project for the first time in your IDE as I did with WebStorm, you may notice that it added a `.idea` folder. We do not want to check that folder into our repository, so open up the .gitignore file in the root of our project and .idea to it:

	node_modules
	dist
	.idea
	.tmp
	.sass-cache
	bower_components
	test/bower_components

Let's commit this change quickly: `git commit -am "Adding .idea folder to the ignore list"`

###setting up our ci environment using codeship###
 * deploy to heroku
* Tdd
 * setting up our unit tests with karma
   * code coverage
 * continuously run unit tests with grunt
 * add them to our deployment process
 * writing our first unit test
 * integration test
 * using stubs
 * using mocks

* other
 * isomorphic apps
 * react for template rendering
 * bake in performance testing



## Appendices ##
### Yeoman output ###

	[?] ==========================================================================
	We're constantly looking for ways to make yo better! 
	May we anonymously report usage statistics to improve the tool over time? 
	More info: https://github.com/yeoman/insight & http://yeoman.io
	==========================================================================: Yes

     _-----_
    |       |
    |--(o)--|   .--------------------------.
    `---------´  |    Welcome to Yeoman,    |
    ( _´U`_ )   |   ladies and gentlemen!  |
    /___A___\   '__________________________'
     |  ~  |
    __'.___.'__
    ´   `  |° ´ Y `

	Out of the box I include HTML5 Boilerplate, jQuery, and a Gruntfile.js to build your app.
	[?] What more would you like? Bootstrap, Sass, Modernizr
	[?] Would you like to use libsass? Read up more at 
	https://github.com/andrew/node-sass#reporting-sass-compilation-and-syntax-issues: Yes
	   	create Gruntfile.js
	   	create package.json
	   	create .gitignore
	   	create .gitattributes
	   	create bower.json
	   	create .jshintrc
	   	create .editorconfig
	   	create app/favicon.ico
	   	create app/404.html
	   	create app/robots.txt
	   	create app/.htaccess
	   	create app/styles/main.scss
	   	create app/index.html
	   	create app/scripts/main.js


	I'm all done. Running bower install & npm install for you to install the required dependencies. If this fails, try running the command yourself.


	npm WARN package.json github@0.0.0 No description
	npm WARN package.json github@0.0.0 No repository field.
	npm WARN package.json github@0.0.0 No README data
	[?] May bower anonymously report usage statistics to improve the tool over time? Yes
	bower jquery#~1.11.0        not-cached git://github.com/jquery/jquery.git#~1.11.0
	bower jquery#~1.11.0           resolve git://github.com/jquery/jquery.git#~1.11.0
	bower modernizr#~2.6.2      not-cached git://github.com/Modernizr/Modernizr.git#~2.6.2
	bower modernizr#~2.6.2         resolve git://github.com/Modernizr/Modernizr.git#~2.6.2
	bower bootstrap-sass-official#~3.1.0       not-cached git://github.com/twbs/bootstrap-sass.git#~3.1.0
	bower bootstrap-sass-official#~3.1.0          resolve git://github.com/twbs/bootstrap-sass.git#~3.1.0
	bower modernizr#~2.6.2                       download https://github.com/Modernizr/Modernizr/archive/v2.6.3.tar.gz
	bower bootstrap-sass-official#~3.1.0         download https://github.com/twbs/bootstrap-sass/archive/v3.1.1+2.tar.gz
	bower jquery#~1.11.0                         download https://github.com/jquery/jquery/archive/1.11.1.tar.gz
	bower bootstrap-sass-official#~3.1.0          extract archive.tar.gz
	bower modernizr#~2.6.2                        extract archive.tar.gz
	bower bootstrap-sass-official#~3.1.0         resolved git://github.com/twbs/bootstrap-sass.git#3.1.1+2
	bower jquery#~1.11.0                          extract archive.tar.gz
	bower modernizr#~2.6.2                       resolved git://github.com/Modernizr/Modernizr.git#2.6.3
	bower jquery#~1.11.0                         resolved git://github.com/jquery/jquery.git#1.11.1
	bower bootstrap-sass-official#~3.1.0          install bootstrap-sass-official#3.1.1+2
	bower modernizr#~2.6.2                        install modernizr#2.6.3
	bower jquery#~1.11.0                          install jquery#1.11.1
	\
	bootstrap-sass-official#3.1.1+2 bower_components/bootstrap-sass-official

	modernizr#2.6.3 bower_components/modernizr

	jquery#1.11.1 bower_components/jquery
	\
	> phantomjs@1.9.7-8 install /Users/gregstewart/Projects/github/node_modules/grunt-mocha/node_modules/grunt-lib-phantomjs/node_modules/phantomjs
	> node install.js

	PhantomJS detected, but wrong version 1.9.0 @ /usr/local/bin/phantomjs.
	Downloading https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-macosx.zip
	Saving to /var/folders/s5/3rn1xjt95bb2l30kgmwl21t80000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip
	Receiving...
	Received 9186K total.
	Extracting zip contents
	Copying extracted folder /var/folders/s5/3rn1xjt95bb2l30kgmwl21t80000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip-extract-1402749857579/phantomjs-1.9.7-macosx -> /Users/gregstewart/Projects/github/node_modules/grunt-mocha/node_modules/grunt-lib-phantomjs/node_modules/phantomjs/lib/phantom
	Writing location.js file
	Done. Phantomjs binary available at /Users/gregstewart/Projects/github/node_modules/grunt-mocha/node_modules/grunt-lib-phantomjs/node_modules/phantomjs/lib/phantom/bin/phantomjs
 
	> optipng-bin@0.3.8 postinstall /Users/gregstewart/Projects/github/node_modules/grunt-contrib-imagemin/node_modules/image-min/node_modules/optipng-bin
	> node index.js

	✓ pre-build test passed successfully

	> pngquant-bin@0.1.7 postinstall /Users/gregstewart/Projects/github/node_modules/grunt-contrib-imagemin/node_modules/image-min/node_modules/pngquant-bin
	> node index.js

	✓ pre-build test passed successfully

	> jpegtran-bin@0.2.6 postinstall /Users/gregstewart/Projects/github/node_modules/grunt-contrib-imagemin/node_modules/image-min/node_modules/jpegtran-bin
	> node index.js


	> gifsicle@0.1.5 postinstall /Users/gregstewart/Projects/github/node_modules/grunt-contrib-imagemin/node_modules/image-min/node_modules/gifsicle
	> node index.js

	✓ pre-build test passed successfully

	> node-sass@0.8.6 install /Users/gregstewart/Projects/github/node_modules/grunt-sass/node_modules/node-sass
	> node build.js

	`darwin-x64-v8-3.14` exists; testing

	  ․․․․․․․․․․․․․․․․․․․․․․

	  22 passing (57ms)

	Binary is fine; exiting
	grunt-contrib-copy@0.5.0 node_modules/grunt-contrib-copy

	grunt-rev@0.1.0 node_modules/grunt-rev

	grunt-contrib-concat@0.3.0 node_modules/grunt-contrib-concat

	grunt-contrib-clean@0.5.0 node_modules/grunt-contrib-clean
	└── rimraf@2.2.8

	time-grunt@0.3.2 node_modules/time-grunt
	├── date-time@0.1.1
	├── pretty-ms@0.1.0
	├── text-table@0.2.0
	├── hooker@0.2.3
	└── chalk@0.4.0 (has-color@0.1.7, ansi-styles@1.0.0, strip-ansi@0.1.1)

	jshint-stylish@0.1.5 node_modules/jshint-stylish
	├── text-table@0.2.0
	└── chalk@0.4.0 (has-color@0.1.7, ansi-styles@1.0.0, strip-ansi@0.1.1)

	grunt-concurrent@0.5.0 node_modules/grunt-concurrent
	├── async@0.2.10
	└── pad-stdio@0.1.1 (lpad@0.2.1)

	grunt-contrib-htmlmin@0.2.0 node_modules/grunt-contrib-htmlmin
	├── each-async@0.1.3
	├── pretty-bytes@0.1.1
	├── html-minifier@0.5.6
	└── chalk@0.4.0 (has-color@0.1.7, strip-ansi@0.1.1, ansi-styles@1.0.0)

	grunt-newer@0.7.0 node_modules/grunt-newer
	├── rimraf@2.2.6
	└── async@0.2.10

	grunt-contrib-jshint@0.9.2 node_modules/grunt-contrib-jshint
	├── hooker@0.2.3
	└── jshint@2.4.4 (console-browserify@0.1.6, minimatch@0.3.0, underscore@1.4.4, exit@0.1.2, shelljs@0.1.4, cli@0.4.5, htmlparser2@3.3.0)

	grunt-contrib-uglify@0.4.0 node_modules/grunt-contrib-uglify
	├── chalk@0.4.0 (ansi-styles@1.0.0, has-color@0.1.7, strip-ansi@0.1.1)
	├── uglify-js@2.4.14 (uglify-to-browserify@1.0.2, async@0.2.10, optimist@0.3.7, source-map@0.1.34)
	└── maxmin@0.1.0 (pretty-bytes@0.1.1, gzip-size@0.1.1)

	grunt-contrib-cssmin@0.9.0 node_modules/grunt-contrib-cssmin
	├── clean-css@2.1.8 (commander@2.1.0)
	├── chalk@0.4.0 (ansi-styles@1.0.0, has-color@0.1.7, strip-ansi@0.1.1)
	└── maxmin@0.1.0 (pretty-bytes@0.1.1, gzip-size@0.1.1)

	grunt-usemin@2.1.1 node_modules/grunt-usemin
	├── debug@0.7.4
	└── lodash@1.0.1

	grunt-autoprefixer@0.7.5 node_modules/grunt-autoprefixer
	├── diff@1.0.8
	├── chalk@0.4.0 (ansi-styles@1.0.0, has-color@0.1.7, strip-ansi@0.1.1)
	└── autoprefixer@1.2.0 (fs-extra@0.9.1, postcss@0.3.5, caniuse-db@1.0.20140611)

	grunt-bower-install@1.4.1 node_modules/grunt-bower-install
	├── wiredep@1.4.4 (chalk@0.1.1, through2@0.4.2, glob@3.2.11, lodash@1.3.1)
	└── bower-config@0.5.1 (osenv@0.0.3, graceful-fs@2.0.3, optimist@0.6.1, mout@0.9.1)

	grunt-modernizr@0.5.2 node_modules/grunt-modernizr
	├── colors@0.6.2
	├── promised-io@0.3.4
	├── uglify-js@1.3.3
	└── request@2.27.0 (json-stringify-safe@5.0.0, aws-sign@0.3.0, forever-agent@0.5.2, qs@0.6.6, tunnel-agent@0.3.0, oauth-sign@0.3.0, cookie-jar@0.3.0, node-uuid@1.4.1, mime@1.2.11, form-data@0.1.3, hawk@1.0.0, http-signature@0.10.0)

	grunt-contrib-watch@0.6.1 node_modules/grunt-contrib-watch
	├── async@0.2.10
	├── tiny-lr-fork@0.0.5 (debug@0.7.4, faye-websocket@0.4.4, noptify@0.0.3, qs@0.5.6)
	├── lodash@2.4.1
	└── gaze@0.5.1 (globule@0.1.0)

	grunt-svgmin@0.4.0 node_modules/grunt-svgmin
	├── each-async@0.1.3
	├── pretty-bytes@0.1.1
	├── chalk@0.4.0 (has-color@0.1.7, ansi-styles@1.0.0, strip-ansi@0.1.1)
	└── svgo@0.4.4 (colors@0.6.2, whet.extend@0.9.9, coa@0.4.1, sax@0.6.0, js-yaml@2.1.3)

	load-grunt-tasks@0.4.0 node_modules/load-grunt-tasks
	├── findup-sync@0.1.3 (glob@3.2.11, lodash@2.4.1)
	└── multimatch@0.1.0 (minimatch@0.2.14, lodash@2.4.1)

	grunt@0.4.5 node_modules/grunt
	├── which@1.0.5
	├── dateformat@1.0.2-1.2.3
	├── eventemitter2@0.4.13
	├── getobject@0.1.0
	├── rimraf@2.2.8
	├── colors@0.6.2
	├── hooker@0.2.3
	├── async@0.1.22
	├── grunt-legacy-util@0.2.0
	├── exit@0.1.2
	├── nopt@1.0.10 (abbrev@1.0.5)
	├── glob@3.1.21 (inherits@1.0.0, graceful-fs@1.2.3)
	├── minimatch@0.2.14 (sigmund@1.0.0, lru-cache@2.5.0)
	├── coffee-script@1.3.3
	├── underscore.string@2.2.1
	├── iconv-lite@0.2.11
	├── lodash@0.9.2
	├── grunt-legacy-log@0.1.1 (underscore.string@2.3.3, lodash@2.4.1)
	├── js-yaml@2.0.5 (esprima@1.0.4, argparse@0.1.15)
	└── findup-sync@0.1.3 (glob@3.2.11, lodash@2.4.1)

	grunt-contrib-connect@0.7.1 node_modules/grunt-contrib-connect
	├── connect-livereload@0.3.2
	├── open@0.0.4
	├── portscanner@0.2.2 (async@0.1.15)
	├── async@0.2.10
	└── connect@2.13.1 (uid2@0.0.3, methods@0.1.0, debug@0.8.1, qs@0.6.6, pause@0.0.1, cookie-signature@1.0.1, fresh@0.2.0, bytes@0.2.1, raw-body@1.1.3, buffer-crc32@0.2.1, batch@0.5.0, cookie@0.1.0, compressible@1.0.0, negotiator@0.3.0, send@0.1.4, multiparty@2.2.0)

	grunt-mocha@0.4.11 node_modules/grunt-mocha
	├── lodash@2.3.0
	├── mocha@1.18.2 (diff@1.0.7, growl@1.7.0, commander@2.0.0, mkdirp@0.3.5, debug@1.0.2, glob@3.2.3, jade@0.26.3)
	└── grunt-lib-phantomjs@0.4.0 (eventemitter2@0.4.13, semver@1.0.14, temporary@0.0.8, phantomjs@1.9.7-8)

	grunt-contrib-imagemin@0.6.1 node_modules/grunt-contrib-imagemin
	├── pretty-bytes@0.1.1
	├── mkdirp@0.3.5
	├── chalk@0.4.0 (ansi-styles@1.0.0, has-color@0.1.7, strip-ansi@0.1.1)
	├── async@0.2.10
	└── image-min@0.2.3 (win-spawn@2.0.0, filesize@2.0.3, stream-combiner@0.0.4, multipipe@0.0.2, through2@0.4.2, concat-stream@1.4.6, mout@0.9.1, map-key@0.1.4, optipng-bin@0.3.8, pngquant-bin@0.1.7, jpegtran-bin@0.2.6, gifsicle@0.1.5)

	grunt-sass@0.11.0 node_modules/grunt-sass
	├── each-async@0.1.3
	├── chalk@0.4.0 (strip-ansi@0.1.1, ansi-styles@1.0.0, has-color@0.1.7)
	└── node-sass@0.8.6 (node-watch@0.3.4, mkdirp@0.3.5, nan@0.8.0, optimist@0.6.1, sinon@1.10.2, mocha@1.18.2)
	   	invoke   mocha:app
		create test/bower.json
	   	create test/.bowerrc
		create test/spec/test.js
	    create test/index.html


	I'm all done. Running bower install for you to install the required dependencies. If this fails, try running the command yourself.


	bower chai#~1.8.0           not-cached git://github.com/chaijs/chai.git#~1.8.0
	bower chai#~1.8.0              resolve git://github.com/chaijs/chai.git#~1.8.0
	bower mocha#~1.14.0         not-cached git://github.com/visionmedia/mocha.git#~1.14.0
	bower mocha#~1.14.0            resolve git://github.com/visionmedia/mocha.git#~1.14.0
	bower chai#~1.8.0             download https://github.com/chaijs/chai/archive/1.8.1.tar.gz
	bower mocha#~1.14.0           download https://github.com/visionmedia/mocha/archive/1.14.0.tar.gz
	bower chai#~1.8.0              extract archive.tar.gz
	bower mocha#~1.14.0            extract archive.tar.gz
	bower chai#~1.8.0             resolved git://github.com/chaijs/chai.git#1.8.1
	bower mocha#~1.14.0           mismatch Version declared in the json (1.12.0) is different than the resolved one (1.14.0)
	bower mocha#~1.14.0           resolved git://github.com/visionmedia/	mocha.git#1.14.0
	bower chai#~1.8.0              install chai#1.8.1
	bower mocha#~1.14.0            install mocha#1.14.0

	chai#1.8.1 bower_components/chai

	mocha#1.14.0 bower_components/mocha
