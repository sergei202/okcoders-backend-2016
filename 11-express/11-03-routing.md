Express: Routing
==============================

The express web framework makes it easy for us to target specific http requests with the appropriate responses.

When a request reaches our web server it includes information such as the request action (GET, POST, PUT, DELETE), the requested resource in the form of a url, and additional headers as well as any body content. Normally we would need to parse that information manually and, using application logic, ensure that the correct response is generated.

For example, we could parse the action and url to see that the request wants to `GET` the `/about.html` resource and so return the document located in the server's `public` (or `app`) folder at the location. Or we could see that the requests wants to `POST` to `/items` and contains body data and so create a new item with that data and return it.

Express provides facilities for ensuring that the right part of our application responds to the right requests without having to manually parse everything. This is called *routing*.

Routing allow us to specify an action and a url path that will match against incoming requests. We also provide a callback for custom code that is executed anytime someone makes a request that matches that action and url. In our callback we'll be able to access additional information in the url as well as any request headers easily and we'll be able to formulate a response to the request with very little work.


## Resources

[Express](http://expressjs.com/)

The Express homepage. Particularly important are the Guide and API Reference.

[Scotch.io](https://scotch.io/tutorials/learn-to-use-the-new-router-in-expressjs-4)

A great overview of the basics of express routing.

## Basic Routing

Basic express routing is straightforward.  Let's begin by creating a new application. Create a new directory, cd into it, and install the express module:

	$ mkdir my-first-express-route
	$ cd my-first-express-route
	$ npm install express

Let's create a new file called server.js:

```js
// Require the express module
var express = require('express');
// Create an application instance of express
var app = express();

// Let's define our routes
app.get('/', function(req,res) {
	res.send('Hello World!');
});

// Start our server
app.listen(8080);
console.log('Server started.  Visit: http://localhost:8080');
```

You can run our new server by telling node to execute `server.js`:  `node server.js`.

To add a route, call a function on the `app` object that corresponds to the http action you want to address and provide two parameters. The first is the path on your server for that resource and the second is a callback that takes two parameters: the request object `req` and the response object `res`. This callback will be called anytime someone makes a request to your server with that action and that path:

```js
app.get('/heartbeat', function(req, res) {
    res.send("Application Heartbeat: the application is running");
});
```

Because we've modified the application itself we must restart the server to see the changes. Ctrl-c `^C` to quit the application and start it up again with `node server.js`.

In this example we are targeting `GET` requests to `/heartbeat`, so browse to *http://localhost:8080/heartbeat* and you should see the response you've specified.

Notice the use of the `res.send` function to actually generate the server's response. This replaces the use of

```js
res.writeHead(200, {'Content-Type': 'text/plain'});
res.end("Unable to read file index.html\n");
```

that we saw with the `http` module.

We can of course specify more complex routes, for example routes with subpaths. Use the same format but modify the first parameter to the `get` function to target that resource:

```js
app.get('/orders/123/detail', function(req, res) {
    res.send('Return the order details for order 123');
});
```

Here we are targeting a `GET` request to `/heartbeat/format/index.html`, so once again restart your server and browse to that url on localhost:3000.

Even with just a few examples we can already see how routing allows specific parts of our application -- functions in the form of callbacks -- to respond to specific requests.

We can respond to other types of requests as well. Respond to a `POST` request with the `post` function:

```js
app.post('/heartbeat', function(req, res) {
    res.send("Cannot post to heartbeat");
});
```

Notice that we are using the same url twice, once for the get and once for the post. A request is identified by its action and its path together. `POST /heartbeat` is not the same as `GET /heartbeat`. Although they are the same path they are different requests. Modern web application make use of this fact to organize their service in a [RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer) manner, which we'll learn about in future chapters.

Now we don't have a simple way of generating a POST request from a web browser. By default when you type a URL into a web browser it is always uses a get request. Normally you need to submit a form for the browser to generate a POST request.

Fortunately we can generate post requests with a free Chrome extension called *Postman - REST Client*. Install that extension in your Chrome browser and start it up.

To use Postman, enter a url and select the http action. We want `http://localhost:3000/heartbeat` and `POST`. You can also add url parameters and additional headers which we don't need now. Make sure your application is still running and send the request. Try switching the request to GET and resending it to see the result for a get request.

Postman is a useful tool that we'll use to test our web application as we develop it.






Finally it is worth mentioning that we could also respond to 'PUT' and 'DELETE' requests like so:

```js
app.put('/heartbeat', function(req, res) {
    res.send("Cannot put to heartbeat");
});
app.delete('/heartbeat', function(req, res) {
    res.send("Cannot delete heartbeat");
});
```

All we need to do is change the function we are calling on `app` to `put()` or `delete()`. Verify that these routes work using Postman.

## Advanced Routing

Often we'll want to respond to similar but slightly different requests with the same code, or we'll want to be able to respond to requests whose resources we can't identify in advance.

This happens when we want the same code to run for many resources of the same type that are stored in a database but have unique identifiers. For example, it would be useful if the following paths:

	/orders/124
	/orders/567

executed the same code because all that is different is the order ID, and we can use that to look it up in a database.

Moreover we may not know the IDs of all the orders when we write the application. How can we specify a route for a resource which doesn't even exist?

**Named parameters**

Named parameters allow us to identify changes in a path around which the rest of the path remains the same. Then in our callback we are able to access the value of the variable portion. Named parameters are identified with a colon `:`.

Let's use a named parameter for our people path:

```js
app.get('/orders/:orderId', function(req, res) {
	var orderId = req.params.orderId;
	res.send("You requested order " + orderId);
});
```

The path `/orders/:orderId` allows the route to match any GET request with a path that looks like `/orders/x` where x is a single additional component in the path (e.g. no additional `/`). For example the path will match:

	/orders/1234
	/orders/blah
	/orders/12?format=json
	/orders/1234-5678-ABCD-EF90

But it will **not** match:

	/orders/123/more
	/orders/456/subpath

Then in the callback function express adds a `params` object to the `req` parameter which includes whatever named values the path specified. Access the value in the path by its name, so:

	req.params.orderId

Contains whatever text appears where

	:orderId

is in the path originally specified for the route.

Let's try another example:

```js
app.get('/blog/posts/:id', function(req, res) {
	var postId = req.params.id;
	res.send("You requested blog post " + postId);
});
```

This route will match any GET request for a `/blog/posts/x` where x is any single component, so paths like:

	/blog/posts/1
	/blog/posts/the-time-i-wrote-a-web-application
	/blog/posts/abcd

**Subpaths with named parameters**

We can use named parameters with subpaths. By default a named parameter only matches a single component of a path, the text between two forward slashes `/.../`, so we can add path information after the named parameter to further refine our matching. For example, you might want to be able to update a blog post or delete it with paths like:

	/blog/posts/1/update
	/blog/posts/1/delete

But you don't want to execut the same code used for viewing the post. Refine the matched path simply by adding the additional path information after the named parameter. The path for the router will look like:

	/blog/posts/:id/update
	/blog/posts/:id/delete

Here's the code:

```js
app.get('/blog/posts/:id/update', function(req, res) {
	var postId = req.params.id;
	res.send("You want to update blog post " + postId);
});

app.get('/blog/posts/:id/delete', function(req, res) {
	var postId = req.params.id;
	res.send("You want to delete blog post " + postId);
});
```

Or perhaps our posts also have comments that we might want to view:

```js
app.get('/blog/posts/:id/comments', function(req, res) {
	var postID = req.params.id;
	res.send("You want to view the blog post's commments " + postID);
});
```

Save your files and restart the server to view your changes.

**Multiple named parameters**

A single route can match against multiple variables in the path, and in fact it is common to have multiple named parameters. For example, what if I wanted to view a specific comment attached to a specific blog post? The url must identify both the blog post and the comment. We need two named parameters:

	/blogs/posts/:postid/comments/:commentid

This is a totally acceptable path for express and we'll send it to the `get` function as we normally do:

```js
app.get('/blog/posts/:postid/comments/:commentid', function(req, res) {
	var postId = req.params.postid;
	var commentId = req.params.commentid;
	res.send("You want to view the comment " + commentId + "for blog post " + postId);
});
```

Notice that both the named parameters are available on the `req.params` object. Be careful which name you refer to!

Restart the web server and visit paths such as:

	/blog/posts/12/comments/1
	/blog/posts/1234-5678-abcd-ef90/comments/foo-bar

## Organizing Routes

At this point the *app.js* file has gotten out of hand. We've already added forty lines of code but barely have a functional product yet. If we keep adding routes for additional paths and add complexity to our callback functions such as user logins, database access and better response generation the file will become bloated and difficult to understand at a glance.

Express helps us here by making it possible to organize our routes into modules. Notice the templated code we pushed down when we started writing our own routes:

```js
app.use('/', routes);
app.use('/users', users);
```

There's obviously something going on with paths here and so routes, but what? `USE` is not an http verb. And what are the `routes` and `users` variables?

Looking at the top of the file we see where these variables are defined:

```js
var routes = require('./routes/index');
var users = require('./routes/users');
```

These variables are modules just as we've seen with other node modules, and they refer to two local files as indicated by the `./` in the folder `routes`. Let's have a look at one of those files and break it down. Here is *routes/index.js*:

```js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

This is a simple looking module. First it includes the `express` module and then creates a router from it. It then attaches a `GET /` route to the router with its own callback, ignoring `res.render` for now, and finally exports the router. The *routes/users.js* file is similar.

This is how express orgnizes routes. Related routes should all go in their own modules kept in the routes folder. Each routes module creates its own subrouter with:

```js
var express = require('express');
var router = express.Router();
```

and then attaches its routes to it with `router.get()`, `router.post()`, `router.delete()` and so on, providing the same path matching and request-response callback.

Finally, the module should export the router directly onto the module's exports variable:

```js
module.exports = router;
```

Back in the main *app.js* file for the application, include the module with the `require` function, assigning it to a variable, and pass that variable to the `app.use()` function:

```js
app.use('/', routes);
```

**Scoping**

Notice that when `app.use` is called like this it takes two parameters. The second is your routing module variable, but the first is a path. Express *scopes* the paths in the module to that path. This means that any paths defined in the module must first begin with the path defined here.

For example, notice that the *routes/index.js* file defines one route for `GET /` and that here `app.use` scopes the module to `/`. This means that `/` in *index.js* will only match routes where the first part of the path is `/`, effectively only the `/` path itself:

	/

That might seem a little confusing, but it's more straightforward with the users routing module. That module also defines a single route, namely `GET /`, so the same one we see in *index.js*, but the `app.use` call is different:

```js
app.use('/users', users);
```

Passing in `/users` for the first parameter means that all the routes defined in *routes/users.js* are *scoped to* the `/users` path. They must being with `/users`, so that `/` in users now actually matches:

	/users/

**Example**

Let's make this clearer. Let's move our `/people/:username` route to the users routing module. We want it to match `/users/:username` instead, so delete the `/people` portion of the path and make sure to call `router.get` instead of `app.get`. With the correct changes, the following code is added to *routes/users.js*:

```js
router.get('/:username', function(req, res) {
    var username = req.params.username;
    res.send("You requested user " + username);
});
```

Save all the edited files and restart the server.

Now you can't just visit a path like

	/phil
	/okcoders

even though the users router is only matching `/:username`. That's because the `/:username` path is scoped to `/users` by the `app.use('/users', users)` call in *app.js*, so that the full matching path is actually `/users/:username`:

	/users/phil
	/users/okcoders

Scoping is a powerful feature just introduced into express and will help us greatly when it comes to orgnazing our code.

## Middleware

Express routing is organized around the concept of middleware. Middleware is code that executes between the server receiving the request and your own appliction's callback code. Routing is itself middleware, as it parses the request before your application even sees it and calls the right callback function in your application for you.

An application can have multiple layers of middleware connected together in a chain. We see them in express with the `app.use` function and any `app.VERB` function like `app.get` or `app.post`. Middelwares are executed in the order in which they are defined.

For example, we need to use the `bodyParser` middleware to parse `POST` requests:

```js
...
var bodyParser = require('body-parser');
...
app.use(bodyParser.json());
```

We'll use `bodyParser` in later projects.

Because the middleware is included before the routing, where your application's custom code resides, they are executed before your application has a chance to see the request.

Each middleware has access to the request and response objects just like your routing callback functions do. Middleware can do whatever it wants with the information in the request as long as it either sends a response back or calls the next middelware item in the chain.

At some point in the middleware chain a response must be generated by calling the appropriate functions on the `res` object such as `res.send`, `res.sendfile` or `res.json`. We'll learn more about the response object in the next chapter. For now it's important to know that once a response is generated, the chain immediately exits and no additional code is executed.

This means that middleware can respond to a request without letting your own application code see it. It also means that only a single custom route has the opportunity to actually generate a response. If more than one route matches an incoming request, whichever route is defined first is the one that responds to it.

**The public directory**

There is one particularly special piece of middleware we've already seen in class.  It is express's own `static` middleware, added to the application with:

```js
app.use(express.static(__dirname +  '/public'));
```

The `static` middleware is used to send files in the specified directory automatically. It looks in the directory specified and if it finds a file that matches a GET request's path it automatically sends that file.

By default express uses a `public` directory but you could use whatever directory you want and even add additional directories.
