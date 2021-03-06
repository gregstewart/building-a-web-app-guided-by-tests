#Unit tests#
Up until now we have been very much focused on setting up our build pipeline and writing a high level feature tests. And while I promised that it was time to write some code, we do have do a few more setup steps to carry out before we can get stuck in. To get confidence in our code we will be writing JavaScript modules using tests and we want those tests to run all the time (i.e. with each save). To that end we need to set up some more tasks to run those tests for us and add them to our build process.

##Setting up our unit test runner using karma##
I have chosen [Karma](https://www.npmjs.org/package/karma) as our Unit test runner, if you are new to [Karma](http://karma-runner.github.io/) I suggest you take a peak at some of the videos on the site. It comes with a variety of plugins and supports basically all of the popular unit test frameworks. As our testing framework we will use Jasmine.

Before going to far, let's quickly create a few folders in the root of our project. You should already have a `node_modules/weatherly/src/js`, which is where we will store all of our JavaScript source code. We already have a task to concatenate/minify and move it to our app folder. Now we just need to create a `unit` folder for our unit tests. So the structure of our code and tests will look as follows:

	-> tests
		-> unit
	-> node_modules
		-> weatherly
			->js
		
As with all tasks, let's create a new branch:

	> git checkout -b test-runner
	
And then let's install the package and add it to our package.json file:

	> npm install karma --save-dev
	
Ok time to create our Karma configuration file, typically you would type in the root of your project:

	> karma init karma.conf.js

This would guide you through the process of setting up your test runner, here's how I answered the setup questions:

	Which testing framework do you want to use ?
	Press tab to list possible options. Enter to move to the next question.
	> jasmine

	Do you want to use Require.js ?
	This will add Require.js plugin.
	Press tab to list possible options. Enter to move to the next question.
	> no

	Do you want to capture any browsers automatically ?
	Press tab to list possible options. Enter empty string to move to the next question.
	> PhantomJS
	> 

	What is the location of your source and test files ?
	You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
	Enter empty string to move to the next question.
	> node_modules/weatherly/js/**/*.js,
	> tests/unit/**/*.js
	> 

	Should any of the files included by the previous patterns be excluded ?
	You can use glob patterns, eg. "**/*.swp".
	Enter empty string to move to the next question.
	> 

	Do you want Karma to watch all the files and run the tests on change ?
	Press tab to list possible options.
	> no

	Config file generated at "/Users/writer/Projects/github/weatherly/karma.conf.js".

And here's the corresponding configuration that was generated:

    // Karma configuration
    // Generated on Sun Jul 20 2014 16:18:54 GMT+0100 (BST)
    
    module.exports = function (config) {
        config.set({
    
            // base path that will be used to resolve all patterns (eg. files, exclude)
            basePath: '',
    
    
            // frameworks to use
            // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
            frameworks: ['jasmine'],
    
    
            // list of files / patterns to load in the browser
            files: [
                'node_modules/weatherly/js/**/*.js',
                'tests/unit/**/*.js'
            ],
    
    
            // list of files to exclude
            exclude: [
            ],
    
    
            // preprocess matching files before serving them to the browser
            // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
            preprocessors: {
            },
    
    
            // test results reporter to use
            // possible values: 'dots', 'progress'
            // available reporters: https://npmjs.org/browse/keyword/karma-reporter
            reporters: ['progress'],
    
    
            // web server port
            port: 9876,
    
    
            // enable / disable colors in the output (reporters and logs)
            colors: true,
    
    
            // level of logging
            // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
            logLevel: config.LOG_INFO,
    
    
            // enable / disable watching file and executing tests whenever any file changes
            autoWatch: false,
    
    
            // start these browsers
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['PhantomJS'],
    
    
            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: false
        });
    };

Let's take it for a spin:

	> karma start
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> WARN [watcher]: Pattern "/Users/writer/Projects/github/weatherly/tests/unit/**/*.js" does not match any file.
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket iqriF61DkEH0qp-sXlwR with id 10962078
	> PhantomJS 1.9.7 (Mac OS X): Executed 0 of 0 ERROR (0.003 secs / 0 secs)

So we got an error, but that is because we have no tests. Let's wrap this into a grunt task:

	> npm install grunt-karma --save-dev

Create a `build\test.js file`:

    (function (module) {
        'use strict';
        var config = {
            karma: {
            	unit: {
	            	configFile: 'karma.conf.js'
            	}
        	}
        };
        
        module.exports = function (grunt) {
            grunt.loadNpmTasks('grunt-karma');
            
            grunt.config('karma', config);
        };
    })(module);

Let's try this out our new grunt task:
    
	> grunt karma:unit
	
	> Running "karma:unit" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> WARN [watcher]: Pattern "/Users/gregstewart/Projects/github/weatherly/src/js/**/*.js" does not match any file.
	> WARN [watcher]: Pattern "/Users/gregstewart/Projects/github/weatherly/tests/unit/**/*.js" does not match any file.
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket QO4qLCSO-4DZVO7eaRky with id 9893379
	> PhantomJS 1.9.7 (Mac OS X): Executed 0 of 0 ERROR (0.003 secs / 0 secs)
	> Warning: Task "karma:unit" failed. Use --force to continue.

	> Aborted due to warnings.

Similar output, with the difference that our process terminated this time because of the warnings about no files macthing our pattern. We'll fix this issue by writing our very first unit test!

##Writing and running our first unit test 
In the [previous chapter](link here) we created a source folder and added a sample module, to confirm our build process for our JavaScript assets worked. Let's go ahead and create one test file, as well as some of the folder structure for our project: 

	> mkdir tests/unit/
	> mkdir tests/unit/model/
	> touch tests/unit/model/TodaysWeather-spec.js

What we want to do know is validate our Karma configuration before we starting our real tests, so let's add a sample test to our `TodaysWeather-spec.js`:

	'use strict';
	/* exported TodaysWeather */
	var TodaysWeather = require('weatherly/js/model/TodaysWeather');

	describe('Today \'s weather', function () {
    	it('should return 2', function () {
       		expect(1+1).toBe(2);
	    });
	});

We could try and run our Karma task again, but this would only result in an error, because we are using the [CommonJS](http://wiki.commonjs.org/wiki/CommonJS) module approach and we would see an error stating that `module` is not defined, because our module under tests uses:

	module.exports = TodaysWeather;

We need to somehow tell our test runner that we use the CommonJS module type and resolve `module` and `require`. Once again we will resort to a npm module: `karma-commonjs`:

	> npm install karma-commonjs --save-dev

Next we need to update our `karma.conf.js` file:
	
	// Karma configuration
    // Generated on Sun Jul 20 2014 16:18:54 GMT+0100 (BST)

    module.exports = function (config) {
        config.set({

            // base path that will be used to resolve all patterns (eg. files, exclude)
            basePath: '',


            // frameworks to use
            // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
            frameworks: ['jasmine', 'commonjs'],


            // list of files / patterns to load in the browser
            files: [
                'node_modules/weatherly/js/**/*.js',
                'tests/unit/**/*.js'
            ],


            // list of files to exclude
            exclude: [
            ],


            // preprocess matching files before serving them to the browser
            // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
            preprocessors: {
                'node_modules/weatherly/js/**/*.js': ['commonjs'],
                'tests/unit/**/*.js': ['commonjs']
            },


            // test results reporter to use
            // possible values: 'dots', 'progress'
            // available reporters: https://npmjs.org/browse/keyword/karma-reporter
            reporters: ['progress'],


            // web server port
            port: 9876,


            // enable / disable colors in the output (reporters and logs)
            colors: true,


            // level of logging
            // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
            logLevel: config.LOG_INFO,


            // enable / disable watching file and executing tests whenever any file changes
            autoWatch: false,


            // start these browsers
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['PhantomJS'],


            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: true
        });
    };

We added `commonjs` to the frameworks block and then configured the `preprocessor` block to process our source and test files using our chosen module type. Let's try this again:

	> grunt karma:unit
	> Running "karma:unit" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 3i335x4S_5GG88E7TsOM with id 58512369
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.005 secs / 0.002 secs)
	
	> Done, without errors.

Perfect!

## Running our tests as part of the build
Now that we have our test runner set up' let's add it to our build process. This is going to require us to register a new task as we will need to do a few things:

* build our assets
* run our unit tests
* run our end to end tests

Let's go ahead and create a task called `test` in our `Gruntfile` and configure it to execute these tasks:

    module.exports = function (grunt) {
		'use strict';
        
        grunt.registerTask('generate', ['less:production', 'copy:fonts', 'browserify:code']);
        grunt.registerTask('build', ['bower:install', 'generate']);
        
        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    
        grunt.registerTask('test', ['build', 'karma:unit', 'e2e']);
    
        grunt.registerTask('heroku:production', 'build');
    };
    
And let's make sure everything runs as intended:

	> grunt test
	> Running "less:production" (less) task
	> File app/css/main.css created: 131.45 kB → 108.43 kB

	> Running "copy:fonts" (copy) task
	> Copied 4 files

	> Running "browserify:dist" (browserify) task

	> Running "karma:unit" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 1kikUD-UC4_Gd6Qh9T49 with id 53180162
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.002 secs / 0.002 secs)

	> Running "selenium_start" task
	> seleniumrc webdriver ready on 127.0.0.1:4444

	> Running "express:test" (express) task
	> Starting background Express server
	> Listening on port 3000

	> Running "cucumberjs:src" (cucumberjs) task
	> ...

	> 1 scenario (1 passed)
	> 3 steps (3 passed)

	> Running "selenium_stop" task

	> Running "express:test:stop" (express) task
	> Stopping Express server

	> Done, without errors.

