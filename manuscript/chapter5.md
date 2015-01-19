#Refactoring our build file#
Before we move on to generating our assets, we are going to take a small detour and refactor our build file. In the upcoming sections we will be adding more tasks to it and it will be become difficult to get an overview of what is happening in there.

Here's what our `Gruntfile.js` currently looks like:

	module.exports = function(grunt) {
		'use strict';

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

And here's what it will end up looking like once we are done:

	module.exports = function(grunt) {
		'use strict';

	   	grunt.loadTasks('build');		
	   	
		grunt.registerTask('e2e', [
    		'selenium_start',
    		'express:test',
    		'cucumberjs',
    		'selenium_stop',
    		'express:test:stop'
		]);
	};

We kick things off as always with a new branch:

	git checkout -b refactor-gruntfile
	
Create a new folder in the root of our project called `build`, this is where we will put our specfici grunt tasks. Let's start with the `express` configuration. In our build folder create a file called `express.js` and add the following:

    (function(module) {
        'use strict';
        var config = {
            test: {
                options: {
                    script: './server.js'
                }
            }
        };
    
        module.exports = function(grunt) {
            grunt.loadNpmTasks('grunt-express-server');
    
            grunt.config('express', config);
        };
    })(module);
    
We hav basically taken all of the **Express** configuration out of the `Gruntfile.js` and moved the loading of the npm task into this file. In order for `Grunt` to now load this file we can use the `grunt.loadTasks('build');` directive.

	module.exports = function(grunt) {
		'use strict';

		grunt.initConfig({
    		cucumberjs: {
      			src: 'tests/e2e/features/',
      			options: {
        			steps: 'tests/e2e/steps/'
      			}
    		}
		});
		
		grunt.loadTasks('build');
		
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

Running `> grunt e2e` again should confirm that all is still well and our end to end test still work. Next let's move the Cucumber task into `build\cucumber.js`

    (function(module) {
        'use strict';
        var config = {
            src: 'tests/e2e/features/',
            options: {
                steps: 'tests/e2e/steps/'
            }
        };
    
        module.exports = function(grunt) {
            grunt.loadNpmTasks('grunt-selenium-webdriver');
            grunt.loadNpmTasks('grunt-cucumber');
    
            grunt.config('cucumberjs', config);
        };
    })(module);
    
Once we have removed the configuration of the task and the loading of these we are in the state described at the outset of this chapter:

	module.exports = function(grunt) {
		'use strict';

	   	grunt.loadTasks('build');		
	   	
		grunt.registerTask('e2e', [
    		'selenium_start',
    		'express:test',
    		'cucumberjs',
    		'selenium_stop',
    		'express:test:stop'
		]);
	};
	
Let's validate things once more time: `> grunt e2e` and you should see the following output:

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

	> git add .
	> git commit -m "Refactored our Gruntfile"
	> git checkout master
	> git merge refactor-gruntfile
	> git push
	
##Extending our build file by linting our code
To appreciate how convenient this approach is, let's add a new task to [lint our code](http://en.wikipedia.org/wiki/Lint_%28software%29): 
	
	checkout -b jshint-task
	
I like using [JSHint](http://jshint.com/), it's prescriptive enough without being overbearing. You can install it using:

	npm install grunt-contrib-jshint --save-dev
	
You can override the settings by either by specifying a .jshintrc file. This is a good idea particularly if you working as part of a team as you can share the configuration and thus all adhere to the conventions of the project. Let's create a placeholder file:

	touch .jshintrc
	
Here are a few settings that have been useful in previous projects to get the call rolling:

    {
        "strict": true,
        "unused": true,
        "undef": true,
        "camelcase": true,
        "curly": true,
        "eqeqeq": true,
        "forin": true,
        "indent": 4,
        "newcap": true,
        "trailing": true,
        "maxdepth": 2,
        "browser": true,
        "devel": true,
        "node": true,
        "quotmark": true,

        "globals": {
            "sinon": false,
            "define": false,
            "beforeEach": false,
            "afterEach": false,
            "expect": false,
            "describe": false,
            "it": false,
            "xdescribe": false,
            "ddescribe": false,
            "xit": false,
            "iit": false,
            "jasmine": false
        }
    }
    
 Now for defining the task, create a `build/lint.js` file with the following content:
 
     (function (module) {
        var config = {
            options: {
                jshintrc: './.jshintrc'
            },
            source: {
                src: [
                    './Gruntfile.js',
                    './build/**/*.js',
                    './tests/**/*.js',
                    './js/**/*.js',
                ]
            },
        };
        
        module.exports = function (grunt) {
            grunt.loadNpmTasks('grunt-contrib-jshint');
            
            grunt.config('jshint', config);
        }
    })(module);
    
You can now run this task using `> grunt jshint`:

	Running "jshint:source" (jshint) task
	>> 5 files lint free.

	Done, without errors.

You can add this task to a watcher, but I find that it slows the feedback loop, instead I add to a pre-commit hook, so that this only runs before I check my code in. We will also add it to our CI process, just in case someone forgets to add it to their pre-commit hook. To add a pre-commit hookl we need to do is create the following file `.git/hooks/pre-commit` with the following content:

	#!/bin/sh
	#
	# Pre-commit hooks

	# Run lint task before committing
	grunt jshint

_Add exit code._ 

To test this, let's commit and merge:

	git add .
	git commit -m "Added linting"
	
	Running "jshint:source" (jshint) task

	build/lint.js
      2 |    var config = {
             ^ Missing "use strict" statement.
     15 |    }
              ^ Missing semicolon.
     21 |    }
              ^ Missing semicolon.

	>> 3 errors in 5 files
	Warning: Task "jshint:source" failed. Use --force to continue.
	
Oh lucky we added linting! Let's fix the problem:

	(function (module) {
        'use strict';
        var config = {
            options: {
                jshintrc: './.jshintrc'
            },
            source: {
                src: [
                    './Gruntfile.js',
                    './build/**/*.js',
                    './node_modules/weatherly/**/*.js',
                    './tests/**/*.js',
                    './js/**/*.js'
                ]
            }
        };
    
        module.exports = function (grunt) {
            grunt.loadNpmTasks('grunt-contrib-jshint');
    
            grunt.config('jshint', config);
        };
    })(module);

And now let's ammend our commit:

	git add .
	git commit -m "Added linting"
	Running "jshint:source" (jshint) task
	>> 5 files lint free.

	Done, without errors.
	[jshint-task 8126006] Fixing linting issues
	8 files changed, 30 insertions(+), 30 deletions(-)
	rewrite build/lint.js (99%)

Now we can merge our changes in:	
	
	git checkout master
	git merge jshint-task
	git push


 



##Recap##
We split our tasks into individual modules and introduced `grunt.loadTasks` directive to pull these modules into our build file. We then added a new `lint` task to make sure our code was ship shape.