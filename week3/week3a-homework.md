Homework for June 21st (Due June 23rd)
======================

Read up on Express Routing
--------------------------
Review [11-express](https://github.com/sergei202/okcoders-backend/tree/master/11-express), make sure you have a firm grasp of express routing.

If you need more (or better) documention, check out these resources:
- [ExpressJS Hello World Tutorial](http://expressjs.com/en/starter/hello-world.html)
- [Scotch.io](https://scotch.io/tutorials/learn-to-use-the-new-router-in-expressjs-4)


Weather Forecast App
------------------------
You are going to create a very simple application that will return the weather for a given zip code.  The heart of this project will be the  [forecast](https://github.com/jameswyse/forecast) node module.

Here is an example of using the `forecast` module:
```js
var Forecast = require('forecast');                         // Require the forecast module

var forecast = new Forecast({                               // Initialize an instance of Forecast
	service: 'forecast.io',
	key: '5b2a502a3e9cce8715fcebc3343372e0',                // forecast.io API key
	units: 'F'                                              // F or C (only the first letter is parsed)
});

forecast.get([36.12, -95.1], function(err,weather) {       // Retrieve weather information from the passed coordinates (Tulsa, OK) (latitude, longitude)
	if(err) return console.dir(err);                       // If error, print out the error and return
	console.dir(weather);                                  // List our weather object
});

```
This is great, but we need the latitude/longitude to look up the weather; not something most people know off the top of their head.  We need a way to convert a zip code into lat/long coordinates.  Lucky for us, we've already used a node module that does that: `zipcodes`!

```js
var zipcodes = require('zipcodes');                       // Require the zipcodes module

var location = zipcodes.lookup('74133');                  // Look up a zip code and assign the returned object to the location variable
console.log(location.longitude, location.latitude);       // Print out our coordinates
```

Everyone will need to incorporate express routing to take a passed zip code and return the weather, returning the current summary text would be ideal:

Going to `http://localhost:8080/weather/74133` would return `Partly Cloudy`.

We will build on top of this project, so create a separate directory (`weather` or similar) for this project.  Feel free to commit to github if you remember the steps.