If you recall we configured our build to execute `grunt e2e`, we need to update this now to execute `grunt test`. Log into to your Codeship dashboard and edit the test configuration:

![Codeship dashboard with updating test configuration](images/Screenshot 2014-09-07 12.59.58.png)

Ready to give this is a spin? 

 	> git status
 	> git add .
 	> git commit -m "Karma test configuration added and new build test task created"
 	> git checkout master
 	> git merge test-runner
 	> git push
 
If we keep an eye on our dashboard we should see a build kicked-off and `test` task being executed:

![Codeship dashboard with updating test configuration](images/Screenshot 2014-09-07 13.08.51.png)
 

## Continuously running our tests
So far so good. While it's it's nice to have our tests execute manually, let's automate this. Could it be as simple as setting the `karma.conf.js` setting `singleRun` to `false` and `autoWatch` to `true`, well kind of. When running things on our build server we still want the single run to be true, however for local development purposes it should always run. So let's tackle this:

	> git checkout -b run-tests-continuously

Let'start by modifying our `karma.conf.js` file to run tests continuously:

	// Karma configuration
    // Generated on Sun Jul 20 2014 16:18:54 GMT+0100 (BST)

    module.exports = function (config) {
        config.set({

            // base path that will be used to resolve all patterns (eg. files, exclude)
            basePath: '',


            // frameworks to use
            // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
            frameworks: ['jasmine', 'commonjs'],


            // list of files / patterns to load in the browser
            files: [
                'node_modules/weatherly/js/**/*.js',
                'tests/unit/**/*.js'
            ],


            // list of files to exclude
            exclude: [
            ],


            // preprocess matching files before serving them to the browser
            // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
            preprocessors: {
                'node_modules/weatherly/js/**/*.js': ['commonjs'],
                'tests/unit/**/*.js': ['commonjs']
            },


            // test results reporter to use
            // possible values: 'dots', 'progress'
            // available reporters: https://npmjs.org/browse/keyword/karma-reporter
            reporters: ['progress'],


            // web server port
            port: 9876,


            // enable / disable colors in the output (reporters and logs)
            colors: true,


            // level of logging
            // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
            logLevel: config.LOG_INFO,


            // enable / disable watching file and executing tests whenever any file changes
            autoWatch: true,


            // start these browsers
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['PhantomJS'],


            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: false
        });
    };

