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

Visually we will be working with this area of the UI:

![Rendered HTML hosted by our Connect server](images/Screenshot 2015-02-22 13.37.26.png)

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

Setting these properties would not make for a very useful test as we would just be validating that BackBone is doing it's job, of more interest though is the fact that we need to display the temperatures in either Fahrenheit or Celsius based of our location. That makes for some more interesting tests. Having said that let's start by intialising our `TodaysWeatherForecast`class and making sure we have a temperatures set as our first test, before we move on to transforming the values into Celsius if appropriate.

Given that we have nothing let's start with the simplest of tests though. At the terminal type:

	grunt karma:dev

This starts our test runner and runs the tests each time we save our code. Open up the `TodaysWeatherForecast-spec.js` test class we created in the Unit Test chapter and add the following code and save it:
    
    'use strict';
    
    var TodaysWeatherForecast = require('weatherly/js/model/TodaysWeatherForecast');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
            this.currentWeatherConditions = 'Clear';
        });
    
        it('stores the values passed in', function () {
            var todaysWeatherForecast = new TodaysWeatherForecast({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature,
            	currentWeatherConditions: this.currentWeatherConditions});
    
            expect(todaysWeatherForecast.get('location')).toBe(this.location);
            expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
            expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
	    expect(todaysWeatherForecast.get('currentWeatherConditions')).toBe(this.currentWeatherConditions);
        });
    });

The console with test runner should show something like the following:    
 
	Running "karma:dev" (karma) task
	INFO [karma]: Karma v0.12.17 server started at http://localhost:9876/
	INFO [launcher]: Starting browser PhantomJS
	INFO [PhantomJS 1.9.7 (Mac OS X)]: Connected on socket 3j7gCnFkGjjUdhbmJ_YS with id 10391093
	PhantomJS 1.9.7 (Mac OS X) Today 's weather stores the values passed in FAILED
        TypeError: 'undefined' is not an object (evaluating 'this.TodaysWeatherForecast.location')
            at /Users/gregstewart/Projects/github/weatherly/tests/unit/model/TodaysWeatherForecast-spec.js:8
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
	
Which tells us that this.TodaysWeatherForecast.location is undefined, which makes sense since we haven't defined this.TodaysWeatherForecast or this.TodaysWeatherForecast.location. Let's rectify this:

    'use strict';
    
    var TodaysWeatherForecast = require('weatherly/js/model/TodaysWeatherForecast');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
        });
    
        it('stores the values passed in', function () {
            var todaysWeatherForecast = new TodaysWeatherForecast({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature});
    
            expect(todaysWeatherForecast.get('location')).toBe(this.location);
            expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
            expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
        });
    });
    
That gets us a step closer but the test still fails because we haven't implemented anything:

    var TodaysWeatherForecast = Backbone.Model.extend({
    
    });
    
    module.exports = TodaysWeatherForecast;
    
And that is all we need for now and the test should be green:

	INFO [watcher]: Changed file "/Users/gregstewart/Projects/github/weatherly/tests/unit/model/TodaysWeatherForecast-spec.js".
	PhantomJS 1.9.7 (Mac OS X): Executed 1 of 1 SUCCESS (0.008 secs / 0.004 secs)

	=============================== Coverage summary ===============================
	Statements   : 100% ( 5/5 )
	Branches     : 100% ( 2/2 )
	Functions    : 100% ( 1/1 )
	Lines        : 100% ( 2/2 )
	================================================================================
	
