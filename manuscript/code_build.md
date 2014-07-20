#Building our app#
Now that we have a pipeline up and running it's time to turn our attention to dealing with building our code and assets and figure out how are our app will consume these. We do not want to check in our generated assets, however Heroku deploys using a git mechanism.

Let's start by creating some tasks to generate our css, then concatenate and ulgyfy our JS and we'll finish by deploying these assets to Heroku as part of a successful build. We'll also add some tasks to run these tasks on our local machine and have these re-generated when we save changes to the file. 

	git checkout -c generate-assets

_TODO: why do we generate our assets?_

##Compile our sass to css##
Let's tackle with our CSS files. The first thing we want to do is add the destination folder for our css content to the `.gitignore` list:

	node_modules
	.idea
	bower_components
	phantomjsdriver.log
	app/css/


Under our source folder let's create a scss folder and create a main.scss file in it. Here's content of the file:

	@import '../../bower_components/bootstrap-sass-official/vendor/assets/stylesheets/bootstrap';

Our build tasks will take this import directive and create us a nice main.css file that we can then use in our app.

	npm install --save-dev grunt-sass
	
And now let's create a task to generate our css files by adding to our `Gruntfile`:

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
            sass: {
                dist: {
                    options: {
                        sourceMap: true
                    },
                    files: {
                        'app/css/main.css': 'src/scss/main.scss'
                    }
                }
            },
        });

        grunt.loadNpmTasks('grunt-express-server');
        grunt.loadNpmTasks('grunt-selenium-webdriver');
        grunt.loadNpmTasks('grunt-cucumber');
        grunt.loadNpmTasks('grunt-sass');
        
        grunt.registerTask('build', ['sass']);

        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);

    };
    
When you run `grunt build`, you should see the following output:

	Running "sass:dist" (sass) task
	File app/css/main.css created.
	File app/css/main.css.map created.
	
If you were to start up our server and browse to localhost:3000, our UI should have more of a Bootstrap feel to it!

![Rendered HTML with generated css](Screenshot 2014-07-20 23.00.00.png)        	
Now bootstrap also needs some fonts, so let's move these across as part of the build.

	npm install grunt-contrib-copy --save-dev
	
And add a simple task to copy our fonts across as well:

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
            sass: {
                dist: {
                    options: {
                        sourceMap: true
                    },
                    files: {
                        'app/css/main.css': 'src/scss/main.scss'
                    }
                }
            },
            copy: {
                fonts: {
                    expand: true,
                    src: ['bower_components/bootstrap-sass-official/vendor/assets/fonts/bootstrap/*'],
                    dest: 'app/css/bootstrap/',
                    filter: 'isFile',
                    flatten: true
                }
            }
        });
    
        grunt.loadNpmTasks('grunt-express-server');
        grunt.loadNpmTasks('grunt-selenium-webdriver');
        grunt.loadNpmTasks('grunt-cucumber');
        grunt.loadNpmTasks('grunt-sass');
        grunt.loadNpmTasks('grunt-contrib-copy');
    
        grunt.registerTask('build', ['sass', 'copy:fonts']);
    
        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    
    };

Running `grunt build` task should now also copy our fonts across

![Fonts now included](Screenshot 2014-07-20 23.18.07.png)   

This is great, but how do we get this to run as part of our successful build? 

###Option 1 - postinstall script###
Well we can add a postinstall script to our package.json file!

    {
        "name": "weatherly",
        "version": "0.0.0",
        "description": "Building a web app guided by tests",
        "main": "index.js",
        "engines": {
            "node": "~0.10.28"
        },
        "scripts": {
            "test": "grunt test",
            "postinstall": "grunt build"
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
            "express": "^4.4.5",
            "grunt": "^0.4.5",
            "grunt-sass": "^0.14.0",
            "grunt-contrib-copy": "^0.5.0"
        },
        "devDependencies": {
            "chai": "^1.9.1",
            "cucumber": "^0.4.0",
            "grunt-cucumber": "^0.2.3",
            "grunt-express-server": "^0.4.17",
            "grunt-selenium-webdriver": "^0.2.420",
            "webdriverjs": "^1.7.1"
        }
    }

_didn't work very well with codeship_

###Option 2 - Heroku build packs###
heroku login

heroku config:add BUILDPACK_URL=https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git --app lit-meadow-5649

heroku config:set NODE_ENV=production --app lit-meadow-5649

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
            sass: {
                dist: {
                    options: {
                        sourceMap: true
                    },
                    files: {
                        'app/css/main.css': 'src/scss/main.scss'
                    }
                }
            },
            copy: {
                fonts: {
                    expand: true,
                    src: ['bower_components/bootstrap-sass-official/vendor/assets/fonts/bootstrap/*'],
                    dest: 'app/css/bootstrap/',
                    filter: 'isFile',
                    flatten: true
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
        grunt.loadNpmTasks('grunt-sass');
        grunt.loadNpmTasks('grunt-contrib-copy');
        grunt.loadNpmTasks('grunt-karma');
    
        grunt.registerTask('build', ['sass', 'copy:fonts']);
    
        grunt.registerTask('e2e', [
            'selenium_start',
            'express:test',
            'cucumberjs',
            'selenium_stop',
            'express:test:stop'
        ]);
    
        grunt.registerTask('unit', [
            'karma:unit'
        ]);
    
        grunt.registerTask('heroku:production', 'build');
    };
    
Re-jigg package.json:

    {
        "name": "weatherly",
        "version": "0.0.0",
        "description": "Building a web app guided by tests",
        "main": "index.js",
        "engines": {
            "node": "~0.10.28"
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
            "express": "^4.4.5",
            "grunt-sass": "^0.14.0",
            "grunt-contrib-copy": "^0.5.0"
        },
        "devDependencies": {
            "chai": "^1.9.1",
            "cucumber": "^0.4.0",
            "grunt": "^0.4.5",
            "grunt-contrib-copy": "^0.5.0",
            "grunt-cucumber": "^0.2.3",
            "grunt-express-server": "^0.4.17",
            "grunt-karma": "^0.8.3",
            "grunt-selenium-webdriver": "^0.2.420",
            "karma": "^0.12.17",
            "karma-jasmine": "^0.1.5",
            "karma-phantomjs-launcher": "^0.1.4",
            "webdriverjs": "^1.7.1"
        }
    }
    
_No luck get a segmentation fault on deploy when trying to generate sass assets_

* use browserify
* introduce backbone/underscore
* compile our modules

If you recall in our getting started section we set up our project and used [Bower](http://bower.io/) to manage our front end dependencies. For our code we will be [Browserify](http://browserify.org/) and adopting a CommonJS approach to dealing with modules and dependencies. 


_TODO: CommonJS vs AMD and why_

	npm install grunt-browserify --save-dev
	
Is all that is needed to install browserify and the reason we have chosen a grunt task is that we will