Try this out by running `grunt karma:unit` and after the first run has completed making a change to our source `TodaysWeather` file:

	> grunt karma:unit
	> Running "karma:unit" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 9k2m2-j9Ets37p7sWD2Y with id 21051890
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.007 secs / 0.003 secs)
	> INFO [watcher]: Changed file "/Users/gregstewart/Projects/github/weatherly/node_modules/weatherly/js/model/TodaysWeather.js".
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.005 secs / 0.002 secs)

Exactly what we wanted. Let's now tackle our build versus development problem. We will just create a seperate build task in our `test.js` file that uses the same configuration file but overrides the properties we need for our build environment:

    (function (module) {
        'use strict';
        var config = {
            dev: {
                configFile: 'karma.conf.js'
            },
            ci: {
                configFile: 'karma.conf.js',
                singleRun: true,
                autoWatch: false
            }
        };
    
        module.exports = function (grunt) {
            grunt.loadNpmTasks('grunt-karma');
    
            grunt.config('karma', config);
        };
    })(module);

    
I took the chance to rename the `karma:unit` task to `karma:dev` and create a `karma:ci` task. The next thing was update the `test` task in our `Gruntfile.js` to also execute `karma:ci` instead of `karma:unit`. 

    module.exports = function (grunt) {
        'use strict';
    
        grunt.loadTasks('build');
    
        grunt.registerTask('generate', ['less:production', 'copy:fonts', 'browserify:code']);
        grunt.registerTask('build', ['bower:install', 'generate']);
    
        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    
        grunt.registerTask('test', ['build', 'karma:ci', 'e2e']);
        grunt.registerTask('heroku:production', 'build');
    };

