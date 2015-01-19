#Fetching UI data#

Our goal was to display local weather forecast and to that end we'll start with implementing the AJAX request to fetch the data from [forecast.io](https://developer.forecast.io/). Well not quite, instead we'll be sending a request to our server, which in turn will proxy the request to  [forecast.io](https://developer.forecast.io/). Why? For security reason, we do not want to include our API key for to [forecast.io](https://developer.forecast.io/) in our UI code, since anyone could retrieve it and use it. We don't want that, using an approach where we proxy the request through our server, we can make sure that no one gets access to the key, short of having access to the server. Their are other benefits that you couple explore later on such as just returning the data you want or performing transformations on the data itself.

We'll start of as usualy by creating a branch:

	git checkout -b ui-display-model-data 
	
To help us organise our code, we'll use [Backbone.js](http://backbonejs.org/) and lucky for us there's an [NPM module](https://www.npmjs.com/package/backbone) as well.

	npm install backbone --save-dev
	
We could use [Bower](http://bower.io/) as well, however given that we use CommonJS and [Browserify](http://browserify.org/) this approach works well, and we can use [Backbone.js](http://backbonejs.org/) on the server side as well. However we won't be bundling the node module as part of our final bundle, instead we will leverage a CDN, however for testing purposes, we still need the framework.

We need to tell our test suite to include [Backbone.js](http://backbonejs.org/) as well, so edit the `karma.conf` file and look for the files block and once you have added: `'node_modules/backbone/backbone.js'` and `'node_modules/backbone/node_modules/underscore/underscore.js'`, it should look as follows:

	// list of files / patterns to load in the browser
    files: [
    	'node_modules/backbone/node_modules/underscore/underscore.js',
        'node_modules/backbone/backbone.js',
        'node_modules/weatherly/js/**/*.js',
        'tests/unit/**/*.js'
   	],

##A story for our code##

Given the following story

	As a user 
	I want to see the current weather conditions in my area
	So that I can decide whether to take an umbrella or not
	

Our first task task will be to display the current weather conditions for our vicinity. The acceptance criteria for this story might well state:

	Display the following:

 	* our location
 	* the actual temperature
 	* the apparent temperature
 	* the current weather conditions
 	* the weather conditions for the next hour
 	* the weather conditions for the next 24 hours
 	
	The temperature should be displayed in either Fahrenheit or Celsius depending on the location. The following coutries use Fahrenheit: the Bahamas, Belize, the Cayman Islands, Palau and the United States and its associated territories of American Samoa and the U.S. Virgin Islands (source http://en.wikipedia.org/wiki/Fahrenheit).


##The Model##
For our view we require a few things:

 * our location
 * the actual temperature
 * the apparent temperature
 * the current weather conditions
 * the weather conditions for the next hour
 * the weather conditions for the next 24 hours
 
So the model that backs our view should store that data, so let's review what the API returns. There's a lot of data in the JSON response from Forecast, but here's what we are interested in for the current weather conditions:
 
 	"currently":{"time":1419008168,"summary":"Clear","icon":"clear-night","nearestStormDistance":122,"nearestStormBearing":330,"precipIntensity":0,"precipProbability":0,"temperature":47.39,"apparentTemperature":41.78,"dewPoint":36.63,"humidity":0.66,"windSpeed":13.19,"windBearing":267,"visibility":9.32,"cloudCover":0.01,"pressure":1019.39,"ozone":295.69}
 	
There's also a `hourly` block that we'll use for the last 2 items. Our model will therefore have the following attributes:

* location - _we'll need to derive that from either the timezone or another source_
* temperature - currently:temperature
* apparentTemperature - currently:temperature
* currentWeatherConditions - currently:summary
* weatherConditionsInHour - hourly:data[1]:summary
* weatherConditionsInTwentyFourHours - hourly:data[23]:summary

Setting these properties would not make for a very useful test as we would just be validating that BackBone is doing it's job, of more interest though is the fact that we need to display the temperatures in either Fahrenheit or Celsius based of our location. That makes for some more interesting tests. Having said that let's start by intialising our `TodaysWeather`class and making sure we have a temperatures set as our first test, before we move on to transforming the values into Celsius if appropriate.

Given that we have nothing let's start with the simplest of tests though. At the terminal type:

	grunt karma:dev

This starts our test runner and runs the tests each time we save our code. Open up the `TodaysWeather-spec.js` test class we created in the Unit Test chapter and add the following code and save it:
    
    'use strict';
    
    var TodaysWeather = require('weatherly/js/model/TodaysWeather');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
        });
    
        it('stores the values passed in', function () {
            var todaysWeather = new TodaysWeather({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature});
    
            expect(todaysWeather.get('location')).toBe(this.location);
            expect(todaysWeather.get('temperature')).toBe(this.temperature);
            expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
        });
    });

The console with test runner should show something like the following:    
 
	Running "karma:dev" (karma) task
	INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	INFO [launcher]: Starting browser PhantomJS
	INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 3j7gCnFkGjjUdhbmJ_YS with id 10391093
	PhantomJS 1.9.7 (Mac OS X) Today 's weather stores the values passed in FAILED
        TypeError: 'undefined' is not an object (evaluating 'this.todaysWeather.location')
            at /Users/gregstewart/Projects/github/weatherly/tests/unit/model/TodaysWeather-spec.js:8
            at /Users/gregstewart/Projects/github/weatherly/node_modules/karma-jasmine/lib/adapter.js:166
            at http://localhost:9876/karma.js:189
            at http://localhost:9876/context.html:43
	PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 (1 FAILED) ERROR (0.005 secs / 0.002 secs)

	=============================== Coverage summary ===============================
	Statements   : 83.33% ( 5/6 )
	Branches     : 100% ( 2/2 )
	Functions    : 50% ( 1/2 )
	Lines        : 66.67% ( 2/3 )
	================================================================================
	
Which tells us that this.todaysWeather.location is undefined, which makes sense since we haven't defined this.todaysWeather or this.todaysWeather.location. Let's rectify this:

    'use strict';
    
    var TodaysWeather = require('weatherly/js/model/TodaysWeather');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
        });
    
        it('stores the values passed in', function () {
            var todaysWeather = new TodaysWeather({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature});
    
            expect(todaysWeather.get('location')).toBe(this.location);
            expect(todaysWeather.get('temperature')).toBe(this.temperature);
            expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
        });
    });
    
