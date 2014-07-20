##Development guided by tests##
Up until now we have been very much focused on setting up our build pipeline and writing a high level feature tests. And while I promised that it was time to write some code, we do have do one more setup step before we can get stuck in. To get confidence in our code we will be writing JavaScript modules using tests and we want those tests to run all the time (i.e. with each save). To that end we need to set up some more tasks to run those tests for us and add them to our our deployment process. 

###Setting up our unit test runner using karma###
I have chosen [Karma](https://www.npmjs.org/package/karma) as our Unit test runner, if you are new to [Karma](http://karma-runner.github.io/) I suggest you take a peak at some of the videos on the site. It comes with a variety of plugins and supports basically all of the popular unit test frameworks. As our testing framework we will use Jasmine.

Before going to far, let's quickly create a few folders in the root of our project. `src/js` is where we will store all of our JavaScript source code, later on we will create a task to concatenate/minify and move it to our app folder:

	-> tests
		-> unit
	-> src
		->js
		
_TODO: this for now but really I want to do commonJS_		

As with all tasks, let's create a new branch:

	git checkout -b test-runner
	
And then let's install the package and add it to our package.json file:

	npm install karma --save-dev
	
Ok time to create our Karma configuration file, typically you would type in the root of your project:

	karma init karma.conf.js

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
	> src/js/**/*.js
	WARN [init]: There is no file matching this pattern.

	> tests/unit/**/*.js
	WARN [init]: There is no file matching this pattern.

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
                'src/js/**/*.js',
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
            autoWatch: true,
    
    
            // start these browsers
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['PhantomJS'],
    
    
            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: true
        });
    };

Let's take it for a spin:

	karma start
	INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	INFO [launcher]: Starting browser PhantomJS
	WARN [watcher]: Pattern "/Users/writer/Projects/github/weatherly/tests/unit/**/*.js" does not match any file.
	INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket iqriF61DkEH0qp-sXlwR with id 10962078
	PhantomJS 1.9.7 (Mac OS X): Executed 0 of 0 ERROR (0.003 secs / 0 secs)

So we got an error, but that is because we have no tests. Let's wrap this into a grunt task:

	npm install grunt-karma --save-dev

And update our Gruntfile to load the task:

	module.exports = function (grunt) {
        grunt.initConfig({
            express: {
                test: {
                    options: {
                        script: './server.js'
                    }
                }
            },
            cucumberjs: {
                src: 'tests/e2e/features/',
                options: {
                    steps: 'tests/e2e/steps/'
                }
            },
            karma: {
                unit: {
                    configFile: 'karma.conf.js'
                }
            }
        });

        grunt.loadNpmTasks('grunt-express-server');
        grunt.loadNpmTasks('grunt-selenium-webdriver');
        grunt.loadNpmTasks('grunt-cucumber');
        grunt.loadNpmTasks('grunt-karma');

        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    };

Let's try this out our new grunt task:
    
	grunt karma:unit
	
	Running "karma:unit" (karma) task
	INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	INFO [launcher]: Starting browser PhantomJS
	WARN [watcher]: Pattern "/Users/gregstewart/Projects/github/weatherly/src/js/**/*.js" does not match any file.
	WARN [watcher]: Pattern "/Users/gregstewart/Projects/github/weatherly/tests/unit/**/*.js" does not match any file.
	INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket QO4qLCSO-4DZVO7eaRky with id 9893379
	PhantomJS 1.9.7 (Mac OS X): Executed 0 of 0 ERROR (0.003 secs / 0 secs)
	Warning: Task "karma:unit" failed. Use --force to continue.

	Aborted due to warnings.

Similar output, with the difference that our process terminated this time because of the warnings about no files macthing our pattern. We'll fix this issue by writing our very first unit test!

###Writing our first unit test### 
Let's go ahead and create one source file and one test file, as well as some of the folder structure for our project: 

	mkdir src/js/model
	touch src/js/model/TodaysWeather.js
	mkdir tests/unit/
	mkdir tests/unit/model/
	touch tests/unit/model/TodaysWeather-spec.js



 * code coverage
 * continuously run unit tests with grunt
 * add them to our deployment process
 * writing our first unit test
 * integration test
 * using stubs
 * using mocks