Let's test all those changes:

	> grunt test       
	> Running "bower:install" (bower) task
	> >> Installed bower packages
	> >> Copied packages to /Users/gregstewart/Projects/github/weatherly/bower_components
	
	> Running "less:production" (less) task
	> File app/css/main.css created: 131.45 kB → 108.43 kB
	
	> Running "copy:fonts" (copy) task
	> Copied 4 files
	
	> Running "browserify:code" (browserify) task
	
	> Running "karma:ci" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 67MYRP0HFPzhHE9wY3vt with id 58848767
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.002 secs / 0.002 secs)
	
	> Running "selenium_start" task
	> seleniumrc webdriver ready on 127.0.0.1:4444
	
	> Running "express:test" (express) task
	> Starting background Express server
	> Listening on port 3000
	
	> Running "cucumberjs:src" (cucumberjs) task
	> ...
	
	> 1 scenario (1 passed)
	> 3 steps (3 passed)
	
	> Running "selenium_stop" task
	
	> Running "express:test:stop" (express) task
	> Stopping Express server
	
	> Done, without errors.

Time to commit our changes back to master and push to origin and see the whole thing go through our build pipeline:

	> git add .
	> git commit -m "Run tests continuously in dev and single run on CI"
	> git checkout master
	> git merge run-tests-continuously
	> git push
	
Now check your CI server and you should see the following if it all went to plan:

![Codeship dashboard with updated ci test configuration](images/Screenshot 2014-09-11 21.40.18.png)
	
## Adding code coverage
A little bonus while we are setting up test infrastructure, code coverage. Now code coverage is one of these topics that sparks almost fanatical discussions, however I find a useful tool to make sure I have covered all of the important parts of my code. Lucky for us, it's easy to add support to Karma all we need is a plugin:

	> git checkout -b code-coverage
	> npm install karma-coverage --save-dev
	
And modify our `karma.conf.js` file: 

    // Karma configuration
    // Generated on Sun Jul 20 2014 16:18:54 GMT+0100 (BST)
    
    module.exports = function (config) {
        config.set({
    
            // base path that will be used to resolve all patterns (eg. files, exclude)
            basePath: '',
    
    
            // frameworks to use
            // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
            frameworks: ['jasmine', 'commonjs'],
    
    
            // list of files / patterns to load in the browser
            files: [
                'node_modules/weatherly/js/**/*.js',
                'tests/unit/**/*.js'
            ],
    
    
            // list of files to exclude
            exclude: [
            ],
    
    
            // preprocess matching files before serving them to the browser
            // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
            preprocessors: {
                'node_modules/weatherly/js/**/*.js': ['commonjs', 'coverage'],
                'tests/unit/**/*.js': ['commonjs'],
            },
    
    
            // test results reporter to use
            // possible values: 'dots', 'progress'
            // available reporters: https://npmjs.org/browse/keyword/karma-reporter
            reporters: ['progress', 'coverage'],
    
            coverageReporter: {
                reporters: [
                    { type: 'html'},
                    { type: 'text-summary' }
                ],
                dir: 'reports/coverage'
            },
    
            // web server port
            port: 9876,
    
    
            // enable / disable colors in the output (reporters and logs)
            colors: true,
    
    
            // level of logging
            // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
            logLevel: config.LOG_INFO,
    
    
            // enable / disable watching file and executing tests whenever any file changes
            autoWatch: true,
    
    
            // start these browsers
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['PhantomJS'],
    
    
            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: false
        });
    };

