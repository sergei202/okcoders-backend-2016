Overview for June 23rd
======================

So far we've built node applications that allow us to interact with them over the internet using routes and urls.  This is great, but not very user-friendly.  Today are going to add a *front-end* to our node applications.

We do this by building a front-end (html, css, images, etc) and telling express to serve these static elements.  They are called *static* because the files never change.

Express Static Middleware
-------------------------
We've seen `express.static()` once before, but let's review what it does and how it is used.

`express.static(path)` allows us to serve static content at a specific path (directory/folder).  It is an *express middleware*, so we have to tell express where we want our static content to show up, this is called the *mount point*, it is just another route, usually `'/'`.

`app.use(mount, express.static(path))`

Here is a simple app that will serve whatever files are in the current directory:
```js
var express = require('express');         // Don't forget to install the express module

var app = express();                      // Create our express app

app.use('/', express.static('./'));       // Tell express to mount whatever files are in the current directory at /

app.listen(8080);                         // Start our server
console.log('Listening at http://localhost:8080');

```

If you saved the file as `static.js`, you can go to `http://localhost:8080/static.js` to view it.

Create a new file called `index.html` and add some html code: `<h1>Hello World</h2>`.  You can access it at `http://localhost:8080/index.html`.  You can also access it at `http://localhost:8080/`, how can this be?  The file name `index.html` is special.  If you don't specify a filename, express will automatically try to serve `index.html`.

You add files to that directory and you can access them from the browser. Try adding some images.  You just made a web server!

Express Static Organization and Security
----------------------------------------
As we demonstrated above, serving the current directory also exposes your server-side code (node js files). For our simple/demo projects, it's not a big deal.  But in the real world you might have API keys and other sensitive information inside your code.  We don't people to have access to it.

Our directory will also get messy as we start adding more static files to our project.

We solve both problems with the same solution: We store our static files in a separate directory from our code and only expose that directory to `express.static()`.

If we put all of our static files inside a child directory called `public`, we can serve only it's contents:
```js
app.use('/', express.static('./public'));
```

Remember, `/` is the *mount point* for this static content in our express app.  `./public` is the relative path to the *public* folder from our node program.

jQuery 101
==========
For small very projects, we will use jQuery.  Instead of having to download it, will load it via it's [CDN](https://code.jquery.com/jquery-3.0.0.min.js), *Content Distribution Network*.

Let's see the minimum needed for a jQuery example:
```html
<!DOCTYPE html>
<html>
<head>
	<script src="https://code.jquery.com/jquery-3.0.0.min.js"></script>   <!-- Load jQuery -->
</head>
<body>
	<div id="messageTarget"></div>                <!-- Create an empty div with an id of 'messageTarget'.  This is where we will place our text. -->

	<script type="text/javascript">
		$(document).ready(function() {			                // Our client-side program waits till the document and jQuery are ready
			console.log('jQuery ready!');
			$('#messageTarget').html('Hello <b>World</b>!');    // Using a CSS selector, find our target div by its id, then set it's content to 'Hello World'
		});
	</script>
</body>
</html>
```
We'll use the `$(selector).html(content)` pattern anytime we want to display something in clint-side javascript.

### jQuery and HTML Inputs
Now we know how to output things, how do we get input?  We can use standard html form inputs and a little bit of jQuery:
```html
<input type="text" id="myName">
```
```js
var myNameText = $('#myName').val();
console.log('myName = ', myNameText);
```
If we try running this code in `index.html`, we'll see that the console says *myName* is always blank, regardless of what we put in the input field.  That's because our code reads and outputs the content of *myName* immediately, before we could ever enter something into the field.  Let's have the user tell us when they are ready by adding a button and have our code run when the button is pressed.

```html
<input type="text" id="myName">
<input type="button" id="myButton" name="Submit">
```
```js
$('#myButton').on('click', function() {
	var myNameText = $('#myName').val();
	console.log('myName = ', myNameText);
});
```
We attach an *event listener* that listens for *click* events on `#myButton`, when the button is clicked, jQuery runs our event handler function (just like express does for our routes) and we read and console out the name the user entered.

Below is a full example of asking for a user's name and outputting it back to them.
```html
<!DOCTYPE html>
<html>
<head>
	<script src="https://code.jquery.com/jquery-3.0.0.min.js"></script>   <!-- Load jQuery -->
</head>
<body>

	<input type="text" id="myName">                           <!-- This is an input text field for the user to tell us their name -->
	<input type="button" id="myButton" value="Say My Name">   <!-- Our submit button.  'value' is the button label -->

	<div id="nameOutput"></div>                               <!-- Create an empty div for our output -->

	<script type="text/javascript">
		$(document).ready(function() {                        // Our client-side program waits till the document and jQuery are ready
			console.log('jQuery ready!');

			$('#myButton').on('click', function() {           // Attach a click event handler onto our button
				var myNameText = $('#myName').val();
				var outputHtml = '<h1>Hi ' + myNameText + '!!!</h1>';
				$('#nameOutput').html(outputHtml);
			});
		});
	</script>
</body>
</html>
```

### jQuery Ajax
We know how to make express routes, and we know how to trigger them by browsing to that location; but how can we trigger them dynamically, like when a user presses a button?

Enter Ajax (*Asynchronous Javascript and XML*).  Ajax allows to request web resources dynamically.  Since we are only using GET requests so far, let's start there:
```js
$.get(url, function(result) {
	console.log('Server returned: ', result);
});
```

Let's see `$.get()` in action:
```js
// This is our server-side Node app (eg: ajax.js)
var express = require('express');                   // Don't forget to install the express module

var app = express();                                // Create our express app

app.use('/', express.static('./public'));           // Tell express to mount whatever files are in the current directory at /

app.get('/time', function(req,res) {                // Add a '/time' route that returns the current date/time
	var now = new Date();							// Create a new Date object (defaults to the current time)
	res.json(now.toString());						// Respond with our date converted to a string
});

app.listen(8080);									// Start our server
console.log('Listening at http://localhost:8080');
```

```html
<!-- This is public/index.html -->
<!DOCTYPE html>
<html>
<head>
	<script src="https://code.jquery.com/jquery-3.0.0.min.js"></script>   <!-- Load jQuery -->
</head>
<body>

	<script type="text/javascript">
		$(document).ready(function() {                        // Our client-side program waits till the document and jQuery are ready
			console.log('jQuery ready!');

			$.get('/time', function(result) {                 // Send a GET request to our /time route
				console.log('The server says: ', result);     // Console log whatever the server returned
			});
		});
	</script>
</body>
</html>
```


### More Resources
If you are not familar with jQuery, here are a few resources:
- http://jqfundamentals.com/chapter/jquery-basics
- https://www.codeschool.com/courses/try-jquery
- https://www.codecademy.com/learn/jquery