With that out of the way we can tackle converting our temperatures from Fahrenheit to Celcius if the location matches one of these countries: the Bahamas (BS), Belize (BZ), the Cayman Islands (KY), Palau (PW) and the United States (US) and its associated territories of American Samoa (AS) and the U.S. Virgin Islands (VI). The formula looks like this: `([°F] − 32) × 5⁄9`. Let's add some tests for that:

    'use strict';
    
    var TodaysWeatherForecast = require('weatherly/js/model/TodaysWeatherForecast');
    
    describe('Today \'s weather', function () {
    
        beforeEach(function () {
            this.location = 'Boston';
            this.country = 'US';
            this.temperature = 47.39;
            this.apparentTemperature = 41.78;
        });
    
        it('stores the values passed in', function () {
            var todaysWeatherForecast = new TodaysWeatherForecast({ location: this.location,
                country: this.country,
                temperature: this.temperature,
                apparentTemperature: this.apparentTemperature});
    
            expect(todaysWeatherForecast.get('location')).toBe(this.location);
            expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
            expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
        });
    
        describe('Temperature conversion', function () {
            describe('given we are in the "UK"', function () {
                it('converts to Celsius', function () {
                    var expectedTemperature = (this.temperature - 32) * (5/9),
                        expectedApparentTemperature = (this.apparentTemperature - 32) * (5/9),
                        todaysWeatherForecast = new TodaysWeatherForecast({ location: 'London',
                        country: 'UK',
                        temperature: this.temperature,
                        apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(expectedTemperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(expectedApparentTemperature);
                });
            });

            describe('given we are in a country that uses Fahrenheit', function () {
                it('does not convert to Celsius when we are in Nassau', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Nassau',
                            country: 'BS',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Boston', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: this.location,
                            country: this.country,
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Charlotte Amalie', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Charlotte Amalie',
                            country: 'VI',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Pago Pago', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Pago Pago',
                            country: 'AS',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Ngerulmud', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Ngerulmud',
                            country: 'PW',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in George Town', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'George Town',
                            country: 'KY',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });

                it('does not convert to Celsius when we are in Belmopan', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Belmopan',
                            country: 'BZ',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(this.apparentTemperature);
                });
            });
        });
    });
        
In order to make this test pass we need to implement a converter function and it will be called when the values for `temperature` and `apparentTemperature` are initially set. Here's the code that implements our converter and is called on initial set.

    var TodaysWeatherForecast = Backbone.Model.extend({
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
    
    module.exports = TodaysWeatherForecast;
    
And with that all of the tests pass once again:

	INFO [watcher]: Changed file "/Users/gregstewart/Projects/github/weatherly/node_modules/weatherly/js/model/TodaysWeatherForecast.js".
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
                        todaysWeatherForecast = new TodaysWeatherForecast({ location: 'London',
                            country: 'UK',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(expected);

                    TodaysWeatherForecast.set('temperature', 99);
                    expect(todaysWeatherForecast.get('temperature')).toBe(changeExpected);
                });

                it('converts the apparentTemperature value if we are in the "UK"', function () {
                    var expected = (this.apparentTemperature - 32) * (5/9),
                        changeExpected = (80 - 32) * (5/9),
                        todaysWeatherForecast = new TodaysWeatherForecast({ location: 'London',
                            country: 'UK',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(expected);

                    TodaysWeatherForecast.set('apparentTemperature', 80);
                    expect(todaysWeatherForecast.get('apparentTemperature')).toBe(changeExpected);
                });

                it('does not convert the temperature value if we are in the US', function () {
                    var todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Boston',
                            country: 'US',
                            temperature: this.temperature,
                            apparentTemperature: this.apparentTemperature});

                    expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);

                    TodaysWeatherForecast.set('temperature', 80);
                    expect(todaysWeatherForecast.get('temperature')).toBe(80);
                });
         	});
 
 Let's get these tests passing as well:
 
     var TodaysWeatherForecast = Backbone.Model.extend({
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

    module.exports = TodaysWeatherForecast;
    
#### Red, Green, Refactor ####
There are few improvements we can make to this code, now that all of our tests are passing and our coverage looks solid, it's time for the refactor phase. What is refactoring? Martin Fowler [put it best in his book](http://refactoring.com/):

> …is a disciplined technique for restructuring an existing body of code, altering its internal structure without changing its external behavior.

> Its heart is a series of small behavior preserving transformations. Each transformation (called a “refactoring”) does little, but a sequence of transformations can produce a significant restructuring. Since each refactoring is small, it’s less likely to go wrong. The system is kept fully working after each small refactoring, reducing the chances that a system can get seriously broken during the restructuring.

Let's start by applying the `extract method` technique ([see here for an example](http://refactoring.com/catalog/extractMethod.html)):

	var TodaysWeatherForecast = Backbone.Model.extend({
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

    module.exports = TodaysWeatherForecast;
    
Two methods were extracted, one for figuring out whether the country we are in wants the temperature in Celsius or Fahrenheit (`shouldConvertToCelsius`) and another to do the actual conversion from Fahrenheit to Celsius (`convertToCelsius`). The upshot of this step is that  the `convertTemperature` is easier to read and simpler to understand.
    
###Where another requirement emerges
Just as you think you are done, your product owner points out the following scenario: "What happens if the location changes? What if travelled from the US to the UK? Shouldn't the temperature be updated as well?" We should deal with this and here are the tests to cover this:
	
	...
		describe('Location changes', function () {
            it('should convert if we were in a Fahrenheit country but are now in the "UK"', function () {
                var expected = (this.temperature - 32) * (5/9),
                    todaysWeatherForecast = new TodaysWeatherForecast({ location: 'Boston',
                    country: 'US',
                    temperature: this.temperature,
                    apparentTemperature: this.apparentTemperature});

                expect(todaysWeatherForecast.get('temperature')).toBe(this.temperature);

                TodaysWeatherForecast.set('location', 'UK');
                expect(todaysWeatherForecast.get('temperature')).toBe(expected);
            });

            it('should convert if we were in a Celsius country but are now in the "US"', function () {
                var originalTemperature = this.temperature,
                    expected = (this.temperature - 32) * (5/9),
                    todaysWeatherForecast = new TodaysWeatherForecast({ location: 'London',
                        country: 'UK',
                        temperature: this.temperature,
                        apparentTemperature: this.apparentTemperature});

                expect(todaysWeatherForecast.get('temperature')).toBe(expected);

                TodaysWeatherForecast.set('location', 'US');
                expect(todaysWeatherForecast.get('temperature')).toBe(originalTemperature);
            });
        });


The code to make these tests pass:

    'use strict';
    
    var TodaysWeatherForecast = Backbone.Model.extend({
        initialize: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
    
            this.on('change:temperature', this.temperatureChanged, this);
            this.on('change:apparentTemperature', this.apparentTemperatureChanged, this);
            this.on('change:country', this.countryHasChanged, this);
        },
        temperatureChanged: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
        },
        apparentTemperatureChanged: function () {
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
        },
        countryHasChanged: function () {
            if (this._previousAttributes.country !== this.get('country')) {
                if(this.shouldConvertToCelsius(this.get('country'))) {
                    this.attributes.temperature = this.convertToCelsius(this.get('temperature'));
                    this.attributes.apparentTemperature = this.convertToCelsius(this.get('apparentTemperature'));
                } else {
                    this.attributes.temperature = this.convertToFahrenheit(this.get('temperature'));
                    this.attributes.apparentTemperature = this.convertToFahrenheit(this.get('apparentTemperature'));
                }
            }
        },
        convertTemperature: function (country, temperature) {
            if(this.shouldConvertToCelsius(country)) {
                return this.convertToCelsius(temperature);
            }
    
            return temperature;
        },
        shouldConvertToCelsius: function (country) {
            return ['BS', 'BZ', 'KY', 'PW', 'US', 'AS', 'VI'].indexOf(country) === -1;
        },
        convertToCelsius: function (temperature) {
            return (temperature - 32) * (5/9);
        },
        convertToFahrenheit: function (temperature) {
            return (temperature * (9/5)) + 32;
        }
    });
    
    module.exports = TodaysWeatherForecast;
    
At this stage we can once again do some more refactoring. The functions for converting temperatures have nothing to with Today's Weather object, so let's extract these into it's own module `TemperatureConverter.js`:

    'use strict';
    
    var TemperatureConverter = function () {
        return {
            toCelsius: function (temperature) {
                return (temperature - 32) * (5/9);
            },
            toFahrenheit: function (temperature) {
                return (temperature * (9/5)) + 32;
            },
            shouldConvertToCelsius: function (country) {
                return ['BS', 'BZ', 'KY', 'PW', 'US', 'AS', 'VI'].indexOf(country) === -1;
            },
        	convert: function (country, temperature) {
            	if (this.shouldConvertToCelsius(country)) {
                	return this.toCelsius(temperature);
	            }
	
    	        return temperature;
        	}
        }
    }
    
    module.exports = TemperatureConverter;
    
This technique is called `extract class` ([see here for an example](http://refactoring.com/catalog/extractClass.html)). Next let's replace the code in those functions with calls to the API exposed by our new object:

    'use strict';
    
    var TemperatureConverter = require('weatherly/js/model/TemperatureConverter');
    
    var TodaysWeatherForecast = Backbone.Model.extend({
        initialize: function () {
            this.temperatureConverter = new TemperatureConverter();
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
    
            this.on('change:temperature', this.temperatureChanged, this);
            this.on('change:apparentTemperature', this.apparentTemperatureChanged, this);
            this.on('change:country', this.countryHasChanged, this);
        },
        temperatureChanged: function () {
            this.attributes.temperature = this.convertTemperature(this.get('country'), this.get('temperature'));
        },
        apparentTemperatureChanged: function () {
            this.attributes.apparentTemperature = this.convertTemperature(this.get('country'), this.get('apparentTemperature'));
        },
        countryHasChanged: function () {
            if (this._previousAttributes.country !== this.get('country')) {
                if (this.shouldConvertToCelsius(this.get('country'))) {
                    this.attributes.temperature = this.convertToCelsius(this.get('temperature'));
                    this.attributes.apparentTemperature = this.convertToCelsius(this.get('apparentTemperature'));
                } else {
                    this.attributes.temperature = this.convertToFahrenheit(this.get('temperature'));
                    this.attributes.apparentTemperature = this.convertToFahrenheit(this.get('apparentTemperature'));
                }
            }
        },
        convertTemperature: function (country, temperature) {
            return this.temperatureConverter.convert(country, temperature);
        },
        shouldConvertToCelsius: function (country) {
            return this.temperatureConverter.shouldConvertToCelsius(country);
        },
        convertToCelsius: function (temperature) {
            return this.temperatureConverter.toCelsius(temperature);
        },
        convertToFahrenheit: function (temperature) {
            return this.temperatureConverter.toFahrenheit(temperature);
        }
    });
    
    module.exports = TodaysWeatherForecast;
    
Test should still be passing, so we can continue with our refactoring by replacing the wrapper calls with direct calls to the API, one method at a time, giving us:

    'use strict';
    
    var TemperatureConverter = require('weatherly/js/model/TemperatureConverter');
    
    var TodaysWeatherForecast = Backbone.Model.extend({
        initialize: function () {
            this.temperatureConverter = new TemperatureConverter();
    
            this.attributes.temperature = this.temperatureConverter.convert(this.get('country'), this.get('temperature'));
            this.attributes.apparentTemperature = this.temperatureConverter.convert(this.get('country'), this.get('apparentTemperature'));
    
            this.on('change:temperature', this.temperatureChanged, this);
            this.on('change:apparentTemperature', this.apparentTemperatureChanged, this);
            this.on('change:country', this.countryHasChanged, this);
        },
        temperatureChanged: function () {
            this.attributes.temperature = this.temperatureConverter.convert(this.get('country'), this.get('temperature'));
        },
        apparentTemperatureChanged: function () {
            this.attributes.apparentTemperature = this.temperatureConverter.convert(this.get('country'), this.get('apparentTemperature'));
        },
        countryHasChanged: function () {
            if (this._previousAttributes.country !== this.get('country')) {
                if (this.temperatureConverter.shouldConvertToCelsius(this.get('country'))) {
                    this.attributes.temperature = this.temperatureConverter.toCelsius(this.get('temperature'));
                    this.attributes.apparentTemperature = this.temperatureConverter.toCelsius(this.get('apparentTemperature'));
                } else {
                    this.attributes.temperature = this.temperatureConverter.toFahrenheit(this.get('temperature'));
                    this.attributes.apparentTemperature = this.temperatureConverter.toFahrenheit(this.get('apparentTemperature'));
                }
            }
        }
    });
    
    module.exports = TodaysWeatherForecast;
    
There's one more thing we can improve on in this object. Nested `if` statements are a problem, hard to read and hard to reason about. We can improve the readability of the `countryHasChanged` function by implemeting [a guard clause](http://refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html):

    countryHasChanged: function () {
        if (this._previousAttributes.country === this.get('country')) {
            	return;
        }
        if (this.temperatureConverter.shouldConvertToCelsius(this.get('country'))) {
        	this.attributes.temperature = this.temperatureConverter.toCelsius(this.get('temperature'));
        	this.attributes.apparentTemperature = this.temperatureConverter.toCelsius(this.get('apparentTemperature'));
        } else {
        	this.attributes.temperature = this.temperatureConverter.toFahrenheit(this.get('temperature'));
        	this.attributes.apparentTemperature = this.temperatureConverter.toFahrenheit(this.get('apparentTemperature'));
        }
    }

### Before checkin
Now that we are linting our code, if we ran the `jshint` task we would see the following error:

	Running "jshint:source" (jshint) task
   	node_modules/weatherly/js/model/TodaysWeatherForecast.js
      	3 |var TodaysWeatherForecast = Backbone.Model.extend({
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

## The View
Let's move on from our model and turn our attention to our `View`. We'll start off by creating a view object that will use static data to render the page fragment. Once we are happy that this works, we'll make things a little more dynamic by using model data for the relevant view fragments.

To avoid any errors during testing we need to edit our `karma.conf.js` and tell it to include `jQuery` since Backbone needs it:
	
    // list of files / patterns to load in the browser
    files: [
    	'bower_components/jquery/dist/jquery.js',
        'node_modules/backbone/node_modules/underscore/underscore.js',
        'node_modules/backbone/backbone.js',
        'node_modules/weatherly/js/**/*.js',
        'tests/unit/**/*.js'
    ],
    
So let's start with a test. Create `TodaysWeatherForecastView-spec.js` in `/unit/view/` and add the following code:

	'use strict';

	var TodaysWeatherForecastView = require('weatherly/js/view/TodaysWeatherForecast');

	describe('Today \'s weather view', function () {
    	var view;

    	beforeEach(function () {
        	view = new TodaysWeatherForecastView();
        	view.render();
        });

    	it('creates a container for the weather right now', function () {
        	expect(view.el.id).toBe('right-now');
    	});
	});

This is a very simple test that makes sure when we initialise our view object it creates an `el` element with and Id of `right-now`. And the code to make this pass is:

	'use strict';

	var TodaysWeatherForecastView = Backbone.View.extend({
    	tagName: 'div',
    	id: 'right-now',
    	render: function() {
        	return this;
    	}
	});

	module.exports = TodaysWeatherForecastView;

With that done let's add a template and render it. We'll be making use of `underscore` [templates](http://underscorejs.org/#template). They are simple to use and are already included by virtue of Backbone's dependency on `Underscore.js`. If you wish you could [choose any other template engine](http://backbonejs.org/#View-template) out there to render your views. Here are a few you could use:

* [Mustache.js](http://github.com/janl/mustache.js)
* [React](http://facebook.github.io/react/)
* [Haml-js](http://github.com/creationix/haml-js)

Our mark-up in our `index.html` file shows that we display `<h1>London Right Now</h1>`, so let's start with this by adding the following test:

    'use strict';
    
    var TodaysWeatherForecastView = require('weatherly/js/view/TodaysWeatherForecast');
    
    describe('Today \'s weather view', function () {
        var view;
    
        beforeEach(function () {
            view = new TodaysWeatherForecastView();
            view.render();
        });
    
        it('creates a container for the weather right now', function () {
            expect(view.el.id).toBe('right-now');
        });
    
        it('has a header element', function () {
            expect(view.el.querySelector('h1').innerText).toBe('London Right Now');
        });
    });
    
To make the test pass, let's create a helper function called `header` which calls the templating engine and then we inject the result into the `el` element we created previously using `jQuery`:

    'use strict';
    
    var TodaysWeatherForecastView = Backbone.View.extend({
        tagName: 'div',
        id: 'right-now',
        header: _.template('<h1>London Right Now</h1>'),
        render: function() {
            this.$el.html(this.header());
            return this;
        }
    });
    
    module.exports = TodaysWeatherForecastView;
    
So that's our static approach done, let's make this data driven by using a model attribute. `London` in our example is the part of the string that is variable and dynamic. If you recall we created a model with a `location` attribute, which is perfect for this. So let's amend our `header` helper method in our `view` code a little to use interpolation (`<%- location %>`). Furthermore we will need to pass in the value to that helper method (`this.model.get('location')`):

    'use strict';
    
    var TodaysWeatherForecastView = Backbone.View.extend({
        tagName: 'div',
        id: 'right-now',
        header: _.template('<h1><%- location %> Right Now</h1>'),
        render: function() {
            this.$el.html(this.header({location: this.model.get('location')}));
            return this;
        }
    });
    
    module.exports = TodaysWeatherForecastView;
    
When you save the file you will see that our tests now fail, because `this.model` is undefined. So we need to fix how instantiate the view. We do this by passing in a model. Now we don't need to actually use the model we created, we can use a `Backbone.Model` and set a `location` attribute and give it a value. When writing unit tests we generally want to limit the interaction between objects to a minimum, and while we could have used other tools to `mock` our model, in this case the basic `Backbone.Model` is good enough for now. We will discuss `mock`s, `stub`s and `fake`s later on.

    'use strict';
    
    var TodaysWeatherForecastView = require('weatherly/js/view/TodaysWeatherForecast');
    
    describe('Today \'s weather view', function () {
        var view;
    
        beforeEach(function () {
            var model = new Backbone.Model({location: 'London'});
            view = new TodaysWeatherForecastView({model: model});
            view.render();
        });
    
        it('creates a container for the weather right now', function () {
            expect(view.el.id).toBe('right-now');
        });
    
        it('has a header element', function () {
            expect(view.el.querySelector('h1').innerText).toBe('London Right Now');
        });
    });
    
Test should be green again, so we let's continue by adding the next few elements for that area of the display:

    'use strict';

    var TodaysWeatherForecastView = require('weatherly/js/view/TodaysWeatherForecast');

    describe('Today \'s weather view', function () {
        var view;

        beforeEach(function () {
            var model = new Backbone.Model({location: 'London', temperature: '14', currentWeatherConditions: 'Clear', apparentTemperature: '14'});
            view = new TodaysWeatherForecastView({model: model});
            view.render();
        });

        it('creates a container for the weather right now', function () {
            expect(view.el.id).toBe('right-now');
        });

        it('has a header element', function () {
            expect(view.el.querySelector('h1').innerText).toBe('London Right Now');
        });

        it('has a temperature element', function () {
            expect(view.el.querySelector('p.temperature').innerText).toBe('14 degrees');
        });

        it('has a feels-like element', function () {
            expect(view.el.querySelector('p.feels-like').innerText).toBe('Clear - feels like 14 degrees');
        });
    });

And now the code to make this pass: 

    'use strict';

    var TodaysWeatherForecastView = Backbone.View.extend({
        tagName: 'div',
        id: 'right-now',
        header: _.template('<h1><%- location %> Right Now</h1>'),
        temperature: _.template('<p class="temperature"><%- temperature %> degrees</p>'),
        feelsLike: _.template('<p class="feels-like"><%- currentWeatherConditions %> - feels like <%- apparentTemperature %> degrees</p>'),
        render: function() {
            var header = this.header({location: this.model.get('location')}),
                temperature = this.temperature({temperature: this.model.get('temperature')}),
                feelsLike = this.feelsLike({currentWeatherConditions: this.model.get('currentWeatherConditions'), apparentTemperature: this.model.get('apparentTemperature')});

            this.$el.append(header).append(temperature).append(feelsLike);
            return this;
        }
    });

    module.exports = TodaysWeatherForecastView;
    
We are now in a good spot, we are now in a position to render all of the content we displayed inside of our `jumbotron` element. We are not quite done yet, it's once again time to improve our code. As a rule you shouldn't mix your JS with HTML let's extract all of the templates, starting with our header, let's create a `template` folder and add the following to a `header.js` file:

    'use strict';
    
    var header = _.template('<h1><%- location %> Right Now</h1>');
    
    module.exports = header;
    
The other two follow the same pattern, here is `temperature.js`:
    
    'use strict';

	var temperature = _.template('<p class="temperature"><%- temperature %> degrees</p>');

	module.exports = temperature;
	
And lastly `feelsLike.js`: 

    'use strict';
    
    var feelsLike = _.template('<p class="feels-like"><%- currentWeatherConditions %> - feels like <%- apparentTemperature %> degrees</p>');
    
    module.exports = feelsLike;

With that we can require them into our view and assign them to the context of the view by adding an `initialize` method: 

    'use strict';
    
    var header = require('weatherly/js/templates/header'),
        temperature = require('weatherly/js/templates/temperature'),
        feelsLike = require('weatherly/js/templates/feelsLike');
    
    var TodaysWeatherForecastView = Backbone.View.extend({
        initialize: function() {
            this.header = header;
            this.temperature = temperature;
            this.feelsLike = feelsLike;
        },
        tagName: 'div',
        id: 'right-now',
        render: function() {
            this.$el.append(this.header({location: this.model.get('location')}))
                .append(this.temperature({temperature: this.model.get('temperature')}))
                .append(this.feelsLike({currentWeatherConditions: this.model.get('currentWeatherConditions'), apparentTemperature: this.model.get('apparentTemperature')}));
            return this;
        }
    });
    
    module.exports = TodaysWeatherForecastView;
    
You might be tempted to pass in the model as an argument to the templates to keep the number of arguments down, but that's not a good design. If you do that the templates all of a sudden need to have knowledge about an object external to them. In other words if you passed in model, then the template would need to know about the `get` method on the object and the actual attribute name, e.g. `location`. Furthermore you would have coupled the template to `BackBone.Model`

We could also fix duplication in our temperature templates, however I am going to hold off for a little longer. To paraphrase [Sandy Metz](http://www.sandimetz.com/):

> fixing duplication is easier than fixing the wrong absraction

So let's wait until we have dealt with all of the ways we display temperature in our UI.

__TODO can we make render open closed?__

At this stage let's commit all of our work, but before we do we need to update .jshintrc so that it includes `_` (a.ka. `Underscore.js`), otherwise we would have linting errors on commit:
    
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
            "_": false,
            "Backbone": false
        }
    }

Right let's commit:

	$ git add .
	$ git commit -m "Hero content uses BackBone.View and templaes to render"


## The Router
Ok View and Model are done, time to tackle the glue that binds our Model to our View, a `BackBone.Router` ([documentation](http://backbonejs.org/#Router)). Our tests fall into two categories: routes and router. The routes part of the tests wil ensure that when a certain route is invoked by the browser, it calls the right binding. The router part will deal with making sure the correct code is executed. Because we will be testing the interplay of our objects and the router intreracts with the `Window` object we will be stubbing our View, Model and Window objects. To that end we'll use a library called [Sinon.js](http://sinonjs.org/). Thankfully there's a [module for karma](https://www.npmjs.com/package/karma-sinon), that we can install:

	$ npm install karma-sinon --save-dev
	
Next amend the Karma configuration to use that framework:

	...
	// frameworks to use
   	// available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine', 'commonjs', 'sinon'],        
	...
	
_What are Spies, Stubs and Mocks_
	
### Routes

Test code: 

    'use strict';
    
    var ApplicationRouter = require('weatherly/js/ApplicationRouter.js');
    
    describe('Application Routes', function () {
        beforeEach(function () {
            this.router = new ApplicationRouter();
            this.indexRouteStub = sinon.stub(this.router, 'index');
    
            Backbone.history.start({silent: true, pushState: true});
            this.router.navigate('elsewhere', {trigger: true});
        });
    
        afterEach(function () {
            Backbone.history.stop();
            this.indexRouteStub.restore();
            this.router.navigate('elsewhere', {trigger: true});
        });
    
        describe('index route', function () {
    
            it('/ - route exists and points to the right method', function () {
                expect(this.router.routes['']).toEqual('index');
            });
    
            it('calls the index route by navigating to /', function () {
                var self = this,
                    pushStateStub = sinon.stub(window.history, 'pushState', function (data, title, url) {
                        expect(url).toEqual('/');
                        self.router.index();
                    });
    
                this.router.navigate('/', {trigger: true});
                expect(pushStateStub.called).toBe(true);
                expect(this.indexRouteStub.called).toBe(true);
    
                pushStateStub.restore();
            });
        });
    });
    
Here we stub out two things, the `window.hostory.pushState` function and the function that responds `index` route. For this particular set of tests we are checking two things, we have configured the correct route for our default app state and secondly when `triggered` we call the associated `index` method. We are not interested in what happens when the method is called and therefore we can stub it.  
    
Code to make this test pass: 

    'use strict';
    
    var ApplicationRouter = Backbone.Router.extend({
        routes: {
            '': 'index',
        },

        index: function () {
        },
    
    });
    
    module.exports = ApplicationRouter;
    
### Router
When it comes to testing our actual router code, the testing story becomes interesting. The typical implementation for this would look something like this:

    index: function () {
        var model = new TodaysWeatherForecast(),
              view = new TodaysWeatherForecastView({model: model});
        
         view.render();
    }

Nice and straightforward, however not testable. Well not quite true, you could add a fixture to your test code and then assert that the element built by our render function has been injected into the fixture. That's a valid test, however at this stage of the application I am not quite ready to run such a comprehensive test, such an integration test. I would still like to focus the tests at the `Unit level`. Why is this not testable? Before answering maybe the better question would be what would we test? The thing to test in my mind is that when we call index(), we should view.render to have been called. Typically you would use a `Spy` to achieve that. [What is a spy](http://sinonjs.org/docs/#spies)?

> A test spy is a function that records arguments, return value, the value of this and exception thrown (if any) for all its calls. A test spy can be an anonymous function or it can wrap an existing function.

With this implementation there's no way for our test code to get a hold of the instance created by this code and attach a spy to the View's `render` method and then assert it was called. So we need to find a way to inject some objects that we can create and set up spies. 

### Enter the builder
Our builder object is a kind of [Factory](http://en.wikipedia.org/wiki/Factory_method_pattern#Definition) that will help us create the `View` object and thus allow us to inject some fake objects. At this stage some folks will raise their hands in the air and mutter something about adding code for test purposes to our production code. The method for injecting code is not great, but I can easily make a case for this Builder in my Router. Why should my router have knowledge about the Model and the View? Why should it know how these are composed in order to their job? By adding this Builder object not only are decoupling our the View/Model relationship from the Router, but we are also adding a seam for our test code. So we solve two issues in one go and that I am Ok with.

    'use strict';
    
    var TodaysWeatherForecast = require('weatherly/js/model/TodaysWeatherForecast.js'),
        TodaysWeatherForecastView = require('weatherly/js/view/TodaysWeatherForecast.js');
    
    module.exports = (function () {
        var _view;
    
        function todaysWeatherForecast () {
            var model = new TodaysWeatherForecast();
    
            return new TodaysWeatherForecastView({model: model});
        }
    
        return {
            build: {
                todaysWeatherForecastView: function () {
                    if(_view) {
                        return _view;
                    }
    
                    return todaysWeatherForecast();
                }
            },
            _setView: function (view) {
                _view = view;
            }
        };
    })();


## The Application