Running our `grunt karma:dev` task yields the following output:

	> Running "karma:dev" (karma) task
	> INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	> INFO [launcher]: Starting browser PhantomJS
	> INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket OVbmSp9OmFMK23l5jA2J with id 80988698
	> PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.005 secs / 0.002 secs)
	
	> =============================== Coverage summary ===============================
	> Statements   : 83.33% ( 5/6 )
	> Branches     : 100% ( 2/2 )
	> Functions    : 50% ( 1/2 )
	> Lines        : 66.67% ( 2/3 )
	> ================================================================================
	> ^C
	> Done, without errors.
  
There's more information in the shape of an HTML report to be found under `reports/coverage/`. Here's a what that looks like

![Istanbule code coverage report](images/Screenshot 2014-09-11 22.14.57.png)
 
One final thing before we close off this section. You may not want to run your coverage report as part the CI process. If you do then ignore this part. By editing our `test.js` and for our `karma:ci` task overriding the reporter step with `reporters: ['progress']` we can skip this step for our build.

    (function (module) {
        'use strict';
        var config = {
            dev: {
                configFile: 'karma.conf.js'
            },
            ci: {
                configFile: 'karma.conf.js',
                singleRun: true,
                autoWatch: false,
                reporters: ['progress']
            }
        };

        module.exports = function (grunt) {
            grunt.loadNpmTasks('grunt-karma');

            grunt.config('karma', config);
        };
    })(module);
      
We also do not want to commit our reports to our repos, so another `.gitignore` tweak is needed:

	.idea
	bower_components
	reports
    phantomjsdriver.log
	app/css
	app/fonts
	app/js
	node_modules/*
	!node_modules/weatherly
	
Time to commit and merge our changes:
	
	> git add .	
	> git commit -m "added coverage to dev process"
	> git checkout master
	> git merge code-coverage 
	> git push
	
##Linting as part of the build
Since we are about to write some code we should circle back to adding the lint task to ci process. This is really straight forward now:

	 git checkout -b linting-the-build
	 
 Open up `Gruntfile.js`	 and add the `jshint` task to our `test` task.
 
     module.exports = function (grunt) {
        'use strict';
    
        grunt.loadTasks('build');
    
        grunt.registerTask('generate', ['less:production', 'copy:fonts', 'browserify:code']);
        grunt.registerTask('build', ['bower:install', 'generate']);
    
        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    
        grunt.registerTask('test', ['jshint', 'build', 'karma:ci', 'e2e']);
        grunt.registerTask('heroku:production', 'build');
    };	
    
To verify it works locally just type `> grunt test`

	Running "jshint:source" (jshint) task
	>> 5 files lint free.

	Running "bower:install" (bower) task
	>> Installed bower packages
	>> Copied packages to /Users/gregstewart/Projects/github/weatherly/bower_components

	Running "less:production" (less) task
	File app/css/main.css created: 131.45 kB → 108.43 kB

	Running "copy:fonts" (copy) task
	Copied 4 files

	Running "browserify:code" (browserify) task

	Running "karma:ci" (karma) task
	INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	INFO [launcher]: Starting browser PhantomJS
	INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket lR_IEn9s3FvaphN90JfV with id 32288954
	PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.002 secs / 0.002 secs)

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
	
Looking good let's commit the change and watch it go through the build:

	git add .
	git commit -m "Added lint to ci/test task"
	git checkout master
	git pull origin master
	git merge linting-the-build
	git push origin  master
	
## Recap
We covered quite a bit of ground here:

* we set up Karma
* wrote a very basic unit test to validate the test runner was working
* configured the tests for local continuous testing and single run during the build
* added code coverage to our tooling
* added linting to our ci run as well

With that onwards to development guided by tests!