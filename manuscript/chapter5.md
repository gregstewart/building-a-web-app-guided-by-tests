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
        }
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
        }
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

##Recap##
We split our tasks into individual modules and introduced `grunt.loadTasks` directive to pull these modules into our build file.