That gets us a step closer but the test still fails because we haven't implemented anything:

    var TodaysWeather = Backbone.Model.extend({
    
    });
    
    module.exports = TodaysWeather;
    
And that is all we need for now and the test should be green:

	INFO [watcher]: Changed file "/Users/gregstewart/Projects/github/weatherly/tests/unit/model/TodaysWeather-spec.js".
	PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.008 secs / 0.004 secs)

	=============================== Coverage summary ===============================
	Statements   : 100% ( 5/5 )
	Branches     : 100% ( 2/2 )
	Functions    : 100% ( 1/1 )
	Lines        : 100% ( 2/2 )
	================================================================================
	
With that out of the way we can tackle converting our temperatures from Fahrenheit to Celcius if the location matches one of these countries: the Bahamas (BS), Belize (BZ), the Cayman Islands (KY), Palau (PW) and the United States (US) and its associated territories of American Samoa (AS) and the U.S. Virgin Islands (VI). The formula looks like this: `([°F] − 32) × 5⁄9`. Let's add some tests for that:

    'use strict';
    
    var TodaysWeather = require('weatherly/js/model/TodaysWeather');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
        });
    
        it('stores the values passed in', function () {
            var todaysWeather = new TodaysWeather({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature});
    
            expect(todaysWeather.get('location')).toBe(this.location);
            expect(todaysWeather.get('temperature')).toBe(this.temperature);
            expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
        });
    
        describe('Temperature conversion', function () {
            describe('given we are in the "UK"', function () {
                it('converts to Celsius', function () {
                    var expectedTemperature = (this.temperature - 32) * (5/9),
                        expectedApparentTemperature = (this.apparentTemperature - 32) * (5/9),
                        todaysWeather = new TodaysWeather({ location: 'London',
                        country: 'UK',
                        temperature: this.temperature,
                        apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(expectedTemperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(expectedApparentTemperature);
                });
            });

            describe('given we are in a country that uses Fahrenheit', function () {
                it('does not convert to Celsius when we are in Nassau', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Nassau',
                            country: 'BS',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Boston', function () {
                    var todaysWeather = new TodaysWeather({ location: this.location,
                            country: this.country,
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Charlotte Amalie', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Charlotte Amalie',
                            country: 'VI',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Pago Pago', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Pago Pago',
                            country: 'AS',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Ngerulmud', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Ngerulmud',
                            country: 'PW',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in George Town', function () {
                    var todaysWeather = new TodaysWeather({ location: 'George Town',
                            country: 'KY',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Belmopan', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Belmopan',
                            country: 'BZ',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);
                    expect(todaysWeather.get('apparentTemperature')).toBe(this.apparentTemperature);
                });
            });
        });
    });
        
In order to make this test pass we need to implement a converter function and it will be called when the values for `temperature` and `apparentTemperature` are initially set. Here's the code that implements our converter and is called on initial set.

    var TodaysWeather = Backbone.Model.extend({
        initialize: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
        },
        convertTemperature: function (country, temperature) {
            var countriesToIgnore = ['BS', 'BZ', 'KY', 'PW', 'US', 'AS', 'VI'];
    
            if(countriesToIgnore.indexOf(country) === -1) {
                return (temperature - 32) * (5/9);
            }
    
            return temperature;
        }
    });
    
    module.exports = TodaysWeather;
    
And with that all of the tests pass once again:

	INFO [watcher]: Changed file "/Users/gregstewart/Projects/github/weatherly/node_modules/weatherly/js/model/TodaysWeather.js".
	PhantomJS 1.9.7 (Mac OS X): Executed 9 of 9 SUCCESS (0.009 secs / 0.006 secs)

	=============================== Coverage summary ===============================
	Statements   : 100% ( 11/11 )
	Branches     : 100% ( 4/4 )
	Functions    : 100% ( 3/3 )
	Lines        : 100% ( 8/8 )
	================================================================================
	
This conversion should also take place if either value changes:

	        ...
			
			describe('Temperature changes afterwards', function () {

                it('converts the temperature value if we are in the "UK"', function () {
                    var expected = (this.temperature - 32) * (5/9),
                        changeExpected = (99 - 32) * (5/9),
                        todaysWeather = new TodaysWeather({ location: 'London',
                            country: 'UK',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(expected);

                    todaysWeather.set('temperature', 99);
                    expect(todaysWeather.get('temperature')).toBe(changeExpected);
                });

                it('converts the apparentTemperature value if we are in the "UK"', function () {
                    var expected = (this.apparentTemperature - 32) * (5/9),
                        changeExpected = (80 - 32) * (5/9),
                        todaysWeather = new TodaysWeather({ location: 'London',
                            country: 'UK',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('apparentTemperature')).toBe(expected);

                    todaysWeather.set('apparentTemperature', 80);
                    expect(todaysWeather.get('apparentTemperature')).toBe(changeExpected);
                });

                it('does not convert the temperature value if we are in the US', function () {
                    var todaysWeather = new TodaysWeather({ location: 'Boston',
                            country: 'US',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeather.get('temperature')).toBe(this.temperature);

                    todaysWeather.set('temperature', 80);
                    expect(todaysWeather.get('temperature')).toBe(80);
                });
         	});
 
 Let's get these tests passing as well:

	'use strict';	
 
     var TodaysWeather = Backbone.Model.extend({
        initialize: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));

            this.on('change:temperature', this.temperatureChanged, this);
            this.on('change:apparentTemperature', this.apparentTemperatureChanged, this);
        },
        temperatureChanged: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
        },
        apparentTemperatureChanged: function () {
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
        },
        convertTemperature: function (country, temperature) {
            var countriesToIgnore = ['BS', 'BZ', 'KY', 'PW', 'US', 'AS', 'VI'];

            if(countriesToIgnore.indexOf(country) === -1) {
                return (temperature - 32) * (5/9);
            }

            return temperature;
        }
    });

    module.exports = TodaysWeather;
    
#### Red, Green, Refactor ####
There are few improvements we can make to this code, now that all of our tests are passing and our coverage looks solid, it's time for the refactor phase:
	
	'use strict';
	
	var TodaysWeather = Backbone.Model.extend({
        initialize: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));

            this.on('change:temperature', this.temperatureChanged, this);
            this.on('change:apparentTemperature', this.apparentTemperatureChanged, this);
        },
        temperatureChanged: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
        },
        apparentTemperatureChanged: function () {
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
        },
        convertTemperature: function (country, temperature) {
            if(this.isCelsiusCountry(country)) {
                return this.convertToCelsius(temperature);
            }

            return temperature;
        },
        isCelsiusCountry: function (country) {
            return ['BS', 'BZ', 'KY', 'PW', 'US', 'AS', 'VI'].indexOf(country) === -1;
        },
        convertToCelsius: function (temperature) {
            return (temperature - 32) * (5/9);
        }
    });

    module.exports = TodaysWeather;
    
A final scenario that springs to mind is what happens if the location changes? What if travelled from the US to the UK? We should deal with this as well

##Before checkin
Now that we are linting our code, if we ran the `jshint` task we would see the following error:

	Running "jshint:source" (jshint) task
   	node_modules/weatherly/js/model/TodaysWeather.js
      	3 |var TodaysWeather = Backbone.Model.extend({
                             ^ 'Backbone' is not defined.
                             
`jshint` is not actually running our code and as a result does not know that we are including Backbone as a dependency. We can fix this by editing our `.jshintrc` file and telling it to include it as a global (on the third to last line):

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
            "jasmine": false,
            "Backbone": false
        }
    }

We should also not forget to lint our test code, so let's update the `jshint` task:

    (function (module) {
        'use strict';
        var config = {
            options: {
                jshintrc: './.jshintrc'
            },
            source: {
                src: [
                    './Gruntfile.js',
                    './build/grunt/**/*.js',
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